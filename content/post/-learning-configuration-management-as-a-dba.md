+++
date = "2015-02-13 01:49:42"
draft = "false"
title = "Learning configuration management as a DBA"
slug = "-learning-configuration-management-as-a-dba"

+++


### How it's been	
Before SendGrid, I used to deploy all my databases by hand. I'd have a documentation page (a google doc, internal wiki page...whatever). And it would be a long bulleted list of "Install this, then install this". If you have ever maintained 'How to' documents like that, This picture will eventually ring true.

![code comments](http://s4.postimg.org/44kc215hp/code_comment.png)

This was not a good approach, obviously. Especially when small details start changing but the 'documentation' lags behind. Now you have a situation that enables tribal knowledge which means 3 am ops person who is not the DBA has even less of an ability to know what should be running on a database and how it should look like under normal operations.


### Multiply by...a lot
Then came my largest deployment to date at Sendgrid. We needed a data store for storing the click and open tracking for our short URLs and we decided to use MySQL as the place for this. This was going to be a ton of rows with a high demand on fast writes and supporting a *lot* of reads. So a single instance was not going to cut it. 

Because MySQL 5.5 was the standard GA version at the time, we were still limited to a MySQL that didn't very efficiently use all the cores newer server configuraitons could offer. So to squeeze out the most performance out of our not so cheap hardware, we also decided to house 5 MySQL instances per box. The way to do that was add a Virtual IP per instance on the box, use distinct data directories and config files per instance while still making sure that all 5 instances are 'equal' so as not to let one starve the others of system resources.

### Enter chef
As you can see where I am going with this, it became very clear to me that I could not successfully deploy this new cluster (the biggest I had done yet) using the same old method. I needed a way to automate the building of these clusters and I needed that to also be an easy method of maintaining the state of these clusters (configuration or MySQL and teh system underneath) in code.

So why chef? Simply put, it was what Sendgrid had already been using for configuration management and what is now often called 'Infrastructure as code'. I wanted this datastore to begin the effort of not making what I do seem like black magic...because it really isn't. I work with a team of great operations engineers and when trying to scale traffic to double or more annually with a not very big team, consistency of tools is of extreme importance.


### What I learned
Learning chef as a DBA was an interesting experience. I will preface this section by saying that 2 years later, I am rewriting not just the cookbook for this data cluster but all of my chef code at Sendgrid. There are many things I learned the hard way in that first major iteration and I can imagine the same pitfalls happening to others in a DBA or similar roles in other companies.

_Write your own cookbook_

I am not going to go into code samples. There are a few community cookbooks for installing MySQL/Percone Server and I consider them all a great place to find examples. Yes, you can absolutely grab them and deploy MySQL with them and I imagine for many budding teams this may be a very fine path to take. But know the debt you take on when grabbing someone else's code to deploy your infrastructure. I chose from the very beginning to write my own cookbook because by the time I started, Sendgrid was already doing a huge throughput and that comes with a number of tweaks.

 _When things are similar but not the same_

When I started on this cookbook writing adventure, I thought my different database clusters were similar enough to use one cookbook with just some attribute differences. And maybe when I strated 2+ years ago that was true. Very quickly though, as we sharded more tables into their own clusters, plus added a few more brand new projects using MySQL, that stopped being true. I found myself maintaining the monorail of database cookbooks. Making its testing strategy truly comprehensive meant 3 test kitchen suites *per database kind*. Build times grew exponentially. 

This is why in this rewrite I am heavily using what is basically a wrapper style. Yes, most of my MySQL deployments use what is more or less the same pattern, but usually in the post server install time, things diverge. And there are few things as frustrating as watching a multi hour jenkins build because I changed a config file for a specific database type.

_Embrace your organisation's cookbook hierarchy_

Besides automation, making my life easier...etc. First and foremost, I decided to learn chef and write cookbooks for our databases because _database land should not be an island_. This is why in my rewrites I made sure the operations engineering team reviewed my code. Not only is peer review from them, being immersed in chef the most, so useful. They also know what parts of system management we decided to turn into internal lightweight resources, making my code even simpler and no reinventing my mostly the same but not quite wheel. This has made the rewritten cookbooks much easier to follow and maintain.


This rewrite is not done. I only have a few clusters left with cookbooks in progress for them already. I have learned quite a lot about being a operations engineer working on this project.