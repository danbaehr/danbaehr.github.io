---
layout: post
title: "Configure a new job in Jenkins"
date: 2016-04-29
---

Alright, the Jenkins server is up and running on our EC2 instance. Time to create a new job. 

For the purposes of this post (and since it seems like a typical setup) I'll usea  github repository for the new job. The plan is point our new
Jenkins job at a repo in my github repository and have it automatically do a build and test anytime we update the repo. I've been toying around 
with AMQP and the RabbitMQ implementation so I'll use the Python client library "Pika" as our repo. 

The latest Jenkins server already has the git plugin installed but if not, add it. Also add the Cobertura and Violations plugins. Then create 
your new project.

On the configuration page for the project go to the Source Code Management tab and select Git. For your Repository URL point it at your github repo.

![Image of SCM screen](https://danbaehr.github.io/images/SCM_screen.png)

Now we need to set up our build trigger to watch the repo for changes and start our build whenever something happens. Check "Poll SCM" and
for the schedule add *****. It uses crontab formatting. 

![Image of build triggers](https://danbaehr.github.io/images/build_triggers.png)


