---
title: Installation
layout: default
currentpage: installation
---

Software Install Guide
===

There are three ways to install Jasper on your Raspberry Pi.


<h1 class="linked" id='quick-start'><a href="#quick-start" title="Permalink to this headline">Method 1: Quick Start (Recommended)</a></h1>

The quickest way to get up and running with Jasper is to download the pre-compiled disk image [available here for Model B](http://sourceforge.net/projects/jasperproject/files/jasper-disk-image.tar.gz/download). There is also an unofficial image for the B+ [available here](https://groups.google.com/forum/#!topic/jasper-support-forum/xHgonA-TZYw). After imaging your SD card, clone the repository and install the Python dependencies as described in [Install Jasper](#install-jasper). Then, skip to [Configuration](/documentation/configuration/).

If you want to understand how all of the supporting libraries are compiled on the Raspberry Pi, Method 3 may be to your liking (or, at the very least, helpful for debugging).

<h1 class="linked" id='package-manager-installation'><a href="#package-mangager-installation" title="Permalink to this headline">Method 2: Installation via Package Manger</a></h1>

<h2 class="linked" id='debian-packages'><a href="#debian-packages" title="Permalink to this headline">Debian/Raspbian</a></h2>

Unfortunately, there are currently no packages available for Debian or Raspbian. Please use [Method 3](#manual-installation).

<h2 class="linked" id='archlinux-packages'><a href="#archlinux-packages" title="Permalink to this headline">ArchLinux/ArchLinuxARM</a></h2>

If you're using ArchLinux, there are packages available in the [Arch User Repository](https://aur.archlinux.org/packages/jasper-voice-control-git/). To install them:
{% highlight bash %}
yaourt -S jasper-voice-control-git
yaourt -S jasper-plugins
{% endhighlight %}

You'll also need a Text-to-Speech (TTS) and a Speech-to-Text (STT) engine. Check out the configuration section to learn what STT/TTS engines are and what you need to do to use them.

After you've done that, you can start Jasper as a systemd service:
{% highlight bash %}
sudo systemctl start jasper-voice-control
{% endhighlight %}

If the systemd service keeps failing, your audio device might already be in use by MPD, Pulseaudio, your Desktop Environment or some other process. In this case, start Jasper as your current user:
{% highlight bash %}
mkdir -p ~/.jasper
cp -r /var/lib/jasper/.jasper/profile.yml ~/.jasper
jasper-voice-control
{% endhighlight %}

<h1 class="linked" id='manual-installation'><a href="#manual-installation" title="Permalink to this headline">Method 3: Manual Installation</a></h1>

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

Run the following commands to update Pi and install some useful tools.

{% highlight bash %}
sudo apt-get update
sudo apt-get upgrade --yes
sudo apt-get install nano git-core python-dev bison libasound2-dev libportaudio-dev python-pyaudio --yes
sudo apt-get remove python-pip
sudo easy_install pip
{% endhighlight %}

Plug in your USB microphone. Let's create an ALSA configuration file:

{% highlight bash %}
sudo nano /lib/modprobe.d/jasper.conf
{% endhighlight %}

Add the following lines:

{% highlight bash %}
# Load USB audio before the internal soundcard
options snd_usb_audio index=0
options snd_bcm2835 index=1

# Make sure the sound cards are ordered the correct way in ALSA
options snd slots=snd_usb_audio,snd_bcm2835
{% endhighlight %}

Back in the shell, restart your Pi:

{% highlight bash %}
sudo shutdown -r now
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

With that, we're ready to install Jasper.

<h2 class="linked" id='install-jasper'><a href="#install-jasper" title="Permalink to this headline">Install Jasper</a></h2>

In the home directory of your Pi, clone the Jasper source code:

{% highlight bash %}
git clone https://github.com/jasperproject/jasper-client.git jasper
{% endhighlight %}

Jasper requires various Python libraries that we can install in one line with:

{% highlight bash %}
sudo pip install --upgrade setuptools
sudo pip install -r jasper/client/requirements.txt
{% endhighlight %}

Sometimes it might be neccessary to make <tt>jasper.py</tt> executable:

{% highlight bash %}
chmod +x jasper/jasper.py
{% endhighlight %}

You've now installed the Jasper core software. If you're following Method I (Quick Start), continue with [Configuration](/documentation/configuration/). Otherwise, continue with the dependency installation below.

<h2 class="linked" id='installing-dependencies'><a href="#installing-dependencies" title="Permalink to this headline">Installing dependencies</a></h2>

To be able to understand what you say, Jasper still needs a Speech-to-Text (STT) engine. Jasper also needs a Text-to-Speech (TTS) engine to answer to your commands. Jasper aims to be modular and thus gives you the choice which STT/TTS engine you want to use. Depending on your choice, it may be required to install additional software.

Head over to the [Configuration section](/documentation/configuration/). During configuration, you'll learn what STT/TTS engines are and chose your flavour. You can then come back here and install the required dependencies for the STT/TTS engine of your choice (if neccessary).

<h3 class="linked" id='installing-sphinx'><a href="#installing-sphinx" title="Permalink to this headline">Install Dependencies for PocketSphinx STT engine</a></h3>

*Note: Installing pocketsphinx will take quite some time because you need to compile some stuff from source.*

Jasper can use PocketSphinx for voice recognition. If you want to use Pocketsphinx as [STT Engine](/documentation/configuration/#stt), you'll have to install:

- sphinxbase & pocketsphinx
- CMUCLMTK
- MIT Language Modeling Toolkit
- m2m-aligner
- OpenFST & Phonetisaurus

If you're using ArchLinux, you're lucky: Just install the [according AUR package](https://aur.archlinux.org/packages/jasper-stt-pocketsphinx/) and you're done:
{% highlight bash %}
yaourt -S jasper-stt-pocketsphinx
{% endhighlight %}

Everyone else needs to install the above tools manually:

<h4>Installing Sphinxbase/Pocketsphinx</h4>

First, you need to install Pocketsphinx. If you're using Debian Sid (unstable) or Jessie (testing), you can just do:
{% highlight bash %}
sudo apt-get update
sudo apt-get install pocketsphinx python-pocketsphinx
{% endhighlight %}

If you're not using Debian Sid/Jessie, you need to compile and install them from source:
{% highlight bash %}
wget http://downloads.sourceforge.net/project/cmusphinx/sphinxbase/0.8/sphinxbase-0.8.tar.gz
tar -zxvf sphinxbase-0.8.tar.gz
cd ~/sphinxbase-0.8/
./configure --enable-fixed
make
sudo make install
wget http://downloads.sourceforge.net/project/cmusphinx/pocketsphinx/0.8/pocketsphinx-0.8.tar.gz
tar -zxvf pocketsphinx-0.8.tar.gz
cd ~/pocketsphinx-0.8/
./configure
make
sudo make install
cd ..
sudo easy_install pocketsphinx
{% endhighlight %}

<h4>Installing CMUCLMTK</h4>
Begin by installing some dependencies:

{% highlight bash %}
sudo apt-get install subversion autoconf libtool automake gfortran g++ --yes
{% endhighlight %}

Next, move into your home (or Jasper) directory to check out and install CMUCLMTK:

{% highlight bash %}
svn co https://svn.code.sf.net/p/cmusphinx/code/trunk/cmuclmtk/
cd cmuclmtk/
./autogen.sh && make && sudo make install
cd ..
{% endhighlight %}

Then, when you've left the CMUCLTK directory, download the following libraries:

<h4>Installing Phonetisaurus, m2m-aligner and MITLM</h4>

To use the Pocketsphinx STT engine, you also need to install MIT Language Modeling Toolkit, m2m-aligner and Phonetisaurus (and thus OpenFST).

On Debian, you can install these from the `experimental` repository:
{% highlight bash %}
sudo su -c "echo 'deb http://ftp.debian.org/debian experimental main contrib non-free' > /etc/apt/sources.list.d/experimental.list"
sudo apt-get update
sudo apt-get -t experimental install phonetisaurus m2m-aligner mitlm libfst-tools
{% endhighlight %}

If you're not using Debian, perform these steps:

{% highlight bash %}
wget http://distfiles.macports.org/openfst/openfst-1.3.4.tar.gz
wget https://github.com/mitlm/mitlm/releases/download/v0.4.1/mitlm_0.4.1.tar.gz
wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/m2m-aligner/m2m-aligner-1.2.tar.gz
wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/phonetisaurus/is2013-conversion.tgz
{% endhighlight %}

Untar the downloads:

{% highlight bash %}
tar -xvf m2m-aligner-1.2.tar.gz
tar -xvf openfst-1.3.4.tar.gz
tar -xvf is2013-conversion.tgz
tar -xvf mitlm_0.4.1.tar.gz
{% endhighlight %}

Build OpenFST:

{% highlight bash %}
cd openfst-1.3.4/
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
cd is2013-conversion/phonetisaurus/src
sudo make
{% endhighlight %}

Move some of the compiled files:

{% highlight bash %}
sudo cp ~/m2m-aligner-1.2/m2m-aligner /usr/local/bin/m2m-aligner
sudo cp ~/is2013-conversion/bin/phonetisaurus-g2p /usr/local/bin/phonetisaurus-g2p
{% endhighlight %}

<h4>Building the Phonetisaurus FST model</h4>

{% highlight bash %}
wget https://www.dropbox.com/s/kfht75czdwucni1/g014b2b.tgz
tar -xvf g014b2b.tgz
{% endhighlight %}

Build Phonetisaurus model:

{% highlight bash %}
cd g014b2b/
./compile-fst.sh
cd ..
{% endhighlight %}

Finally, rename the following folder for convenience:

{% highlight bash %}
mv ~/g014b2b ~/phonetisaurus
{% endhighlight %}

Once the installations are complete, restart your Pi.

At this point, we've installed Jasper and all the necessary software to run it. Before we start playing around, though, we need to configure Jasper and provide it with some basic information.

<h3 class="linked" id='installing-julius'><a href="#installing-julius" title="Permalink to this headline">Install Dependencies for Julius STT engine</a></h3>

On Arch Linux, install [`julius` from the `[community]` repo](https://www.archlinux.org/packages/?repo=Community&q=julius):
{% highlight bash %}
sudo pacman -S julius
{% endhighlight %}

If you're not using ArchLinux, you need to compile Julius manually.

{% highlight bash %}
sudo apt-get update
sudo apt-get install build-essential zlib1g-dev flex libasound2-dev libesd0-dev libsndfile1-dev
{% endhighlight %}

Then, download the [Julius source tarball](http://sourceforge.jp/projects/julius/downloads/60273/julius-4.3.1.tar.gz/) and extract it to `~/julius`.

{% highlight bash %}
cd ~/julius
./configure --enable-words-int
make
sudo make install
{% endhighlight %}

Please note that you also need an [acoustic model and a lexicon.](/documentation/configuration/#julius-stt).

<h3 class="linked" id='installing-espeak'><a href="#installing-espeak" title="Permalink to this headline">Install Dependencies for eSpeak TTS engine</a></h3>

On Arch Linux, install [`jasper-tts-espeak` from the AUR](https://aur.archlinux.org/packages/jasper-tts-espeak/):
{% highlight bash %}
yaourt -S jasper-tts-espeak
{% endhighlight %}

On Debian, install the `espeak` package:
{% highlight bash %}
sudo apt-get update
sudo apt-get install espeak
{% endhighlight %}

<h3 class="linked" id='installing-festival'><a href="#installing-festival" title="Permalink to this headline">Install Dependencies for Festival TTS engine</a></h3>

On Arch Linux, install [`jasper-tts-festival` from the AUR](https://aur.archlinux.org/packages/jasper-tts-festival/):
{% highlight bash %}
yaourt -S jasper-tts-festival
{% endhighlight %}

On Debian, install `festival` and `festvox-don`:
{% highlight bash %}
sudo apt-get update
sudo apt-get install festival festvox-don
{% endhighlight %}

<h3 class="linked" id='installing-festival'><a href="#installing-flite" title="Permalink to this headline">Install Dependencies for Flite TTS engine</a></h3>

On Arch Linux, install [`jasper-tts-flite` from the AUR](https://aur.archlinux.org/packages/jasper-tts-flite/):
{% highlight bash %}
yaourt -S jasper-tts-flite
{% endhighlight %}

On Debian, install `flite`:
{% highlight bash %}
sudo apt-get update
sudo apt-get install flite
{% endhighlight %}

<h3 class="linked" id='installing-pico'><a href="#installing-pico" title="Permalink to this headline">Install Dependencies for SVOX Pico TTS engine</a></h3>

On Arch Linux, install [`jasper-tts-pico` from the AUR](https://aur.archlinux.org/packages/jasper-tts-pico/):
{% highlight bash %}
yaourt -S jasper-tts-pico
{% endhighlight %}

On Debian, you need to install `libttspico-utils`:
{% highlight bash %}
sudo apt-get update
sudo apt-get install libttspico-utils
{% endhighlight %}

<h3 class="linked" id='installing-googletts'><a href="#installing-googletts" title="Permalink to this headline">Install Dependencies for Google TTS engine</a></h3>

On Arch Linux, install [`jasper-tts-google` from the AUR](https://aur.archlinux.org/packages/jasper-tts-google/):
{% highlight bash %}
yaourt -S jasper-tts-google
{% endhighlight %}

On Debian, you need to install `python-pymad` via APT and `gTTS` via PIP:
{% highlight bash %}
sudo apt-get update
sudo apt-get install python-pymad
sudo pip install --upgrade gTTS
{% endhighlight %}

<h3 class="linked" id='installing-ivonatts'><a href="#installing-ivonatts" title="Permalink to this headline">Install Dependencies for Ivona TTS engine</a></h3>

On Arch Linux, install [`jasper-tts-ivona` from the AUR](https://aur.archlinux.org/packages/jasper-tts-ivona/):
{% highlight bash %}
yaourt -S jasper-tts-ivona
{% endhighlight %}

On Debian, you need to install `python-pymad` via APT and `pyvona` via PIP:
{% highlight bash %}
sudo apt-get update
sudo apt-get install python-pymad
sudo pip install --upgrade pyvona
{% endhighlight %}
