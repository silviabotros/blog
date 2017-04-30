+++
date = "2015-11-03 21:31:47"
draft = "false"
title = "Using Sensu for DBA tasks"
slug = "using-sensu-for-dba-tasks"

+++

### Sensu for monitoring
Here at Sendgrid we spent the last couple of years porting a lot of our service and host monitoring to [Sensu](https://sensuapp.org/). Its solid API support meant we could write all sorts of tooling around it. We also liked the idea of standalone, client side checks that push their status to the Sensu alerting queue asynchronously. If you are new to Sensu or haven't ever read on it, [this is a good place to start](https://sensuapp.org/docs/latest/overview).

### Typical usage example

The typical use I have for such standalone checks is health checks, a simple example looks like this in Sensu's client config

```
{
  "checks": {
    "mysql_alive": {
      "command": "mysql-alive.rb -h <IP> -d mysql -u :::mysql.user|sensu::: -p :::mysql.password:::",
      "handlers": [
        "default"
      ],
      "standalone": true,
      "interval": 10,
      "notification": "OMG MySQL is dead!"
    }
  }
}
```

But if you look closer, all you really do is tell Sensu to run a command. So this can be...any command. This will be useful in a just a moment.

### What I am solving

I have been traditionally using the crond service for running local management tasks on databases like rotating partitions and triggering backups. Along with that, we have a report that would check the logs of these jobs, on each backup replica, and make sure they ended in success messages. This is fragile due to a number of reasons:

* CronD doesn't have any built in monitoring. It does not have a concept of 'stale'
* Those reports - I will call them 'watchers' - are one more moving part adding complexity to the question 'when was the last successful backup of my DB?'
* This setup is prone to race conditions. You must time the task and its watcher in cron exactly or else the watcher can preemptively trigger an alert or signal failure when the backup is not done yet. Any drift in duration of the task will eventually make this happen (like a backup taking longer as a database grows).
* What if the watcher script didn't run? It is also in CronD - either on the same host or on another host, right? Either you find yourself in a rabbit hole of who watches the watchers, or a human has to notice that a report didn't come out...humans aren't good at remembering things.
* Changing the designation of a server means you must change it in a number of places or the watcher will watch the wrong host.
* We are striving to [keep our stack boring](http://mcfunley.com/choose-boring-technology). While newer technologies like chronos and rundeck provide more enhanced scheduling, they also need a service discovery layer to do this right. That was a bigger undertaking and too much scope creep for what I was solving.

So I decided to make Sensu work to my advantage, with the help of chef roles.

### Partition rotation in Sensu

I started off by moving any credentials I need for my partition rotation script into Sensu's redacted configuration. This is good practice for _anything_ you put into sensu that uses secrets. The credentials are added in `/etc/sensu/client.json` then are just referenced in check configurations using `:::secret_thingie:::` notation.

*_Protip_*: Sensu doesn't know what to redact in client.json. You must also define the name of the keys you want redacted. like this..

```
"redact": [
  "other_password",
  "password"
]
```
This is also not a deep merged list. As you can see I had to explicitly include 'password' once I needed to add another key in that list.

Then I needed to define the new Sensu check that rotates the partitions. I am using a chef resource as the code example.

```
sensu_check 'add_table_partitions' do
  command "/usr/local/pdb/bin/pdb-parted --add --interval d +7d.startof h=localhost,u=specialdbuser,p=:::redacted_partition_password:::,D=mydb,t=special_table >> /var/log/partition_rotation.log 2>&1"
  handlers %w(default_handler special_dba_handler)
  interval 86400
  standalone true
  additional(:occurrences => 3, :notification => "#{node['hostname']} failed adding partitions to important table. See log file in /var/log/mail_send_cancel_pause.log")
end
```

Let's inspect what just happened here...

`command`:  [pdb-parted](https://github.com/palominodb/PalominoDB-Public-Code-Repository/blob/master/tools/data_mgmt/t/pdb-parted/pdb-parted.t) is a very useful perl script by the folks from Palominodb (now at [Pythian](http://www.pythian.com/)) for rotating partitions in a MySQL DB. This is the same line I used to maintain in a crontab file configuration.

`interval`: how often this Sensu check runs. In this example these are daily partitions so running the script daily was sufficient. *The important part here is that your script is idempotent*. pdb-parted is. If it finds that the needed partitions already exists, it just outputs a message to that effect and exits nicely.

`occurrences`: This is the number of allowed failures before alerting. It is nice to have a buffer especially when I already configure the command to make partitions days/weeks in advance.  

For those who use tools other than chef and want to see what the final check configuration looks like, here it is, important bits redacted:

```
{
  "checks": {
    "add_table_partitions": {
      "command": "/usr/local/pdb/bin/pdb-parted --add --interval m +7m.startof h=localhost,u=specialdbuser,p=:::redacted_partition_password:::,D=mydb,t=special_table >> /var/log/partition_rotation.log 2>&1",
      "handlers": [
        "default_handler",
        "special_dba_handler"
      ],
      "standalone": true,
      "interval": 86400,
      "occurrences": 3,
      "notification": "hostname failed adding partitions to special table. See log file in /var/log/partition_rotation.log"
    }
  }
}
```

Partition rotation now has built in monitoring. If this script exits with a non zero code, the Sensu handlers take care of getting this knowledge to the predefined group of people using the method we want. We already have multiple on-call team rotations set up and escalation paths and all the other stuff that comes along with monitoring at scale. So we got to just leverage all of that rather than reinventing the wheel.

### Planned future improvements

No solution is perfect. Neither is this one.

* We still run the risk of more than one machine in a cluster holding the primary role. Until service discovery is in place, this is something we can build a check for leveraging chef search.
* Now that tasks like backups are running in a first class citizen tool in our stack, I can more easily add stats and get graphs for how long backups take, how big backups are to make capacity planning and MTTR tracking easier.


