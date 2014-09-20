---
layout: post
title: "Writing a web server from scratch - 2"
date: 2014-04-26 21:33
comments: true
categories: unix
---
<p>Over the past few days I spent some time writing a web server and it has been very gratifying and I learnt about quite a few things. So let me start off with the design, the thing about web servers is that its quite simple to implement the core functionality but the bulk of it is the plumbing work around parsing the different rules of the HTTP protocol, which I&rsquo;ve kept it to the bare minimum as modern browsers have sane defaults.</p>

<h2>Design</h2>

<p>So the basic outline is like this:</p>

<ol>
<li>Open a socket, bind it to a port and start accepting connections.</li>
<li>Once you receive a request from a client, pass the socket information to a handler function and continue servicing other requests.</li>
<li>In the handler function, parse the request headers and generate your response headers and body accordingly. For serving static files, simply do a buffered write on the socket after writing the headers.</li>
<li>Close the socket and any other file descriptors, free resources allocated during the request.</li>
</ol>

<p>So there are a number of ways how you can implement the listening the listen loop with concurrency (Steps 1 and 2) . A rather naive approach would be to fork a new process on every request and let the child handle the request after closing its access to the server socket. This might appear to work but there is a problem, once you hit 700 requests (or whatever your system&rsquo;s limit for processes per user is) your server will stop working and a lot of other processes that are running in the background will begin to act strangely. The reason is that even if you exit the child process after handling the request, they will exist as <a href="http://en.wikipedia.org/wiki/Zombie_process">zombie processes</a>. Zombies store some state information and take up a very small chunk of memory but cause the kernel to freak out upon reaching the user limit. They are destroyed only when the parent process exits. You can check the limit with the shell command <code>ulimit -a</code>.  So the solution to this will be to service the request in the parent process itself and let the child service the next request. This way the parent process will usually be killed before the child and no zombies will be created. Even if they are created, they will be cleared as soon as the parent completes handling the request. But still this approach has performance problems as we will see later.</p>

<p>A second, more common approach is to create a process pool and is used by servers like Apache. After creating the listening socket, you fork the parent process a number of times to create a process pool. Each of the child processes have a separate file descriptor for the listening socket, but the kernel manages things in a way that they all point to the same socket. So depending on which process is active at that point of time, it gets the client request. Other approaches which I have not explored yet, but will do in the coming few days are multithreading and asynchronous I/O.</p>

<p>Parsing requests and generating responses is fairly straightforward, which is why I only implemented a small subset of the HTTP protocol features.</p>

<h2>Benchmarks</h2>

<p>It&rsquo;s time for some benchmarks, I&rsquo;ll be comparing this file server with Apache httpd and Python&rsquo;s SimpleHTTPServer module which I often use for transferring files across wifi. I&rsquo;m not hoping to compete with Apache on speed here but this should be defenetely  I was thinking that I might have to write another program for stress testing the file servers but fortunately <strong>ab</strong> or ApacheBench comes to the rescue. You can specify the number of requests, number of concurrent connections etc. For the tests each of these servers will be serving a file (the compiled binary of my server) and both client and server will be on the same node. </p>

<p>This is the command I used for testing.</p>

<p><code>ab -n 8000 -c 10 -r localhost/fsrv</code></p>

<p><strong>SimpleHTTPServer</strong> failed with 10 concurrent connections, so I reduced it to 4. But it was still failing, so I reduced the number of requests to 300  and this was the result:</p>

<pre><code>Python SimpleHTTPServer

Requests per second:    148.06 [#/sec] (mean)
Time per request:       27.015 [ms] (mean)
Time per request:       6.754 [ms] (mean, across all concurrent requests)
</code></pre>

<p>Then I tested <strong>Apache</strong> with the original command, and Apache turned out to be much more robust with over <strong>4100 requests per second</strong>.</p>

<pre><code>Apache httpd

Requests per second:    4111.07 [#/sec] (mean)
Time per request:       2.432 [ms] (mean)
Time per request:       0.243 [ms] (mean, across all concurrent requests)
</code></pre>

<p>So time to test my server, first the fork on every request model</p>

<pre><code>fsrv fork every request

Requests per second:    355.25 [#/sec] (mean)
Time per request:       28.149 [ms] (mean)
Time per request:       2.815 [ms] (mean, across all concurrent requests)
</code></pre>

<p>Well, its faster than python but doesn&rsquo;t seem all that encouraging, let me try the process pool model with 8 processes.</p>

<pre><code>fsrv process pool (8 procs)

Requests per second:    759.35 [#/sec] (mean)
Time per request:       13.169 [ms] (mean)
Time per request:       1.317 [ms] (mean, across all concurrent requests)
</code></pre>

<p>Better, but pales in comparison to Apache. Time to run a profiler. On a side note Apple&rsquo;s Instruments is a much better profiler than gprof for profiling C/C++ code. </p>

<p>After running the profiler,  I found that the majority of the time was taken in determining the mime type for a file. I had used the unix <code>file</code> command which does some magic behind the scenes to determine a file&rsquo;s mime type. I had used popen to open a pipe and fed data interactively to it upon each request.</p>

{% highlight c%}
void get_mime_type(const char *filename, char *mime_type) {
    static FILE* pipe = NULL;
    if (pipe == NULL) {
        // Run xargs in interactive mode
        pipe = popen("xargs -n 1 file --mime-type -b", "r+");
    }
    fprintf(pipe, "%s\n", filename);
    int read = fscanf(pipe, "%s", mime_type);
    if (!read)
        strcpy(mime_type, "application/octet-stream"); // Default mime type
}
{% endhighlight %}

<p>One way to fix this would be to cache the values for each file or just write a basic pure C mime type generator without any external calls based on a few rules. For the time being I decided to use a simple default header for the mime type. Time for the benchmarks again.</p>

<pre><code>fsrv fork every request

Requests per second:    3119.81 [#/sec] (mean)
Time per request:       3.205 [ms] (mean)
Time per request:       0.321 [ms] (mean, across all concurrent requests)
</code></pre>

<p>I was quite surprised with the results as I thought forks would be quite expensive. After a little bit of searching, I came to know that you can fork around 3-4k times a second.</p>

<pre><code>fsrv process pool

Requests per second:    7256.96 [#/sec] (mean)
Time per request:       1.378 [ms] (mean)
Time per request:       0.138 [ms] (mean, across all concurrent requests)
</code></pre>

<p>Incredible! This is about <strong>1.76x</strong> faster than Apache!. So there you have it you can write a simple file server from scratch using only the Unix APIs, that is quite a lot faster than Apache. Though, I must admit that Apache has tons of features that would slow it down. Feel free to check out the code at [<a href="http://github.com/vivekn/fsrv">http://github.com/vivekn/fsrv</a>]</p>
