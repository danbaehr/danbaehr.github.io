---
layout: post
title: "Configure a new job in Jenkins"
date: 2016-04-29
---

Alright, the Jenkins server is up and running on our EC2 instance. Time to create a new job. 

For the purposes of this post (and since it seems like a typical setup) I'll usea  github repository for the new job. The plan is point our new
Jenkins job at a repo in my github repository and have it automatically do a build and test anytime we update the repo. I've been toying around 
with AMQP and the RabbitMQ implementation so I'll use the Python client library "Pika" as our repo. 

The latest Jenkins server already has the git plugin installed but if not, add it. Also add the Cobertura and Violations plugins. Then create your new project.

On the configuration page for the project go to the Source Code Management tab and select Git. For your Repository URL point it at your github repo. I had to install git here with yum.

![Image of SCM screen](https://danbaehr.github.io/images/SCM_screen2.png)

Now we need to set up our build trigger to watch the repo for changes and start our build whenever something happens. Check "Poll SCM" and
for the schedule add *****. It uses crontab formatting. 

![Image of build triggers](https://danbaehr.github.io/images/build_triggers.png)

We need to define what is actually going to be done when a build is triggered. We do that via the build tab. Select "Execute shell" and add the folowing to the command.

```bash
PYTHONPATH=''
nosetests --with-xunit --all-modules --traverse-namespace --with-coverage --cover-package=pika --cover-inclusive
```

You can look at the man page for nosetests to understand each argument here but the basic idea is that we're running nose to scan all modules for tests and generate a coverage report using the coverage module. 

If we trigger a build right now (by making a change in our git repo) we would see the build run and fail, but we would see it start! The reasons it would fail is because we're missing a bunch of packages and we don't have a rabbitMQ server running. So let's focus on that for a bit.

We're going to have to install a bunch of python packages and the best way to do that is using the Python package manager 'pip' so let's install that first using yum. Unfortunately, pip isn't in the default repos for yum so we need to add the EPEL repo first.

```bash
[ec2-user@ip-x ~]$ wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
--2016-05-02 13:19:10--  https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
Resolving dl.fedoraproject.org (dl.fedoraproject.org)... 209.132.181.23, 209.132.181.24, 209.132.181.25, ...
Connecting to dl.fedoraproject.org (dl.fedoraproject.org)|209.132.181.23|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14432 (14K) [application/x-rpm]
Saving to: ‘epel-release-latest-7.noarch.rpm’

100%[================================================================================================>] 14,432      --.-K/s   in 0.05s   

2016-05-02 13:19:11 (289 KB/s) - ‘epel-release-latest-7.noarch.rpm’ saved [14432/14432]

[ec2-user@ip-x ~]$ sudo rpm -Uvh epel-release-latest-7*.rpm
warning: epel-release-latest-7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:epel-release-7-6                 ################################# [100%]
```
Now we can install pip:
```bash
sudo yum install python-pip
```
Ok, we're ready to start installing packages. We want to install each of the following packages using pip (sudo pip install package_name).

* nose
* coverage
* pylint
* mock
* tornado

Once those are installed we'll install and start the rabbitMQ server.

```bash
wget  https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm’
sudo rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
sudo yum install rabbitmq-server-3.6.1-1.noarch.rpm’
sudo chkconfig rabbitmq-server 
sudo service rabbitmq-server start
```

Alright, the rabbitMQ server is running, which pika will need, and all the needed python packages are installed so let's trigger a build. Touch a file in the repo and a new build will start. 

You'll see your build start on the main job page and you can watch the actual console output to see what the build is currently doing or to see why it failed. 
![Image of build console output](https://danbaehr.github.io/images/build_console_output.png)

If we did everything right then everything should pass and the test should be marked blue. Git checked out our repo, ran nosetests to execute all the unit tests available in pika, and generated a code overage report using the coverage module. 

Notice when you look at your build status that you aren't being shown much yet. It's showing a basic pass/fail status. We can improve that by adding in a couple more build steps.

We'll add in pylint and we'll display our coverage report. Add the following to the build steps to the job and trigger a new build again.
```bash
pylint -f parseable -d I0011,R0801 pika | tee pylint.out
python -m coverage xml --include=pika*
```
The -d flag to pylint simply disables the categories with those IDs.

Now we need to tell Jenkins to interpret the results we've generated and display them for each build.

Go to the "Post-build Actions" tab. Add a new "Publish Cobertura Coverage Report" action. Add "coverage.xml" to the report pattern box. 
![Image of Cobertura](https://danbaehr.github.io/images/cobertura.png)

Next, add a "Publish JUnit test result report" action and add "nosetests.xml" in the "Test report XMLs" field.
![Image of junit](https://danbaehr.github.io/images/junit.png)

Lastly, add a "Report Violations" action. Add "**/pylint.out" to the pylint box in the "XML filename pattern" field.
![Image of Violations](https://danbaehr.github.io/images/violations.png)

Click save and trigger a new build! Once the build is complete you should see new output on the build details page and the main job page. If you click the "Cobertura Coverage Report" link on the build status page you'll see the detailed code coverage report that was generated. 

![Image of code coverage report](https://danbaehr.github.io/images/code_coverage_report.png)
