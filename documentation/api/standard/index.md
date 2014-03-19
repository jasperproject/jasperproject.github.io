---
title: Standard Module
layout: default
currentpage: api
---

How to create a Jasper standard module
===

Augmenting Jasper with a new module is a fairly simple process. With just a few lines of code, you can have Jasper telling you the weather, sending you the top headlines, etc.

## Getting Started

In this tutorial, we'll use the 'Meaning of Life' (Life.py) module as an example. Pretend this module doesn't exist and we want to implement it.

The basic functionality: _You ask Jasper about the meaning of life, and he responds with some variation on "It's 42."_

Here are the questions we need to answer in our implementation:

- What keywords do we need to recognize in the user's speech?
- What does valid input to this module look like?
- What do we do with valid input?


We'll go through them in order.

## What keywords do we need to recognize?

At this level, we're thinking about words we need to detect in the user's speech: that is, what words does the speech-to-text system need to be able to identify when the user speaks into the microphone?

As previously mentioned, we keep the speech-to-text dictionary small to improve accuracy, so __we ask that every module makes public the exact words it will need to detect from speech input__.

In terms of code, the interface requests a list of single-word strings called `WORDS` that sits in the module's global namespace. For _Life.py_, we have the following:

{% highlight python %}
WORDS = ["MEANING", "OF", "LIFE"]
{% endhighlight %}

If your module doesn't require any keywords, leave `WORDS = []` in the code.

## What does valid input look like?

At this level, we're thinking about the translated text that the module will interpret: that is, once the speech-to-text system has done its work, what valid text inputs will this module accept?

In terms of code, the interface requests a method `isValid(input)` that returns `True` if the input is valid for this module, and `False` otherwise.

For _Life.py_, we just want the user input to contain some variation of "meaning of life", such as "What's the meaning of life?" or "Tell me the meaning of life", which we accomplish as follows:

{% highlight python %}
def isValid(input):
    return bool(re.search(r'\bmeaning of life\b', input, re.IGNORECASE))
{% endhighlight %}

Any word used in the `isValid` method should be included in the `WORDS` dictionary. How can you detect a word if the speech-to-text system never identifies it?

## What do we do with valid input?

This is the heart of the module: you know you've been passed valid input, so what do you do next? The interface requires a method `handle(input, mic, profile)`, where `input` is the parsed speech, `mic` is the mic object (used for speaking and taking in additional user input), and `profile` is the user profile.

In _Life.py_, we simply spout out a response:

{% highlight python %}
def handle(input, mic, profile):
    messages = ["It's 42, you idiot.",
                "It's 42. How many times do I have to tell you?"]

    message = random.choice(messages)

    mic.say(message)
{% endhighlight %}

## User Interaction

Some modules require user interaction. For example, in the News module (_News.py_), we ask the user if they want to be emailed links to the top headlines.

In such cases, these methods will come in handy:

1. `mic.say(message)`: reads out `message` to the user.
2. `mic.activeListen()`: beeps (to alert the user that input is required) and listens until the user has finished speaking, returning the parsed speech.

Note that `mic.activeListen(NATIVE=False)` will use the AT&T speech API. Any input involving _numbers_, _dates_, or any other dynamic data should use `NATIVE=False`.

## Profile

All modules have complete access to `profile`, the user's profile. This allows you to send texts or emails, read out timezone-sensitive dates, etc. The profile is simply a dictionary (see the [Profile Guide](#Profile_Guide) for more).

A good example of using the user profile can be found in _Time.py_, where we take note of the user's timezone when reading off the time:

{% highlight python %}
def handle(input, mic, profile):
    user_tz = getTimezone(profile)
    now = datetime.datetime.now(tz=user_tz)
    response = now.strftime("%I:%M %p")
    mic.say("It is %s right now." % (response))
{% endhighlight %}

The profile is also useful for accessing permissions tokens, passwords, and more. In _Birthday.py_, we grab the user's Facebook permissions token from the profile in order to find out their friends' birthdays:

{% highlight python %}
def handle(input, mic, profile):
    oauth_access_token = profile['keys']["FB_TOKEN"]
    ...
{% endhighlight %}
