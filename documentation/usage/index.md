---
title: Using Jasper
layout: default
currentpage: usage
---

Using Jasper
===

Selecting a wireless network
---
When you turn on Jasper for the first time, it will ask you to configure a new wireless network connection. From your laptop, connect to the temporary "Jasper" network that will appear after a few minutes.

After connecting to Jasper's temporary network, browse to [192.168.1.1:8000/cgi-bin/index.cgi](http://192.168.1.1:8000/cgi-bin/index.cgi) and follow the on-screen instructions. When complete, your Jasper will restart and connect to your selected wireless network automatically. 

To change the wireless network in the future, just turn on your Jasper without the wifi adapter, wait a few minutes, then restart Jasper with the wifi adapter plugged in. Jasper will again ask you to select a wireless network and you can just follow the steps outlined above.

Interacting with Jasper
---
The most common way to speak with Jasper is with the following sequence:

You: "Jasper"
Jasper: *high beep*
You: *speak your command*
Jasper: *low beep*
Jasper: *speaks the response*

After saying Jasper, you must wait for the high beep to speak your command. If you don't speak a command within a few seconds, Jasper will stop listening or ask you to repeat your command.

Modules to try
---
By default, we've included the following modules to demonstrate Jasper's capabilities:

- Time: "What's the time?"
- Weather: "How's the weather?... What's the weather like tomorrow?"
- News: "What's in the news?"
- Gmail: "Do I have any email?"
- Hacker News: "What's on Hacker News?"
- Facebook Notifications: "Facebook notifications?"
- Birthday: "Who has a birthday today?"
- Jokes: "Tell me a knock-knock joke."
- Life: "What is the meaning of life?"


To learn how to write your own module, check out the [Developer API documentation](/documentation/api)