---
layout: post
title: Cloud Setup for Data Engineering 
subtitle: Data Engineering Series - 1
#cover-img: /assets/img/flatpak.jpg
thumbnail-img: /assets/img/awslogo.jpg
share-img: /assets/img/awslogo.jpg
permalink: /blog/aws-ec2/
tags: [dataengg]
---


# Table of contents

- [Background](#bg)
- [Setting up big-data environment](#setup)
  - [Account Creation](#account)
  - [Creating EC2 Instance](#ec2)
  - [Accessing cloud via SSH from terminal](#ssh)


## Background {#bg}

I have been learning `Data Engineering` as a part of my job where I am responsible to create data infrastructure in the Organisation. so why I have to write down my learnings as blog? 

<iframe src="https://giphy.com/embed/mQ2QSLQ6UNX9K" width="480" height="261" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/kit-zooey-deschanel-gif-why-not-mQ2QSLQ6UNX9K"></a></p>

This tweet gives me clarity and the purpose of writing down our learnings. Without writing down, we'd become [Dead Neurons](https://medium.com/joelthchao/how-dead-neurons-hurt-training-5fc127d8db6a#:~:text=It%20is%20refer%20as%20%E2%80%9Cdead,value%20and%20have%20zero%20gradient.), unable to revive what we learnt after few days.


<iframe src="https://www.linkedin.com/embed/feed/update/urn:li:share:6940920507697393664" height="716" width="504" frameborder="0" allowfullscreen="" title="Embedded post"></iframe>

## Setting up big-data environment {#setup}

Data Engineering or Big Data can be setup in the following ways:
- Local setup
- Virtual Machine
- Cloud Infrastructure

In this blog, we would see how to setup cloud and access via our local terminal. In the forthcoming blogs, we would see other setups.

### Account Creation {#account}

- Go to [AWS Console](https://console.aws.amazon.com/) and create account
- If we register for the first time or with new mail ID, we are eligible for [free-tier account](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) automatically for 12 months.

### Creating EC2 Instance {#ec2}

- We create Instances in EC2 which provides a wide selection of instance types optimized to fit different use cases.
- Instance types comprise varying combinations of CPU, memory, storage, and networking capacity and give us the flexibility to choose the appropriate mix of resources for our applications.
- We can create new instance in `EC2 Dashboard` and click `Launch Instance`

![Launch Instance](https://raw.githubusercontent.com/edwardcodes/edwardcodes.github.io/main/assets/img/instance-creation.png)

- Provide `name` for the instance under `Name and Tags` section.
- Select `Amazon Linux OS` free tier in Amazon Machine Image (AMI)
- Create new key pair and download `.pem` file in local system to use for accessing cloud
- Leave the rest as it is and now, we can see our launched instance.

### Accessing cloud via SSH from terminal {#ssh}

{: .box-note}
**Note:** This can be done through [puTTY](https://www.putty.org/) too.

Copy/move the EC2 downloaded keypair from Windows folder to linux `.ssh` path. Here I named my `.pem` file as `dataengg-tamilboomi.pem`

```bash
$ mv dataengg-tamilboomi.pem ~/.ssh
```

Try to connect SSH by the following command where XX is the public IP address in the created EC2 instance.

```bash
$ ssh -i dataengg-tamilboomi.pem ec2-user@XX.XXX.XXX.XXX
```

The above steps would connect to instance or else, you would face the below error:

```bash
$ ssh -i dataengg-tamilboomi.pem ec2-user@xx.xxx.xxx.xxx
The authenticity of host 'xx.xxx.xxx.xxx (xx.xxx.xxx.xxx)' can't be established.
ED25519 key fingerprint is SHA256:Gq1KnFeFtkLreTRn3hjOE6Pq68CHWjmymF1j+bjJums.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'xx.xxx.xxx.xxx' (ED25519) to the list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for 'dataengg_tamilboomi.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "dataengg_tamilboomi.pem": bad permissions
ec2-user@xx.xxx.xxx.xxx: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```
The above error is due to file permission errors. To do solve this, follow below steps:

```bash
$ chmod 600 ~/.ssh/id_ed25519

$ sudo chmod 600 ~/.ssh/dataengg-tamilboomi.pem
```

Run again SSH with keypair and it’d connect EC2 successfully.

```bash
$ ssh -i dataengg-tamilboomi.pem ec2-user@XX.XXX.XXX.XXX

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
[ec2-user@ip-172-31-xx-xxx ~]$ du -ah
4.0K    ./.bash_logout
4.0K    ./.bash_profile
4.0K    ./.bashrc
4.0K    ./.ssh/authorized_keys
4.0K    ./.ssh
16K     .
[ec2-user@ip-172-31-87-169 ~]$ exit
logout
Connection to xx.xxx.xxx.xxx closed.
```

{: .box-warning}
**Warning:** It is wise to shut down our Instance once our job done to avoid unwanted bill from AWS.