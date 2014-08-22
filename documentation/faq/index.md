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
- __What does Jasper do with the information I provide in my profile?__
    - The information in the profile allows Jasper modules to tailor their responses and actions to you. For example, the weather module will be able to tell you about the weather in your area, and the time module will be able to report a time that's sensitive to your timezone. None of this information ever leaves the Pi (unless, of course, you create a module that sends the profile information elsewhere).
- __Why does Jasper ask for my Gmail password?__
    - As with any information requested in the profile populator, this is purely optional and will never leave the Pi. If provided, Jasper will be able to tell you when you have new emails. Additionally, Jasper will use this information to send relevant messages to your email and/or phone (e.g., links to articles that you've requested). As an alternative, consider using [Mailgun](/documentation/software/#mailgun).
- __Why do I get a segmentation fault when running `main.py`?__
    - This is usually caused by the speech dictionary not being created during the boot process. Make sure you've [added the boot script](/documentation/software/#install-client) to crontab, then restart your Pi. If a `dictionary.dic` file is still not present in your `client/` directory, just run `~/jasper/boot/boot.py` manually to force the dictionary generation.
- __Can I change Jasper's name?__
    - Sure, you can just replace the "Jasper" name in [main.py](https://github.com/jasperproject/jasper-client/blob/master/client/main.py) and [musicmode.py](https://github.com/jasperproject/jasper-client/blob/master/client/musicmode.py). Then, create a new [dictionary_persona.dic](https://github.com/jasperproject/jasper-client/blob/master/client/dictionary_persona.dic) and [languagemodel_persona.lm](https://github.com/jasperproject/jasper-client/blob/master/client/languagemodel_persona.lm) to include the new name you decide. We recommend the [online lmtool](http://www.speech.cs.cmu.edu/tools/lmtool-new.html) to regenerate the dictionary and language model. More information is available [here](https://github.com/jasperproject/jasper-client/issues/8).
- __Can Jasper understand other languages?__
    - Jasper relies on [CMUSphinx](http://cmusphinx.sourceforge.net/) for voice recognition. You can change the language by configuring CMUSphinx with a new acoustic and language model, as described in the [CMUSphinx FAQ](http://cmusphinx.sourceforge.net/wiki/faq#qwhich_languages_are_supported).
- __Can Jasper work on other platforms? (OS X, Ubuntu, VirtualBox...)__
    - Jasper is targeted at Raspberry Pi, but people have had success porting it to other platforms. Check out these threads for more information: [OS X](https://github.com/jasperproject/jasper-client/issues/35), [Ubuntu](https://github.com/jasperproject/jasper-client/issues/20). If you'd like to share your success using another platform, just [submit a pull request](https://github.com/jasperproject/jasperproject.github.io/blob/master/documentation/faq/index.md) to add to this page.
- __Can I plug in my own speech recognition engine? (Google Speech, etc)__
    - Sure, check out the instructions in [client/stt.py](https://github.com/jasperproject/jasper-client/blob/master/client/stt.py). While Pocketsphinx is the default, an example using the Google Speech API is bundled with the client.
- __Does Jasper work on Raspberry Pi B+?__
    - Method 2 in the [Software Guide](/documentation/software/) should work. Method 1 will not work because it contains Raspberry Pi B firmware.

<h2 class="linked" id='interacting'><a href="#interacting" title="Permalink to this headline">Interacting with Jasper</a></h2>

- __Why isn't Jasper responding to its name?__
    - Make sure your Raspberry Pi is turned on and has been allowed to boot for a few minutes. Try moving closer to the microphone. If all else fails, you can SSH into the Raspberry Pi and run main.py yourself to see Jasper's interpretation of your input.
- __Can I control the volume or make Jasper more sensitive to sound levels?__
    - Run `alsamixer` on your Raspberry Pi for microphone and speaker level controls. Raising the microphone level will help Jasper to hear your voice from a greater distance.
- __Why can't I login to my Raspberry Pi after running Jasper?__
    - Jasper's attempts to connect with a wifi network may be interfering with your attempts to make an ethernet connection. To resolve this, disconnect your wifi adapter, restart your Jasper, then plug in the ethernet cable only _after_ Jasper fails to find a wireless network.
- __Does Jasper work with two-factor email authentication?__
    - Sure does! You simply need to generate an application specific password especially for Jasper, and use this generated password in the setup process. See https://support.google.com/accounts/answer/185833?hl=en for more information about how to generate one.
- __What if I don't have speakers that work with the Pi or don't want Jasper to speak out loud?__
    - I'd suggest using headphones! This is particularly useful for development, during which you might have Jasper repeating himself over and over, speaking sporadically, etc.

<h2 class="linked" id='developing'><a href="#developing" title="Permalink to this headline">Developing on Jasper</a></h2>

- __What if my module's keywords conflict with those of another module?__
    - Modules are ordered using a relative priority system, so a module with higher priority will be given preference when two modules would both accept a certain input. In general, the more specific the module's `isValid` function, the higher it's priority can be, as you'll trigger fewer false positives. For more detail, see [here](/documentation/api/standard/#priorities). For an example in the code, see the ['News'](https://github.com/jasperproject/jasper-client/blob/master/client/modules/News.py) and ['Hacker News'](https://github.com/jasperproject/jasper-client/blob/master/client/modules/HN.py) modules, which would both accept the input "what's on hacker news?"; the higher priority of the ['Hacker News'](https://github.com/jasperproject/jasper-client/blob/master/client/modules/HN.py) module ensures that this input is passed to the correct module.
- __Is there any way to interact with Jasper on the command-line?__
    - Yes! Set the `JASPER_HOME` environment variable, then run `main.py` as follows: `JASPER_HOME="/home/pi" && export JASPER_HOME && python main.py --local`. The `--local` flag enables the command-line interface (CLI). Note that there's no need to type "Jasper" between commands when using the CLI, as you would when interacting through speech.
- __How do I fix the KeyError: 'JASPER_HOME' problem?__
    - JASPER_HOME is an environment variable added to later versions of the client code to be more platform-agnostic. The [boot.sh](https://github.com/jasperproject/jasper-client/blob/master/boot/boot.sh) script sets the JASPER_HOME variable on boot. If you run into this problem while trying to run Jasper manually, just set the JASPER_HOME variable before running main.py, as described in the previous question above.
- __What if my module needs more accurate information from the input text? For example, what if I need to capture a number ("one", "two", "three", etc.)? I can't list every number in the input, so what do I do?__
    - If you need more accurate speech recognition, you could modify Jasper to utilize the [AT&T speech-to-text API](https://developer.att.com/apis/speech). This will capture a larger vocabulary at the cost of being somewhat slower and rate-limited. The primary modifications would come in mic.py, which would need to change its rendering function to call the AT&T API. Alternatively, consider limiting the set of _relevant_ words that the user might say (e.g., if the user is choosing from a list of three items, just add "one", "two", and "three" to the keywords list, rather than worrying about all possible numbers).

- __What if I want to contribute to Jasper?__
    - Great! There's always work to be done. Before you get started, check out our [contributing guide](https://github.com/jasperproject/jasper-client/blob/master/CONTRIBUTING.md).
