+++
date = "2016-12-07 18:38:21"
draft = "false"
title = "On being on call"
slug = "on-being-on-call"

+++

In case you don't follow [SysAdvent](https://sysadvent.blogspot.com/) (in which case, why?? you are missing out on great content every year!), the post on December 6th this year was by [Alice Goldfuss](https://sysadvent.blogspot.com/2016/12/day-6-no-more-on-call-martyrs.html). I have tweeted a link to that post that day and since then my tweet has been quoted, responded to and the ensuing 'conversation' has been eyeopening.

A lot of the response has been anger that an operations engineer is advocating for putting developers on call. A lot were quick to suggest that this is a quick way to lose engineers, that it advocates a terrible quality of life, that it stomps on work/life balance and that it ultimately doesn't serve to improve code quality.

The most surprising response was suggesting that this call comes from the privilege of not having family/kids responsibilities which I suppose implies only the single people are in ops and carrying pagers.

I can go on and on about how little empathy I have seen from people who are supposed to be fellow engineers towards their fellow human ops people

I can go on and on about how many naively suggested that testing and 'QA' are sufficient to not need the people who wrote the code to own their work in the real world environment of a system of scale

I can go on and on about what it says about people who presumes Alice meant "give the developer a pager and send off into the night"...I suppose that's fine when you do it to the ops gal instead.

There is a lot to unpack in these responses, many of which disappointing from engineers I respect, but I want to pinpoint 2 things

What does it mean to put engineers on call

NO it does not mean spite and no it is not punishment for bugs and no it is not a call for dissolving the lines between life and work. I have written about my experience with burnout before and I wish that on no one.

What it means to put engineers who write the code on call is that they are the subject matter experts on what is running in production. They know when error foo happens it means the cache layer failed, DNS is taking too long or maybe that the other microservice in that product comprised of about a dozen of them is the one silently failing. You may think an architecture diagram can make stuff like this clear as day but when YOU, the person on the team who has been building this thing for 2 quarters has to look at your own architecture diagram one quarter later to grok which microservice has failed you will realize why you should be paged first.

This does not mean at all that Ops wants nothing to do with your application paging out at 3 AM. We put our delivery engineering teams (the term 'delivery' here is deliberate and very appropriate) on call but they always have an escalation path to ops that is not gated by a timer. Ops still has your back. If you think this is a network problem you are not familiar with, you can immediately page the on call ops engineer and she will help verify where the broken zeros and ones are.

This is not an effort to spite those who dare push a bug to production. Anyone who has been responsible for production environments knows that will happen. Hell, you don't even HAVE to introduce bugs through a deploy, really. Unforeseen changes in your environment will cause presumptions to stop being valid and take both those who wrote the code and those with less knowledge of it (Ops) by surprise.

What this does mean is that we are explicitly saying that the code is useless unless it provides the customers the value they paid for. And when a C level executive has to explain to customers why an outage took X time to resolve, "we couldn't get the engineering team involved for some time" is not an acceptable answer.

Your code and my servers and databases are a risk center unless we are both invested in making them run. It is as simple as that.

What does it mean to be a senior engineer

It honestly frightened me that people who are senior engineers are balking so hard at giving developers pagers. Maybe they fell for the hyperbole of "we now don't want developers to sleep either", maybe they've been on call in difficult environments before and that's the PTSD talking. But I do not believe that any engineer should be called 'senior', by title or by implication if they refuse to be reachable in a team rotation in case their own work caused a customer facing issue.

Note my words because I am trying my best to choose them carefully. 'customer facing issue'. You can't give people pagers without making sure you are not paging on bullshit signals that aren't actually affecting the customers and the business. And surprise, ops engineers are people too.

So what does this mean? If you fail to see how your code is more than the sum of its functions and test. That it needs to provide real value. If you insist that someone else be the first line of defense when it breaks, then you are failing to acknowledge that as a senior engineer, your job is more than producing code. You are setting a terrible example for junior engineers on your team. They will now learn the lesson of "I don't have to own that my code provides value once it's in production". The damage that mentality will do to an engineering organization is lasting and will take a long time before anyone realizes it is the root for a LOT of tech debt and it takes years to also reverse.

If you are a manager promoting engineers to senior titles and an emphasis on ownership, including stability, is not a non-negotiable criterion of that promotion then you are damaging your organization. And if you think that a sense of ownership and truly understanding what it takes to create stable systems of scale can happen without ever being paged by a service in production then I am not sure why you are in charge of engineers supposedly building such systems of scale.

Finally, no one is saying this all ignores our lives outside work. In fact, the opposite. Ops engineers have lives too. We are men and women with spouses and babies who get sick and sometimes are solo parenting. Mature teams have each others' backs. Mature teams will override set on call shifts when life strikes and that is applauded.

What I (and I think Alice) are saying is "do not systematically make it acceptable to throw code at the production wall"....not let's page developers at 3 AM out of spite.
