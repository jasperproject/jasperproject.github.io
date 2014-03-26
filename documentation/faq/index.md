---
title: Frequently Asked Questions
layout: default
currentpage: faq
---

Frequently Asked Questions
===

Here are some answers to questions you may have as you configure and use Jasper.

<h2 class="linked" id='installation-configuration'><a href="#installation-configuration" title="Permalink to this headline">Installation &amp; Configuration</a></h2>

- __Why can't I can't log in to my Raspberry Pi?__
    - Are you sure you imaged your SD card correctly? Your Raspberry Pi should emit a green LED light when booting. It may help to wipe your SD card clean with a tool like gparted before imaging with dd. See the [Software Guide](/documentation/software/) for full instructions.
- __Why isn't Jasper allowing me to set up a wireless network?__
    - Make sure you've followed all of the instructions in the 'Configure Wireless' section of the [Software Guide](/documentation/software/). If Jasper is still not providing the opportunity to configure the wireless connection, temporarily disconnect the wireless adapter, restart the Pi fully, then restart the Pi once more with the wireless adapter connected. Jasper should now ask you to configure the wireless network.
- __Why does Jasper ask for my Gmail password?__
    - As with any information requested in the profile populator, this is purely optional. If provided, Jasper will be able to tell you when you have new emails. Additionally, Jasper will use this information to send relevant messages to your email and/or phone (e.g., links to articles that you've requested).

<h2 class="linked" id='interacting'><a href="#interacting" title="Permalink to this headline">Interacting with Jasper</a></h2>

- __Why isn't Jasper responding to its name?__
    - Make sure your Raspberry Pi is turned on and has been allowed to boot for a few minutes. Try moving closer to the microphone. If all else fails, you can SSH into the Raspberry Pi and run main.py yourself to see Jasper's interpretation of your input.

- __Does Jasper play well with two-factor email authentication?__
    - Unfortunately, no. If you use two-factor authentication for your Gmail, there's currently no way to integrate it with Jasper (although this would be a welcome pull request!).

<h2 class="linked" id='developing'><a href="#developing" title="Permalink to this headline">Developing on Jasper</a></h2>

- __Is there any way to interact with Jasper on the command-line?__
    - Yes! Try running: "python main.py --local" when SSHed into the Pi to access Jasper's command-line interface (CLI). Note that there's no need to type "Jasper" between commands when using the CLI.
