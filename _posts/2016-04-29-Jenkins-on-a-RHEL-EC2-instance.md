---
layout: post
title: "Installing Jenkins on a RHEL EC2 instance"
date: 2016-04-29
---

So I set about to learn how to use the CI tool Jenkins for automated build and test. More specifically, use Jenkins to do build and test for a Python project. I had recently setup a free AWS account and I didn't really want to dig out an old Linux box at home, so I figured an EC2 instance would do just fine. 

First thing I did was to create an EC2 RHEL instance. I specifically wanted to get it up and running on RHEL so I chose not to use the Amazon Linux instance, which I think is Ubuntu based. 

![Image of AWS micro instance](https://danbaehr.github.io/images/micro_instance.png)


Go through and choose all the default options and then create and launch your instance. You should ultimately see your new RHEL instance up and running. You should also have downloaded a key pair file to use later when connecting to your VM.

[[images/running_instance_1.png]]

Now find the security group associated with your instance and change the inbound connections for SSH and HTTP to be allowed on your IP. Leaving this as "Anywhere" lets your instance be wide open to the evil lurking on the interwebs and can result in a nice letter from Amazon asking you to stop launching DDOS attacks from your client. Besides, HTTP and SSH add in a custom TCP rule for port 8080. Jenkins will listen on that port.

[[images/security_groups.png]]

Alright, sweet, now you should be able to ssh to your instance as the ec2-user using the public DNS name and your key file (creating a new user is probably a good idea though). If you get an error about the permissions of the key file then do a chmod to set it to 400. Now you can start installing Jenkins. 

First up is to add the Jenkins repo for yum. 'wget' isn't installed by default in RHEL 7 apparently so grab that first. 

'''shell
[ec2-user@ip-172-31-17-2 ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo [ec2-user@ip-172-31-17-2 ~]$ 
sudo: wget: command not found
[ec2-user@ip-172-31-17-2 ~]$ sudo yum install wget
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
rhui-REGION-client-config-server-7                                                                                 | 2.9 kB  00:00:00     
rhui-REGION-rhel-server-releases                                                                                   | 3.7 kB  00:00:00     
rhui-REGION-rhel-server-rh-common                                                                                  | 3.8 kB  00:00:00     
(1/5): rhui-REGION-client-config-server-7/x86_64/primary_db                                                        | 5.4 kB  00:00:00     
(2/5): rhui-REGION-rhel-server-rh-common/7Server/x86_64/updateinfo                                                 |  27 kB  00:00:00     
(3/5): rhui-REGION-rhel-server-rh-common/7Server/x86_64/group                                                      |  104 B  00:00:00     
(4/5): rhui-REGION-rhel-server-rh-common/7Server/x86_64/primary_db                                                 |  96 kB  00:00:00     
(5/5): rhui-REGION-rhel-server-releases/7Server/x86_64/primary_db                                                  |  20 MB  00:00:00     
(1/2): rhui-REGION-rhel-server-releases/7Server/x86_64/group_gz                                                    | 134 kB  00:00:00     
(2/2): rhui-REGION-rhel-server-releases/7Server/x86_64/updateinfo                                                  | 1.1 MB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package wget.x86_64 0:1.14-10.el7_0.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================
 Package               Arch                    Version                            Repository                                         Size
==========================================================================================================================================
Installing:
 wget                  x86_64                  1.14-10.el7_0.1                    rhui-REGION-rhel-server-releases                  546 k

Transaction Summary
==========================================================================================================================================
Install  1 Package

Total download size: 546 k
Installed size: 2.0 M
Is this ok [y/d/N]: y
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
'''
