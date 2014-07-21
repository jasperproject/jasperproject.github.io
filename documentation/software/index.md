---
title: Software
layout: default
currentpage: software
---

Software Guide
===

There are two ways to install Jasper's software on your Raspberry Pi.


<h1 class="linked" id='quick-start'><a href="#quick-start" title="Permalink to this headline">Method 1: Quick Start (Recommended)</a></h1>

The quickest way to get up and running with Jasper is to download the pre-compiled disk image [available here](http://sourceforge.net/projects/jasperproject/files/jasper-disk-image.tar.gz/download). After imaging your SD card, skip to the section on [configuring your Jasper Client](#configure-jasper).

If you want to understand how all of the supporting libraries are compiled on the Raspberry Pi, Method 2 may be to your liking (or, at the very least, helpful for debugging).

<h1 class="linked" id='manual-installation'><a href="#manual-installation" title="Permalink to this headline">Method 2: Manual Installation</a></h1>

Follow these instructions only if you wish to compile your Jasper software from scratch. These steps are unnecessary if you follow the recommended "Quick Start" instructions above.


<h2 class="linked" id='burn-image'><a href="#burn-image" title="Permalink to this headline">Burn Raspbian image onto SD card</a></h2>

We'll first clear the SD card using [GParted](http://gparted.org) on Ubuntu, but you can use an equivalent utility or operating system. In GParted: right-click on each partition of the SD card, then select 'Unmount' and 'Delete'. Apply the changes with Edit > Apply All Operations.

Download Raspbian Wheezy from [http://downloads.raspberrypi.org/raspbian_latest](http://downloads.raspberrypi.org/raspbian_latest). While we've tested Jasper on the 2014-01-07 release, newer releases may also work.

We'll use dd to burn the image to the disk. Obtain the address of the SD card with:

{% highlight bash %}
sudo fdisk -l
{% endhighlight %}

Our address was '/dev/mmcblk0', so the following command burns the image to the disk:

{% highlight bash %}
sudo dd if=2013-12-20-wheezy-raspbian.img of=/dev/mmcblk0 bs=2M
{% endhighlight %}

When it's done, remove your SD card, insert it into your Raspberry Pi and connect it to your computer via ethernet.

<h2 class="linked" id='configure-raspbian'><a href="#configure-raspbian" title="Permalink to this headline">Configure Raspbian</a></h2>

We're now going to do some basic housekeeping and install some of the required libraries. You should SSH into your Pi with a command similar to the following. The IP address usually falls in the 192.168.2.3-192.168.2.10 range.

{% highlight bash %}
ssh pi@192.168.2.3 # password (default): raspberry
{% endhighlight %}

Run the following, select to 'Expand Filesystem' and restart your Pi:

{% highlight bash %}
sudo raspi-config
{% endhighlight %}

Run the following commands to update Pi and some install some useful tools.

{% highlight bash %}
sudo apt-get update
sudo apt-get upgrade --yes
sudo apt-get install vim git-core espeak python-dev python-pip bison libasound2-dev libportaudio-dev python-pyaudio --yes
{% endhighlight %}

Plug in your USB microphone. Let's open up an ALSA configuration file in vim:

{% highlight bash %}
sudo vim /etc/modprobe.d/alsa-base.conf
{% endhighlight %}

Change the following line:

{% highlight bash %}
options snd-usb-audio index=-2
{% endhighlight %}

To this:

{% highlight bash %}
options snd-usb-audio index=0
{% endhighlight %}

Back in the shell, run:

{% highlight bash %}
sudo alsa force-reload
{% endhighlight %}

Next, test that recording works (you may need to restart your Pi) by recording some audio with the following command:

{% highlight bash %}
arecord temp.wav
{% endhighlight %}

Make sure you have speakers or headphones connected to the audio jack of your Pi. You can play back the recorded file:

{% highlight bash %}
aplay -D hw:1,0 temp.wav
{% endhighlight %}

Add the following line to the end of ~/.bash_profile (you may need to run `touch ~/.bash_profile` if the file doesn't exist already):

{% highlight bash %}
export LD_LIBRARY_PATH="/usr/local/lib"
source .bashrc
{% endhighlight %}

And this to your ~/.bashrc or ~/.bash_profile:

{% highlight bash %}
LD_LIBRARY_PATH="/usr/local/lib"
export LD_LIBRARY_PATH
PATH=$PATH:/usr/local/lib/
export PATH
{% endhighlight %}

With that, we're ready to install the core software that powers Jasper.

<h2 class="linked" id='installing-sphinx'><a href="#installing-sphinx" title="Permalink to this headline">Install Pocketsphinx</a></h2>

Jasper uses Pocketsphinx for voice recognition. Let's download and unzip the sphinxbase and pocketsphinx packages:

{% highlight bash %}
wget http://downloads.sourceforge.net/project/cmusphinx/sphinxbase/0.8/sphinxbase-0.8.tar.gz
wget http://downloads.sourceforge.net/project/cmusphinx/pocketsphinx/0.8/pocketsphinx-0.8.tar.gz
tar -zxvf sphinxbase-0.8.tar.gz
tar -zxvf pocketsphinx-0.8.tar.gz
{% endhighlight %}

Now we build and install sphinxbase:

{% highlight bash %}
cd ~/sphinxbase-0.8/
./configure --enable-fixed
make
sudo make install
{% endhighlight %}

And pocketsphinx:

{% highlight bash %}
cd ~/pocketsphinx-0.8/
./configure
make
sudo make install
{% endhighlight %}

Once the installations are complete, restart your Pi.

<h2 class="linked" id='install-binaries'><a href="#install-binaries" title="Permalink to this headline">Install CMUCLMTK, OpenFST, MIT Language Modeling Toolkit, m2m-aligner, Phonetisaurus</a></h2>

Note that some of these installation steps take more time to complete than previous steps.

Begin by installing some dependencies:

{% highlight bash %}
sudo apt-get install subversion autoconf libtool automake gfortran g++ --yes
{% endhighlight %}

Next, move into your home (or Jasper) directory to check out and install CMUCLMTK:

{% highlight bash %}
svn co https://svn.code.sf.net/p/cmusphinx/code/trunk/cmuclmtk/
cd cmuclmtk/
sudo ./autogen.sh && sudo make && sudo make install
cd ..
{% endhighlight %}

Then, when you've left the CMUCLTK directory, download the following libraries:

{% highlight bash %}
wget http://distfiles.macports.org/openfst/openfst-1.3.3.tar.gz
wget https://mitlm.googlecode.com/files/mitlm-0.4.1.tar.gz
wget https://m2m-aligner.googlecode.com/files/m2m-aligner-1.2.tar.gz
wget https://phonetisaurus.googlecode.com/files/phonetisaurus-0.7.8.tgz
wget http://phonetisaurus.googlecode.com/files/g014b2b.tgz
{% endhighlight %}

Untar the downloads:

{% highlight bash %}
tar -xvf m2m-aligner-1.2.tar.gz
tar -xvf openfst-1.3.3.tar.gz
tar -xvf phonetisaurus-0.7.8.tgz
tar -xvf mitlm-0.4.1.tar.gz
tar -xvf g014b2b.tgz
{% endhighlight %}

Build OpenFST:

{% highlight bash %}
cd openfst-1.3.3/
sudo ./configure --enable-compact-fsts --enable-const-fsts --enable-far --enable-lookahead-fsts --enable-pdt
sudo make install # come back after a really long time
{% endhighlight %}

Build M2M:

{% highlight bash %}
cd m2m-aligner-1.2/
sudo make
{% endhighlight %}

Build MITLMT:

{% highlight bash %}
cd mitlm-0.4.1/
sudo ./configure
sudo make install
{% endhighlight %}

Build Phonetisaurus:

{% highlight bash %}
cd phonetisaurus-0.7.8/
cd src
sudo make
{% endhighlight %}

Move some of the compiled files:

{% highlight bash %}
sudo cp ~/m2m-aligner-1.2/m2m-aligner /usr/local/bin/m2m-aligner
sudo cp ~/phonetisaurus-0.7.8/phonetisaurus-g2p /usr/local/bin/phonetisaurus-g2p
{% endhighlight %}

Build Phonetisaurus model:

{% highlight bash %}
cd g014b2b/
./compile-fst.sh
{% endhighlight %}

Finally, rename the following folder for convenience:

{% highlight bash %}
mv ~/g014b2b ~/phonetisaurus
{% endhighlight %}

At this point, we've installed Jasper and all the necessary software to run it. Before we start playing around, though, we need to configure Jasper and provide it with some basic information.


<h2 class="linked" id='configure-jasper'><a href="#configure-jasper" title="Permalink to this headline">Configuring the Jasper Client</a></h2>

<h3 class="linked" id='install-client'><a href="#install-client" title="Permalink to this headline">Install Jasper Client</a></h3>

In the home directory of your Pi, clone the Jasper source code:

{% highlight bash %}
git clone https://github.com/jasperproject/jasper-client.git jasper
{% endhighlight %}

Jasper requires various Python libraries that we can install in one line with:

{% highlight bash %}
sudo pip install --upgrade setuptools
sudo pip install -r jasper/client/requirements.txt
{% endhighlight %}

Run `crontab -e`, then add the following line, if it's not there already:

{% highlight bash %}
@reboot /home/pi/jasper/boot/boot.sh;
{% endhighlight %}

Set permissions inside the home directory:

{% highlight bash %}
sudo chmod 777 -R *
{% endhighlight %}

Restart your Raspberry Pi. Doing so will run `boot.py`, which generates the `languagemodel.lm` file in the `client/` folder.

<h3 class="linked" id='generating-profile'><a href="#generating-profile" title="Permalink to this headline">Generating a user profile</a></h3>

In order for Jasper to accurately report local weather conditions, send you text messages, and more, you first need to generate a user profile.

To facilitate the process, run the profile population module that comes packaged with Jasper

{% highlight bash %}
cd ~/jasper/client
python populate.py
{% endhighlight %}

The process is fairly self-explanatory: fill in the requested information, or hit 'Enter' to defer at any step. The resulting profile will be stored as a [YML](http://fdik.org/yml/) file at _profile.yml_.

**Important**: _populate.py_ will request your Gmail password. Of course, this is purely optional and will never leave the device. This password allows Jasper to report on incoming emails _and_ send you text or email notifications, but be aware that the password will be stored as plaintext in _profile.yml_. The Gmail _address_ can be entered alone (without a password) and will be used to send you notifications if you configure a Mailgun account, as described below.

<h3 class="linked" id='mailgun'><a href="#mailgun" title="Permalink to this headline">Mailgun as a Gmail alternative</a></h3>

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



<h3 class="linked" id='facebook-tokens'><a href="#facebook-tokens" title="Permalink to this headline">Facebook tokens</a></h3>

To enable Facebook integration, Jasper requires an API key. Unfortunately, this is a multi-step process that requires manual editing of _profile.yml_. Fortunately, it's not particularly difficult.

1. Go to [https://developers.facebook.com](https://developers.facebook.com) and select 'Apps', then 'Create a new app'.
2. Give your app a name, category, etc. The choices here are arbitrary.
3. Go to the [Facebook Graph API Explorer](https://developers.facebook.com/tools/explorer/) and select your App from the drop down list in the top right (the default choice is 'Graph API Explorer').
4. Click 'Get Access Token' and in the popup click 'Extended Permissions' and make sure 'manage_notifications' is checked. Now click 'Get Access Token' to get your token.
5. Take the resulting API key and add it to _profile.yml_ in the following format:

        ...
        prefers_email: false
        timezone: US/Eastern
        keys:
            FB_TOKEN: abcdefghijklmnopqrstuvwxyz


Note that similar keys could be added when developing other modules. For example, a Twitter key might be required to create a Twitter module. The goal of the profile is to be straightforward and extensible.

<h3 class="linked" id='spotify-integration'><a href="#spotify-integration" title="Permalink to this headline">Spotify integration</a></h3>

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

Having installed the required libraries, it is worth taking a moment to understand how they interact and how the client code is architected.

Jasper utilizes a number of open source libraries to function. [Pocketsphinx](http://cmusphinx.sourceforge.net/2010/03/pocketsphinx-0-6-release/) performs speech recognition via Python bindings to the [CMUSphinx](http://cmusphinx.sourceforge.net) engine. Jasper’s voice is owed to the popular TTS program, [eSpeak](http://espeak.sourceforge.net). [Phonetisaurus](https://code.google.com/p/phonetisaurus/) and CMUCLMTK enable Jasper to generate dictionaries and language models on-the-fly based on the custom module vocabularies. Mopidy enables streaming from Spotify, for those users who wish to use the module.

The client architecture is organized into a number of different components:

![Jasper Client Architecture](architecture.png)

main.py is the program that orchestrates all of Jasper. It creates mic, profile, and conversation instances. Next, the conversation instance is fed the mic and profile as inputs, from which it creates a notifier and a brain.

The brain then receives the mic and profile originally descended from main and loads all the interactive components into memory. The brain is essentially the interface between developer-written modules and the core framework. Each module must implement `isValid()` and `handle()` functions, as well as define a `WORDS = [...]` list.

To learn more about how Jasper interactive modules work and how to write your own, check out the [API guide](/documentation/api)

Next Steps
---
Now that you have fully configured your Jasper software, you're ready to start using it. Check out the [Usage](/documentation/usage) page for next steps.
