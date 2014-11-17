---
layout: post
title: "Yosemite battery issues."
date: 2014-11-16 19:29
comments: true
categories: mac osx
---
After upgrading OSX to Yosemite, I noticed a sharp rise in battery usage in sleep mode. I usually close the lid, instead of shutting down and used to get a really good battery life on my Air. But after the upgrade, I would lose about 30-40% of the charge overnight. 

So when you close the lid and your mac enters sleep mode, it is still running, only the displays have been turned off. After some time, memory is flushed to disk and it actually enters sleep mode. Turns out Apple have increased the delay for this to happen in the default configuration. To view your power management settings, use the command line tool **pmset**.

{% codeblock lang:bash %}
$ pmset -g
Active Profiles:
Battery Power		-1*
AC Power		-1
Currently in use:
 standbydelay         10800
 standby              1
 halfdim              1
 hibernatefile        /var/vm/sleepimage
 darkwakes            0
 disksleep            10
 sleep                1
 autopoweroffdelay    14400
 hibernatemode        3
 autopoweroff         1
 ttyskeepawake        1
 displaysleep         2
 acwake               0
 lidwake              1
{% endcodeblock %}

So the problem here is that **standbydelay** is 10800 or 3 hours, so you laptop is essentially wasting power for three hours when it is in sleep mode. If it doesn't enter standby mode, it can immediately power up by just turning on the display, but it isn't worth it. So to reduce the **standbydelay** to 20 minutes use this command.

{% codeblock lang:bash %}
$ sudo pmset -a standbydelay 1200 
{% endcodeblock %}