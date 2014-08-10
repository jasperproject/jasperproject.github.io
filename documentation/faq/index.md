---
title: Frequently Asked Questions
layout: default
currentpage: faq
---

Frequently Asked Questions
===

Here are some answers to questions you may have as you configure and use Jasper.

<h2 class="linked" id='installation-configuration'><a href="#installation-configuration" title="Permalink to this headline">Installation &amp; Configuration</a></h2>

- __Why can't I can't login to my Raspberry Pi for the first time?__
    - Are you sure you imaged your SD card correctly? Your Raspberry Pi should emit a green LED light when booting. It may help to wipe your SD card clean with a tool like [GParted](http://gparted.org) before imaging with dd. See the [Software Guide](/documentation/software/) for full instructions.
- __Why isn't Jasper allowing me to set up a wireless network?__
    - Make sure you've followed all of the instructions in the 'Configure Wireless' section of the [Software Guide](/documentation/software/). If Jasper is still not providing the opportunity to configure the wireless connection, temporarily disconnect the wireless adapter, restart the Pi fully, then restart the Pi once more with the wireless adapter connected. Jasper should now ask you to configure the wireless network.
- __What does Jasper do with the information I provide in my profile?__
    - The information in the profile allows Jasper modules to tailor their responses and actions to you. For example, the weather module will be able to tell you about the weather in your area, and the time module will be able to report a time that's sensitive to your timezone. None of this information ever leaves the Pi (unless, of course, you create a module that sends the profile information elsewhere).
- __Why does Jasper ask for my Gmail password?__
    - As with any information requested in the profile populator, this is purely optional and will never leave the Pi. If provided, Jasper will be able to tell you when you have new emails. Additionally, Jasper will use this information to send relevant messages to your email and/or phone (e.g., links to articles that you've requested). As an alternative, consider using [Mailgun](/documentation/software/#mailgun).
- __Why do I get a segmentation fault when running `main.py`?__
    - This question concerns the following error:

          SYSTEM_ERROR: "dict.c", line 274: Failed to open dictionary file [...] No such file or directory
          zsh: segmentation fault  python main.py

      It's caused by the speech dictionary not being created during the boot process. Make sure you've [added the boot script](/documentation/software/#install-client) to crontab, then restart your Pi. If a `dictionary.dic` file is still not present in your `client/` directory, just run `~/jasper/boot/boot.py` manually to force the dictionary generation.
- __Can I plug in my own speech recognition engine? (Google Speech, etc)__
    - Sure, check out the instructions in [client/stt.py](https://github.com/jasperproject/jasper-client/blob/master/client/stt.py). While Pocketsphinx is the default, an example using the Google Speech API is bundled with the client.

<h2 class="linked" id='interacting'><a href="#interacting" title="Permalink to this headline">Interacting with Jasper</a></h2>

- __Why isn't Jasper responding to its name?__
    - Make sure your Raspberry Pi is turned on and has been allowed to boot for a few minutes. Try moving closer to the microphone. If all else fails, you can SSH into the Raspberry Pi and run main.py yourself to see Jasper's interpretation of your input.
- __Why can't I login to my Raspberry Pi after running Jasper?__
    - Jasper's attempts to connect with a wifi network may be interfering with your attempts to make an ethernet connection. To resolve this, disconnect your wifi adapter, restart your Jasper, then plug in the ethernet cable only _after_ Jasper fails to find a wireless network.
- __Does Jasper work with two-factor email authentication?__
    - Unfortunately, no. If you use two-factor authentication for your Gmail, there's currently no way to integrate it with Jasper (although this would be a welcome pull request!).
- __What if I don't have speakers that work with the Pi or don't want Jasper to speak out loud?__
    - I'd suggest using headphones! This is particularly useful for development, during which you might have Jasper repeating himself over and over, speaking sporadically, etc.

<h2 class="linked" id='developing'><a href="#developing" title="Permalink to this headline">Developing on Jasper</a></h2>

- __What if my module's keywords conflict with those of another module?__
    - Modules are ordered using a relative priority system, so a module with higher priority will be given preference when two modules would both accept a certain input. In general, the more specific the module's `isValid` function, the higher it's priority can be, as you'll trigger fewer false positives. For more detail, see [here](/documentation/api/standard/#priorities). For an example in the code, see the ['News'](https://github.com/jasperproject/jasper-client/blob/master/client/modules/News.py) and ['Hacker News'](https://github.com/jasperproject/jasper-client/blob/master/client/modules/HN.py) modules, which would both accept the input "what's on hacker news?"; the higher priority of the ['Hacker News'](https://github.com/jasperproject/jasper-client/blob/master/client/modules/HN.py) module ensures that this input is passed to the correct module.

- __Is there any way to interact with Jasper on the command-line?__
    - Yes! Try running: "python main.py --local" when SSHed into the Pi to access Jasper's command-line interface (CLI). Note that there's no need to type "Jasper" between commands when using the CLI, as you would when interacting through speech.
- __What if my module needs more accurate information from the input text? For example, what if I need to capture a number ("one", "two", "three", etc.)? I can't list every number in the input, so what do I do?__
    - If you need more accurate speech recognition, you could modify Jasper to utilize the [AT&T speech-to-text API](https://developer.att.com/apis/speech). This will capture a larger vocabulary at the cost of being somewhat slower and rate-limited. The primary modifications would come in mic.py, which would need to change its rendering function to call the AT&T API. Alternatively, consider limiting the set of _relevant_ words that the user might say (e.g., if the user is choosing from a list of three items, just add "one", "two", and "three" to the keywords list, rather than worrying about all possible numbers).

- __What if I want to contribute to Jasper?__
    - Great! There's always work to be done. Before you get started, check out our [contributing guide](https://github.com/jasperproject/jasper-client/blob/master/CONTRIBUTING.md).
