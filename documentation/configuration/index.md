---
title: Configuration
layout: default
currentpage: configuration
---

Configuring Jasper
===

<h3 class="linked" id='generating-profile'><a href="#generating-profile" title="Permalink to this headline">Generating a user profile</a></h3>

Jasper needs a configuration file that we call "profile". In order for Jasper to accurately report local weather conditions, send you text messages, and more, you first need to generate a user profile.

To facilitate the process, run the profile population module that comes packaged with Jasper

{% highlight bash %}
cd ~/jasper/client
python populate.py
{% endhighlight %}

The process is fairly self-explanatory: fill in the requested information, or hit 'Enter' to defer at any step. By default, the resulting profile will be stored as a [YML](http://fdik.org/yml/) file at `~/.jasper/profile.yml`.

**Important**: _populate.py_ will request your Gmail password. Of course, this is purely optional and will never leave the device. This password allows Jasper to report on incoming emails _and_ send you text or email notifications, but be aware that the password will be stored as plaintext in _profile.yml_. The Gmail _address_ can be entered alone (without a password) and will be used to send you notifications if you configure a Mailgun account, as described below.


<h3 class="linked" id='stt'><a href="#stt" title="Permalink to this headline">Choosing an STT engine</a></h3>
You need to choose which Speech-To-Text (STT) engine Jasper should use. An STT engine is basically a piece of software that takes recorded speech and transforms it into written text. If you say "foo", but Jasper understands "bar", it's either a problem with your microphone or a bad or misconfigured STT engine. So choosing the right STT engine is crucial to use Jasper correctly. While most speech-recognition tools only rely on one single STT engine, Jasper tries to be modular and thus offers a wide variety of STT engines:

- **[Pocketsphinx](#pocketsphinx-stt)** is a open-source speech decoder by the [CMU Sphinx](http://en.wikipedia.org/wiki/CMU_Sphinx) project. It's fast and designed to work well on embedded systems (like the Raspberry Pi). Unfortunately, the recognition rate is not the best and it has a lot of depencies. On the other hand, recognition will be performed offline, i.e. you don't need an active internet connection to use it. It's the right thing to use if you're cautious with your personal data.
- **[Google STT](#google-stt)** is the speech-to-text system by [Google](http://www.google.com/intl/en/chrome/demos/speech.html). If you have an Android smartphone, you might already be familiar with it, because it's basically the same engine that performs recognition if you say *OK, Google*. It can only transcribe a limited amount of speech a day and needs an active internet connection.
- **[AT&T STT](#att-stt)** is a speech decoder by the telecommunications company [AT&T](http://developer.att.com/apis/speech). Like Google Speech, it also performs decoding online and thus needs an active internet connection.
- **[Wit.ai STT](#witai-stt)** relies on the [wit.ai cloud services](https://wit.ai/) and uses crowdsourcing to train speech recognition algorithms. Like you'd expect from a cloud service, you also need an active internet connection.
- **[Julius](#julius-stt)** is a [high-performance open source speech recognition engine](http://julius.sourceforge.jp/en_index.php). It does not need an active internet connection. Please note that you will need to train your own acoustic model, which is a very complex task that we do not provide support for. Regular users are most likely better suited with one of the other STT engines listed here.

**Important:** Except *PocketSphinx* and *Julius*, all of the above STT engines transfer the microphone data over the internet. **If you don't want Google, Wit.ai and AT&T to be able to listen to everything you say, do not use these STT engines!** In this case, use *PocketSphinx* instead.

We also do not recommend using Internet-connected STT engines for passive listening. From privacy and performance standpoints, Pocketsphinx/Julius are superior solutions.

<h4 class="linked" id='pocketsphinx-stt'><a href="#pocketsphinx-stt" title="Permalink to this headline">Configuring the Pocketsphinx STT engine</a></h4>

[Install the required software](/documentation/installation/#installing-sphinx). Then, locate your FST model (`g014b2b.fst`) and your Hidden Markov Model directory (`hub4wsj_sc_8k`). If the paths below are incorrect, add the correct paths to your `profile.yml`:

{% highlight yaml %}
stt_engine: sphinx
pocketsphinx:
  fst_model: '../phonetisaurus/g014b2b.fst'                              #optional
  hmm_dir: '/usr/share/pocketsphinx/model/hmm/en_US/hub4wsj_sc_8k' #optional
{% endhighlight %}

<h4 class="linked" id='julius-stt'><a href="#julius-stt" title="Permalink to this headline">Configuring the Julius STT engine</a></h4>

[Install julius](/documentation/installation/#installing-julius).

You also need an acoustic model in HTK format. Although VoxForge offers speaker-independent models, you will have to [adapt the model and train it with you voice](http://www.voxforge.org/home/dev/acousticmodels/linux/adapt/htkjulius) to get good recognition results. Please note that we do not offer support for this step. If you need help, ask in the [respective forums](http://www.voxforge.org/home/forums). This stt engine also needs a lexicon file that maps words to phonemes and has to contain all words that Julius shall be able to recognize. A very comprehensive lexicon is the [VoxForge Lexicon](http://www.repository.voxforge1.org/downloads/SpeechCorpus/Trunk/Lexicon/VoxForge.tgz).

After creating your own acoustic model, you have to specify the paths to your `hmmdefs`, `tiedlist` and lexicon file in your `profile.yml`:

{% highlight yaml %}
stt_engine: julius
julius:
  hmmdefs:  '/path/to/your/hmmdefs'
  tiedlist: '/path/to/your/tiedlist'
  lexicon:  '/path/to/your/lexicon.tgz'
  lexicon_archive_member: 'VoxForge/VoxForgeDict' # only needed if lexicon is a tar/tar.gz archive
{% endhighlight %}

If you encounter errors or warnings like `voca_load_htkdict: line 19: triphone "r-d+v" not found`, you are trying to recognize words that contain phones are not in your acoustic model. To be able to recognize these words, you need to train them in your acoustic model.

<h4 class="linked" id='google-stt'><a href="#google-stt" title="Permalink to this headline">Configuring the Google STT engine</a></h4>

You need an Google Speech API key to use this. To obtain an API key,
join the [Chromium Dev group](https://groups.google.com/a/chromium.org/forum/?fromgroups#!forum/chromium-dev) and create a project through the
[Google Developers console](https://console.developers.google.com/project).

Then select your project. In the sidebar, navigate to "APIs & Auth." and activate
the Speech API. Under "APIs & Auth," navigate to "Credentials." Create a new key for
public API access.

Add your credentials to your `profile.yml`:

{% highlight yaml %}
stt_engine: google
keys:
    GOOGLE_SPEECH: 'your_google_speech_api_key_here_xyz1234'
{% endhighlight %}

<h4 class="linked" id='att-stt'><a href="#att-stt" title="Permalink to this headline">Configuring the AT&T STT engine</a></h4>

The AT&T STT engine requires an AT&T app_key/app_secret to be present in
profile.yml. Please sign up at http://developer.att.com/apis/speech and
create a new app. You can then take the app_key/app_secret and put it into
your `profile.yml`:

{% highlight yaml %}
stt_engine: att
att-stt:
  app_key:    4xxzd6abcdefghijklmnopqrstuvwxyz
  app_secret: 6o5jgiabcdefghijklmnopqrstuvwxyz

{% endhighlight %}

<h4 class="linked" id='witai-stt'><a href="#witai-stt" title="Permalink to this headline">Configuring the Wit.ai STT engine</a></h4>

This implementation requires an Wit.ai Access Token to be present in
profile.yml. Please sign up at https://wit.ai and copy your instance
token, which can be found under Settings in the Wit console to your
profile.yml:

{% highlight yaml %}
stt_engine: witai
witai-stt:
  access_token:    ERJKGE86SOMERANDOMTOKEN23471AB
{% endhighlight %}

<h3 class="linked" id='tts'><a href="#tts" title="Permalink to this headline">Choosing a TTS engine</a></h3>
A TTS engine does the exact opposite of an [STT engine](#stt): It takes written text and transforms it into speech. Jasper supports many different TTS engines that differ by voice, intonation, "roboticness" and so on.

- **[eSpeak](#espeak-tts)** is a compact [open-source speech synthesizer](http://espeak.sourceforge.net/) for many platforms. Speech synthesis is done offline, but most voices can sound very "robotic".
- **[Festival](#festival-tts)** uses the [Festival Speech Synthesis System](http://www.cstr.ed.ac.uk/projects/festival/), an open source speech synthesizer developed by the Centre for Speech Technology Research at the University of Edinburgh. Like *eSpeak*, also synthesizes speech offline.
- **[Flite](#flite-tts)** uses [CMU Flite (festival-lite)](http://www.festvox.org/flite/), a  lightweight and fast synthesis engine that was primarily designed for small embedded machines. It synthesizes speech offline, so no internet connection is required.
- **[SVOX Pico TTS](#pico-tts)** was the Text-to-Speech engine used in [Android 1.6 "Donut"](http://betanews.com/2009/09/15/android-donut-sdk-released-what-s-new-inside/). It's an open-source small footprint application and also works offline. The quality is rather good compared to
*eSpeak* and *Festival*.
- **[Google TTS](#google-tts)** uses the same Text-to-Speech API which is also used by newer Android devices. The Synthesis itself is done on Google's servers, so that you need an active internet connection and also can't expect a lot of privacy if you use this.
- **[Ivona TTS](#ivona-tts)** uses Amazon's Ivona Speech Cloud service, which is used in the [Kindle Fire](http://www.engadget.com/2013/01/24/amazon-ivona/). Speech synthesis is done online, so an active internet connection and Amazon has access to everything Jasper says to you.
- **[MaryTTS](#mary-tts)** is an open-source TTS system written in Java. You need to set up your own MaryTTS server and configure Jasper to use it. Because the server can be hosted on the same machine that runs Jasper, you do not need internet access.
- **[Mac OS X TTS](#osx-tts)** does only work if you're running Jasper on a Mac. It then uses the `say` command in MacOS to synthesize speech.

**Important:** If you're using *Google STT* (or *Mary TTS* with someone else's server), everything Jasper says will be sent over the internet. **If you don't want Google or someone else to be able to listen to everything Jasper says to you, do not use these TTS engines!**

<h4 class="linked" id='espeak-tts'><a href="#espeak-tts" title="Permalink to this headline">Configuring the eSpeak TTS engine</a></h4>

[Install eSpeak](/documentation/installation/#installing-espeak) and choose `espeak-tts` as your TTS engine in your `profile.yml`:

{% highlight yaml %}
tts_engine: espeak-tts
{% endhighlight %}

Further customization is also possible by tuning the `voice`, `pitch_adjustment` and `words_per_minute` options in your `profile.yml`:

{% highlight yaml %}
espeak-tts:
  voice: 'default+m3'   # optional
  pitch_adjustment: 40  # optional
  words_per_minute: 160 # optional
{% endhighlight %}

<h4 class="linked" id='festival-tts'><a href="#festival-tts" title="Permalink to this headline">Configuring the Festival TTS engine</a></h4>

You need to [install festival (and festival's voices)](/documentation/installation/#installing-festival). If you've done that, you can set `festival-tts` as you TTS engine in your `profile.yml`:

{% highlight yaml %}
tts_engine: festival-tts
{% endhighlight %}

If you change the default voice of festival, Jasper will use this voice as well.

<h4 class="linked" id='flite-tts'><a href="#flite-tts" title="Permalink to this headline">Configuring the Flite TTS engine</a></h4>

[Install Flite](/documentation/installation/#installing-flite) and add it to your `profile.yml`:

{% highlight yaml %}
tts_engine: flite-tts
{% endhighlight %}

If you want to use another voice (e.g. 'slt'), specify it in your `profile.yml`:

{% highlight yaml %}
flite-tts:
  voice: 'slt'
{% endhighlight %}

To get a list of available voices, run `flite -lv` on the command line.

<h4 class="linked" id='pico-tts'><a href="#pico-tts" title="Permalink to this headline">Configuring the SVOX Pico TTS engine</a></h4>

[Install Pico](/documentation/installation/#installing-pico) Then, you just add it to your `profile.yml`:

{% highlight yaml %}
tts_engine: pico-tts

{% endhighlight %}

<h4 class="linked" id='google-tts'><a href="#google-tts" title="Permalink to this headline">Configuring the Google TTS engine</a></h4>

[Install the required dependencies](/documentation/installation/#installing-googletts) for Google TTS. Then set `google-tts` as you TTS engine in your `profile.yml`:

{% highlight yaml %}
tts_engine: google-tts
{% endhighlight %}

<h4 class="linked" id='ivona-tts'><a href="#ivona-tts" title="Permalink to this headline">Configuring the Ivona TTS engine</a></h4>

[Install the required dependencies](/documentation/installation/#installing-ivonatts) for accessing Amazon's Ivona Speech Cloud service. You'll also need to [sign up for free](https://www.ivona.com/us/account/speechcloud/creation/) to use their service. Then set `ivona-tts` as your TTS engine in your `profile.yml` and also paste your Ivona Speech Cloud keys:

{% highlight yaml %}
tts_engine: ivona-tts
ivona-tts:
  # Keys can be obtained via:
  # https://www.ivona.com/us/account/speechcloud/creation/
  access_key: 'access_key' # required
  secret_key: 'secret_key' # required
  voice: 'Eric'            # optional, default is 'Brian'
  region: 'eu-west'        # optional, default is 'us-east'
  speech_rate: 'medium'    # optional
  sentence_break: 400      # optional
{% endhighlight %}

<h4 class="linked" id='mary-tts'><a href="#mary-tts" title="Permalink to this headline">Configuring the Mary TTS engine</a></h4>

Simply set `mary-tts` as you TTS engine in your `profile.yml`. If you want, you can also change the default server to your own MaryTTS server:

{% highlight yaml %}
tts_engine: mary-tts
mary-tts:
  server: 'mary.dfki.de'
  port: '59125'
  language: 'en_GB'
  voice: 'dfki-spike'
{% endhighlight %}

**Note:** Currently, the demo server at [mary.dfki.de:59129](http://mary.dfki.de:59125/) is not working, so you need to set up your own MaryTTS server (which you can [download here](http://mary.dfki.de/download/index.html)).

<h4 class="linked" id='osx-tts'><a href="#osx-tts" title="Permalink to this headline">Configuring the Mac OS X engine</a></h4>

Make sure that you're about to run Jasper on a Mac. Look at the casing of your computer, there should a bitten apple symbol on it. Then set `osx-tts` as you TTS engine in your `profile.yml`:

{% highlight yaml %}
tts_engine: osx-tts
{% endhighlight %}

<h3 class="linked" id='misc'><a href="#misc" title="Permalink to this headline">Other things to configure</a></h3>

<h4 class="linked" id='mailgun'><a href="#mailgun" title="Permalink to this headline">Mailgun as a Gmail alternative</a></h4>

If you'd prefer not to enter your Gmail password, you can setup a free [Mailgun](https://mailgun.com/) account that Jasper will use to send you notifications. It's incredibly painless and Jasper is already setup for instant Mailgun integration. Note that if you don't enter your Gmail _address_, however, Jasper will only be able to send you notifications by text message (as he won't know your email address).

In slightly more detail:

1. Register for a free Mailgun account.
2. Navigate to the "Domains" tab and click on the sandbox server that should be provided initially.
3. Click "Default Password" and choose a password.
4. Take note of the "Default SMTP Login" email address.
5. Edit your _profile.yml_ to read:

       ...
       mailgun:
           username: postmaster@sandbox95948.mailgun.org
           password: your_password
6. Enjoy your notifications.

<h4 class="linked" id='international-weather'><a href="#international-weather" title="Permalink to this headline">Non-US weather data</a></h4>
If you want to use the Weather module, but you don't live in the US, [find out the WMO ID of you local weather station](http://www.wunderground.com/about/faq/international_cities.asp) (last column of the table). The WMO ID is a unique number to identify weather stations and is issued by the [World Meteorological Organization (WMO)](https://www.wmo.int/pages/index_en.html).

Then, add the WMO ID to your profile:
{% highlight yaml %}
wmo_id: 10410    #if you live in Essen (Germany) or surrounding area
{% endhighlight %}

If both `location` and `wmo_id` are in your `profile.yml`, the `wmo_id` takes precedence.

<h4 class="linked" id='facebook-tokens'><a href="#facebook-tokens" title="Permalink to this headline">Facebook tokens</a></h4>

To enable Facebook integration, Jasper requires an API key. Unfortunately, this is a multi-step process that requires manual editing of _profile.yml_. Fortunately, it's not particularly difficult.

1. Go to [https://developers.facebook.com](https://developers.facebook.com) and select 'Apps', then 'Create a new app'.
2. Give your app a name, category, etc. The choices here are arbitrary.
3. Go to the [Facebook Graph API Explorer](https://developers.facebook.com/tools/explorer/) and select your App from the drop down list in the top right (the default choice is 'Graph API Explorer').
4. Click 'Get Access Token' and in the popup click 'Extended Permissions' and make sure 'manage_notifications' is checked. Now click 'Get Access Token' to get your token.
5. (Optional) You'll probably want to extend your token's expiration date. Make a call to the following endpoint to receive a longer-lasting token: 
        https://graph.facebook.com/oauth/access_token?client_id=YOUR_APP_ID&client_secret=YOUR_APP_SECRET&grant_type=fb_exchange_token&fb_exchange_token=YOUR_CURRENT_ACCESS_TOKEN 
6. Take the resulting API key and add it to _profile.yml_ in the following format:

        ...
        prefers_email: false
        timezone: US/Eastern
        keys:
            FB_TOKEN: abcdefghijklmnopqrstuvwxyz


Note that similar keys could be added when developing other modules. For example, a Twitter key might be required to create a Twitter module. The goal of the profile is to be straightforward and extensible.

<h4 class="linked" id='spotify-integration'><a href="#spotify-integration" title="Permalink to this headline">Spotify integration</a></h4>

Jasper has the ability to play playlists from your Spotify library. This feature is optional and requires a Spotify Premium account. To configure Spotify on Jasper, just perform the following steps.

Install Mopidy with:

{% highlight bash %}
wget -q -O - http://apt.mopidy.com/mopidy.gpg | sudo apt-key add -
sudo wget -q -O /etc/apt/sources.list.d/mopidy.list http://apt.mopidy.com/mopidy.list
sudo apt-get update
sudo apt-get install mopidy mopidy-spotify --yes
{% endhighlight %}

We need to enable IPv6:

{% highlight bash %}
sudo modprobe ipv6
echo ipv6 | sudo tee -a /etc/modules
{% endhighlight %}

Now run `sudo vim /root/.asoundrc`, and insert the following contents:

{% highlight bash %}
pcm.!default {
        type hw
        card 1
}
ctl.!default {
        type hw
        card 1
}
{% endhighlight %}

We need to create the following new file and delete the default startup script:

{% highlight bash %}
sudo mkdir /root/.config
sudo mkdir /root/.config/mopidy
sudo rm /etc/init.d/mopidy
{% endhighlight %}

Now let's run `sudo vim /root/.config/mopidy/mopidy.conf` and insert the following

{% highlight bash %}
[spotify]
username = YOUR_SPOTIFY_USERNAME
password = YOUR_SPOTIFY_PASSWORD

[mpd]
hostname = ::

[local]
media_dir = ~/music

[scrobbler]
enabled = false

[audio]
output = alsasink
{% endhighlight %}

Finally, let's configure crontab to run Mopidy by running `sudo crontab -e` and inserting the following entry:

{% highlight bash %}
@reboot mopidy;
{% endhighlight %}

Upon restarting your Jasper, you should be able to issue a "Spotify" command that will enter Spotify mode. For more information on how to use Spotify with your voice, check out the [Usage](/documentation/usage) guide.

<h2 class="linked" id='software-architecture'><a href="#software-architecture" title="Permalink to this headline">Software Architecture</a></h2>

Having installed and configured Jasper and its required libraries, it is worth taking a moment to understand how they interact and how the code is architected.

The program is organized into a number of different components:

![Jasper Client Architecture](architecture.png)

`jasper.py` is the program that orchestrates all of Jasper. It creates mic, profile, and conversation instances. Next, the conversation instance is fed the mic and profile as inputs, from which it creates a notifier and a brain.

The brain then receives the mic and profile originally descended from main and loads all the interactive components into memory. The brain is essentially the interface between developer-written modules and the core framework. Each module must implement `isValid()` and `handle()` functions, as well as define a `WORDS = [...]` list.

To learn more about how Jasper interactive modules work and how to write your own, check out the [API guide](/documentation/api)

Next Steps
---
Now that you have fully configured your Jasper software, you're ready to start using it. Check out the [Usage](/documentation/usage) page for next steps.
