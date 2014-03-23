---
title: Notification Module
layout: default
currentpage: api
---

How to create a Jasper notification module
===

The classic example of a Notification module is one that silently monitors your email address and alerts you when you have a new email. Notification module run in the background, so you can use other modules while Jasper is monitoring your emails, for example.



<h2 class="linked" id='architecture'><a href="#architecture" title="Permalink to this headline">The Basic Architecture</a></h2>

As Notification modules run in the background, they require their own threads. The main idea is that these Notifier's have some operation in which they scrape or check for notifications, which are then stored in a thread-safe Queue and, soonafter, reported by Jasper.

All the code for notifications can be found in _Notifier.py_.

<h2 class="linked" id='NotificationClient'><a href="#NotificationClient" title="Permalink to this headline">NotificationClient</a></h2>

Each Notification module module is wrapped in a `NotificationClient`. The notification client stores two instance variables: a method `gather` that queues up notifications, and some sort of timestamp argument `timestamp`. The requirement is that `gather` returns a new value for `timestamp`, as exhibited by the `run` method:

{% highlight python %}
def run(self):
    self.timestamp = self.gather(self.timestamp)
{% endhighlight %}

For example, with email, the timestamp argument is the time of the most recent email seen. On each iteration, we return an updated timestamp to make sure we don't report new emails multiple times.

For a Twitter module, the timestamp argument might be the ID of the most recent Tweet reported. On each iteration, we'd return an updated ID to make sure that we don't report the same Tweet multiple times.

<h2 class="linked" id='Notifier'><a href="#Notifier" title="Permalink to this headline">Notifier</a></h2>

The _Notifier_ stores a bunch of separate `NotificationClient` objects in a list, `self.notifiers`. Every few seconds, the Notifier executes each of the `NotificationClient` objects and reads off any notifications that are stored in the global queue.

For each `NotificationClient`, you need to provide a method that:

1. Queues up notifications in `self.q`, a thread-safe queue of global notifications that are reported to the user.
2. Returns the new value of its timestamp variable.

For example, we have the following method for checking email:

{% highlight python %}
def handleEmailNotifications(self, lastDate):
    # grab emails
    emails = Gmail.fetchUnreadEmails(self.profile, since=lastDate)
    if emails:
        lastDate = Gmail.getMostRecentDate(emails)

    # notifications read to user as provided
    def styleEmail(e):
        return "New email from %s." % Gmail.getSender(e)

    # put notifications in queue
    for e in emails:
        self.q.put(styleEmail(e))

    # return timestamp of most recent email
    return lastDate
{% endhighlight %}

To run this Notification module on start-up, we'd then have to amend `self.notifiers` to include a new `NotificationClient` with the function `handleEmailNotifications` as its `gather` and `None` as its initial timestamp. The exact code:

{% highlight python %}
self.notifiers = [
    self.NotificationClient(self.handleEmailNotifications, None)]
{% endhighlight %}

<h2 class="linked" id='conclusion'><a href="#conclusion" title="Permalink to this headline">Conclusion</a></h2>

As a recap, there are two high-level steps to adding a new notification module:

1. Write a `gather` function that:
    - Aggregates notifications (based on a timestamp) and places them in the global queue.
    - Returns a new value for its timestamp.
2. Add another `NotificationClient` to `self.notifiers` containing your method and an initial timestamp.

Jasper will continually execute all of the Notification modules stored in `self.notifiers`, so if your `gather` function works correctly, then you're good to go.
