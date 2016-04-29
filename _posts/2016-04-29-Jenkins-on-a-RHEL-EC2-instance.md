---
layout: post
title: "Installing Jenkins on a RHEL EC2 instance"
date: 2016-04-29
---

So I set about to learn how to use the CI tool Jenkins for automated build and test. More specifically, use Jenkins to do build and test for a Python project. I had recently setup a free AWS account and I didn't really want to dig out an old Linux box at home, so I figured an EC2 instance would do just fine. 

First thing I did was to create an EC2 RHEL instance. I specifically wanted to get it up and running on RHEL so I chose not to use the Amazon Linux instance, which I think is Ubuntu based. 

![Image of AWS micro instance](https://danbaehr.github.io/images/micro_instance.png)


Go through and choose all the default options and then create and launch your instance. You should ultimately see your new RHEL instance up and running. You should also have downloaded a key pair file to use later when connecting to your VM.

![Running instance](https://danbaehr.github.io/images/running_instance_1.png)

Now find the security group associated with your instance and change the inbound connections for SSH and HTTP to be allowed on your IP. Leaving this as "Anywhere" lets your instance be wide open to the evil lurking on the interwebs and can result in a nice letter from Amazon asking you to stop launching DDOS attacks from your client. Besides, HTTP and SSH add in a custom TCP rule for port 8080. Jenkins will listen on that port.

![Security group](https://danbaehr.github.io/images/security_groups.png)

Alright, sweet, now you should be able to ssh to your instance as the ec2-user using the public DNS name and your key file (creating a new user is probably a good idea though). If you get an error about the permissions of the key file then do a chmod to set it to 400. Now you can start installing Jenkins. 

First up is to add the Jenkins repo for yum. 'wget' isn't installed by default in RHEL 7 apparently so install that quickly. 

```bash
[ec2-user@ip-x ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo 
sudo: wget: command not found

[ec2-user@ip-x ~]$ sudo yum install wget
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
...
Downloading packages:
wget-1.14-10.el7_0.1.x86_64.rpm                                                                                    | 546 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : wget-1.14-10.el7_0.1.x86_64                                                                                            1/1 
  Verifying  : wget-1.14-10.el7_0.1.x86_64                                                                                            1/1 

Installed:
  wget.x86_64 0:1.14-10.el7_0.1                                                                                                           

Complete! 
```

Now you can get the Jenkins repo.

```bash
[ec2-user@ip-x ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
--2016-04-25 17:02:01--  http://pkg.jenkins-ci.org/redhat/jenkins.repo
Resolving pkg.jenkins-ci.org (pkg.jenkins-ci.org)... 52.91.151.148
Connecting to pkg.jenkins-ci.org (pkg.jenkins-ci.org)|52.91.151.148|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 75
Saving to: ‘/etc/yum.repos.d/jenkins.repo’

100%[================================================================================================>] 75          --.-K/s   in 0s      

2016-04-25 17:02:02 (12.9 MB/s) - ‘/etc/yum.repos.d/jenkins.repo’ saved [75/75]

[ec2-user@ip-x ~]$ sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
```

and finally install Jenkins.

```bash
[ec2-user@ip-172-31-17-2 ~]$ sudo yum install jenkins
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
jenkins                                                                                                            | 2.9 kB  00:00:00     
jenkins/primary_db                                                                                                 |  81 kB  00:00:00     
...
Total download size: 63 M
Installed size: 63 M
Is this ok [y/d/N]: y
Downloading packages:
jenkins-2.0-1.1.noarch.rpm                                                                                         |  63 MB  00:00:07     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : jenkins-2.0-1.1.noarch                                                                                                 1/1 
  Verifying  : jenkins-2.0-1.1.noarch                                                                                                 1/1 

Installed:
  jenkins.noarch 0:2.0-1.1                                                                                                                

Complete!
```

Jenkins needs Java so lets get that too.

```bash
[ec2-user@ip-172-31-17-2 ~]$ sudo yum install java
...
Installed:
  java-1.8.0-openjdk.x86_64 1:1.8.0.91-0.b14.el7_2                                                                                        

Complete!
```

Start the Jenkins daemon. 

```bash
[ec2-user@ip-172-31-17-2 ~]$ sudo service jenkins start
Starting jenkins (via systemctl):                          [  OK  ]
```

Now you can browse to port 8080 on your instance and you should see your Jenkins server!


