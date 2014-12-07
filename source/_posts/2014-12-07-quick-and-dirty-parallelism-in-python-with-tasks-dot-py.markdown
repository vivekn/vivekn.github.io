---
layout: post
title: "Quick and dirty parallelism in Python with tasks.py"
date: 2014-12-07 14:16
comments: true
categories: python multiprocessing concurrency eventlet
---

Python doesn't have a great reputation for executing concurrent or parallel code because of the global interpreter lock (GIL). Only one thread of a process can execute python code at a time. But due to some excellent libraries like multiprocessing and eventlet, it is straightforward to get parallelism on all your sequential functions by just adding a couple of lines of code. 

[__tasks.py__](https://github.com/vivekn/tasks) is a simple and fast task queue for executing multiple tasks in parallel. All you need to do is specify the task as a simple function that takes an argument and you get instant parallelism.

It is ideal for executing multiple network bound tasks in parallel from a single node, like fetching a list of urls, crawling a site or making a lot of third party API calls, without going through the pain of setting up a map reduce cluster. 

Installation
---------------
Since it uses [Redis](http://redis.io) as a backend, install Redis and start the server. If you are already using redis, you can pass the custom connection object using the `tasks.set_redis` call.

Install tasks_py

    $ sudo pip install tasks_py

 Usage
----------

A task is a function that takes a single string argument. Define such a function and register it using `tasks.set_func` . If the function raises an exception, it is considered to have failed.

Call `tasks.main()` to get the interactive command line options. 	

{% codeblock lang:python %}
    import eventlet
    eventlet.monkey_patch()
    import tasks
    
    from urllib2 import urlopen
    
    def fetch(url):
    	f = open('/tmp/download')
    	body = urlopen(url).read()
    	f.write(body)
    	f.close()
    	
    tasks.set_func(fetch)
    tasks.main()
{% endcodeblock %}

Note that it is important to import eventlet and call `monkey_patch` to replace the blocking network calls with asynchronous IO from eventlet.

Now to add jobs, create a file with one argument per line and use this command.

`$ python yourfile.py add <list_of_jobs.txt>`

To start (or restart) the job processing (do this in a **screen** session or close the input stream):

`$ python yourfile.py run`

**tasks** has resume support, so it will start where you left off the last time.

To view the current status while it is running: 

`$ python yourfile.py status`

Once you are done, you can clear the logs and the completed tasks by calling reset.

`$ python yourfile.py reset`


How it works
----------------------
It uses a **multiprocessing** pool to distribute tasks across multiple processes and hence bypasses the GIL. Even with multiple processes, there is a problem as most of the time will be spent blocked on a network request. Here is where, **eventlet** comes in to the picture, each process has an IO loop and a number of coroutines or "green threads".  A green thread is similar to a thread but it is lightweight and has very less overhead. A single green thread runs at a time, but whenever a green thread is waiting on I/O or network, the IO loop switches the thread out and a different green thread is executed. It is a non blocking approach that scales very well. 



The code is available on [Github](https://github.com/vivekn/tasks). Feel free to fork and modify this.