+++
date = "2015-02-08 20:09:47"
draft = "false"
title = "Scaling MySQL at SendGrid"
slug = "scaling-mysql-at-sendgrid"

+++

*Note: This post first appeared at [Sendgrid's official blog](https://sendgrid.com/blog/scaling-mysql-at-sendgrid/) *

SendGrid is the epitome of catching a tiger by the tail. Our systems were not originally designed to handle the massive scale we deal with today. Adding new features at this scale also presents challenges budding companies don’t yet need to design for. With our growth and overall traffic, we have had to come up with solutions to handle challenges related to simply scaling our datastores.

At SendGrid, a large portion of our data is housed in 10 distinct MySQL datastores with a total of 87 physical machines and 255 MySQL instances. We also have a varying combination of challenges that tend to be specific to our clusters. Let’s walk through some of these challenges and how we've tackled them.

### High transaction rate

We collect a lot of statistics on the mail we send. How often email is opened, clicked, marked as spam....etc. These stats used to live in one of our busiest MySQL clusters, peaking at 17,000 transactions per second almost every day. While a lot of our daemons in SendGrid, including our stats writer, use local queues as a method of back pressure at peak traffic times, eventually, we reached a scale that broke known limits of MySQL performance and scalability.  

A benchmark by Percona puts MySQL (with settings comparable to ours) using solid state drives peaking their throughput at 15,000 transactions per second (Benchmark by Percona). There was a very brief moment of pride for our ops team that we had a finely tuned database working as hard as it can be worked. But how do we continue scaling? 

This is when we used a strategy known as “Functional Sharding.”  Functional sharding is when you separate parts of your data set to dedicated database clusters based on common functionality in order to split read heavy data (like user login information) from write heavy data (like statistics tables) and gives the ops team the chance to tune each database separately for their specific workload.

### Insane dataset sizes

Another challenge we face at SendGrid is the sheer size of our data set. MySQL, being the go-to relational database for many startups, is typically deployed in sizes that don't exceed a few hundred gigabytes per instance. Ideally row counts are 500M or less, or in rare occasions on special hardware, 1B rows. Here at SendGrid, our largest single instance is a whopping 21 billion rows, living on 7.3 TB of disk space.

When creating the datastore that will hold information about every single link in a large portion of our email volume, data set size was immediately a concern. Not only the sheer volume of data, but also the fact that we will need to read it at an aggregated rate of 60 thousand IO operations per second and at peak times as millions of recipients open these emails and click links. We also have to keep this data for 8 weeks as people can and will click on links in emails that are that old. This brings the total data set size to 8.8 TBs of data including replicas.

So what is the solution for this? Horizontal sharding. Breaking your data into individual instances of MySQL where each instance looks the same, but the rows contained are separate. This is especially easy to do when the rows can exist independently of each other and you will not do searches across clusters that have to be grouped later. 

In order to also maximize the I/O of our solid state drives and the CPU throughput of all those cores, we housed 5 instances of MySQL per box leading to a lot more parallel reads and writes per machine and more IOPS from a single machine. This data store amounts to 75 individual MySQL instances across 44 servers. 

### How to write apps for it all

Whether it is functional or horizontal sharding, most open source object relational mappers do not support this kind of data distribution where the application has to have a context of where the information for different parts of its stack lives. At SendGrid we developed an application that does it all for us. When we started, solving problems of this scale was not common. However, now there are numerous open source projects that support this style of using MySQL. You can choose between jetpants (By Tumblr), Vitess (By YouTube), MySQL Fabric (By Oracle) and a few other open source and proprietary solutions.

## Things to come
 
Here at SendGrid, we always have a bunch of different new services in development at any given time and invariably most, if not all of them, need a datastore and a lot of times that means MySQL. What that means to the ops team is an imperative to keep our MySQL deployments automated and version controlled, and to always look ahead and design schemas and tune configurations for maximum performance and optimal use of resources. 

