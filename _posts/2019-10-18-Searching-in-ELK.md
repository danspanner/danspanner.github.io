---
layout: post
title: Searching for goodies with ELK
---

I wrote this up so my team could actually make use of all the effort I had put in to standardising our corporate logging, and consolidating them in ELK. Writeup on that adventure coming soon.

In the meantime, the following assumes you're logging in to the Bitnami ELK AMI, and that winlogbeat etc. is already talking to it. Right now, we have an issue, and we want to track it down. So, log in to your ELK box.

The bit we're looking for is Discover.

![discover](/img/ELK/39288899.jpg)

Welcome to the search interface!

ELK has this idea of "Beats", which are just applications that push logs into the stack. Since our DCs are Windows based, it's using a thing called "Winlogbeats", which takes parts of the event log and pushes them to Elasticsearch.

You can setup different partitions and filters, but that's not really necessary or relevant since all we're doing is consolidating our Windows DC logs. Our next step to change our filter to look for stuff Winlogbeat has pushed to ELK.

![](/img/ELK/39288900.jpg)

Click the dropdown and change it to "winlogbeat-\*"

The default timescale is 15 minutes, we can change this too if we're interested in a particular length of time (say the last day, last week, last month) or alternatively, we can give an absolute time in the form of a date range.

![](/img/ELK//39288901.jpg)

On the left are all the available fields that Winlogbeat has found in the log. There are a lot of them, and you can use any of them to filter the logs available to find whatever is of interest to you.

![](/img/ELK/39288904.jpg)

Most likely, the events you are interested in will be

*   winlog.event\_id
*   winlog.event\_data.TargetUserName

What we want to do now is add this as a filter by clicking the "Add Filter" button.

![](/img/ELK/39288905.jpg)

Select the field you want to search (in this example we'll look for a user) and a search operator.

![](/img/ELK/39288906.jpg)

So for a simple user search, we would filter for "winlog.event\_data.TargetUserName" "IS" "d.mackie"

![](/img/ELK//39288908.jpg)

Ta-da! There's all the events I've made. You can keep adding filters until you've fine-tuned your search results, perhaps by adding an event code search on top a username for example.

[Here's a handy list of codes.](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/Default.aspx)

Fingers crossed whatever you're looking for will be listed in there somewhere.

The most common problems are likely to be user account lockouts, or code 4740.
