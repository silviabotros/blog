+++
date = "2016-03-25 03:17:11"
draft = "false"
title = "Capacity planning for databases"
slug = "capacity-planning-databases"

+++

_Note:_ This is inspired by Julia Evans' recent post about ....[capacity planning](http://jvns.ca/blog/2016/03/20/how-do-you-do-capacity-planning/) 😌


### Ground rules

#### RDBMS
Yes..this post is geared for those of us who use MySQL with a single writer at a time and 2 or more read replicas. A lot of what I will talk about here applies differently, or not at all, to multi writer clustered datastores, although those also come with their own set of compromises and caveats. So...your milage will definitely vary.

#### Sharding

I have already covered large strokes of this in one of my [earlier posts](https://blog.dbsmasher.com/2015/02/08/scaling-mysql-at-sendgrid/), I mostly focused there on the benefits of functional or horizontal sharding. Yes that is a prerequisite, since what you use to access the database layer WILL decide how much flexibility you have to scale.

If you are a company that experiences large differences between peak and average traffic, you should be prepared to leave the paradigm of 'the database' as a single physical entity behind.

#### Ability to split reads and writes

This is something you will need to be able to do, but not necessarily enforce as a set in stone rule. There will be use cases where a write needs to be read very soon after and where tolerance for things like lag/eventual consistency is low. Those are ok to have, but in the same applications, you will also have scenarios for reads that *can* tolerate some longer time span of eventual consistency. When such reads are in high volume, do you really want that volume going to your single writer if it doesn't really have to? Do yourself a favor, and make sure soon in your growth days that you can control the use of a read or write IP in your code.

Now onto the thought process of actual capacity planning...

**A database cluster is not keeping up. what do I do?**

#### Determine the system bottleneck

* Is the issue high CPU?
* Is it IO capacity?
* Is it growing lag without a clear query culprit?
* Is is locks?
* How do I know which it is?

#### You need a baseline
Once you know what system metric you are mostly bound to, you need to establish baseline and peak values. Otherwise, determining whether your current issue is a bug vs real growth is going to be a lot more error prone than you'd like.

Basic server metrics can only go so far but at some point you will find you also need context based metrics. Query performance and app side perceived performance will tell you what the application sees as a response time to queries.

#### Learn your business traffic patterns
Are you a business that is susceptible to peaks in specific weekdays (marketing)? do you have regular launches that triple or quadruple your traffic like gaming? These sorts of questions will drive how much of reserved headroom you should keep or whether you need to invest in elastic growth.  

#### Determine the ratio of raw traffic numbers in relation to capacity in use
This is simply the answer to "If we made no code optimizations, how many emails/sales/whatever" can we serve with the database instance we have right now?

Ideally, this a specific value that makes the math towards planning a year's growth a simple math equation. But life is never ideal and this value will vary depending on season or completely external happy factors like signing up a new major customer. In early startups this number is a faster moving target but it should stabilize as the company transitions from early days to more established business with more predictable business growth patterns.

#### Do I really need to buy more machines?
You need to find a way to determine if this is truly capacity (I need to split the writes to support more concurrent write load or add more read replicas) vs code based performance bottleneck (this new query can really have its results cached in something cheaper and not beat the database as much).

How do you do that? You need to get familiar with your queries. The baby step for that is a combination of [innotop](http://innotop.googlecode.com/svn/html/manual.html), slow log and the [percona toolkit](https://www.percona.com/software/mysql-tools/percona-toolkit)'s pt-query-digest. You can automate this by shipping the DB logs to a central location and automating the digest portion.

But that is also not the entire picture, slow logs are performance intensive if you lower their threshold too much. If you need less selective sampling you will need to detect the entire conversations between the application and the datastore. In open source land you can go as basic as tcpdump or you can use hosted products like datadog, newrelic or vivid cortex.

#### Make a call
Capacity planning can be 90% science and 10% art but that 10% shouldn't mean that we shouldn't strive for as much of the picture as we can. As engineers we can sometimes fixate on the missing 10% and not realize that if we did the work, that 90% can get us v far into a better idea of our stack's health, a more efficient use of our time optimizing performance and planning capacity increases carefully which eventually results in much better return on investment for our products. 
