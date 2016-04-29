---
layout: post
title: "Installing Jenkins on a RHEL EC2 instance"
date: 2016-04-29
---

So I set about to learn how to use the CI tool Jenkins for automated build and test. More specifically, use Jenkins to do build and test for a Python project. I had recently setup a free AWS account and I didn't really want to dig out an old Linux box at home, so I figured an EC2 instance would do just fine. 

First thing I did was to create an EC2 RHEL instance. I specifically wanted to get it up and running on RHEL so I chose not to use the Amazon Linux instance, which I think is Ubuntu based. 

[[images/micro_instance.png]]

