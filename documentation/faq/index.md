---
title: Frequently Asked Questions
layout: default
currentpage: faq
---

Frequently Asked Questions
===

Here are some answers to questions you may have as you configure and use Jasper.

Installation &amp; Configuration
---

- __Why can't I can't log in to my Raspberry Pi for the first time?__
    - Are you sure you imaged your SD card correctly? Your Raspberry Pi should emit a green LED light when booting. It may help to wipe your SD card clean with a tool like gparted before imaging with dd. See the [Software Guide](/documentation/software/) for full instructions.
- __Why isn't Jasper allowing me to set up a wireless network?__
    - Make sure you've followed all of the instructions in the 'Configure Wireless' section of the [Software Guide](/documentation/software/). If Jasper is still not providing the opportunity to configure the wireless connection, temporarily disconnect the wireless adapter, restart the Pi fully, then restart the Pi once more with the wireless adapter connected. Jasper should now ask you to configure the wireless network.

Interacting with Jasper
---

- __Why isn't Jasper responding to its name?__
    - Make sure your Raspberry Pi is turned on and has been allowed to boot for a few minutes. Try moving closer to the microphone. If all else fails, you can SSH into the Raspberry Pi and run main.py yourself to see Jasper's interpretation of your input.
- __Why can't I log in to my Raspberry Pi after running Jasper?__
    - Jasper's attempts to connect with a wifi network may be interfering with your attempts to make an ethernet connection. To resolve this, disconnect your wifi adapter, restart your Jasper, then plug in the ethernet cable only _after_ Jasper fails to find a wireless network.

Developing for Jasper
---

- 

