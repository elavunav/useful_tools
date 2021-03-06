# Remapping Elastic IP Addresses

### The Problem

AWS Elastic IP addresses are pretty cool. You can move them to different servers to help with service fail-over with minimal disruption, for example.

However, a number of our customers are still running EC2 Classic instances (instances outside of a VPC).  In that scenario, whenever an instance is shutdown, its Elastic IP gets disassociated. This means that when the instance restarts, it is operating at a different IP and may result in loss of access to the applications running there.

Instances running within a VPC do not suffer from this problem. The EIP stays associated with the instance even on shutdown.

So, what can be done to help customers still running EC2 Classic instances.


### A Solution

There are a number of approaches that can be taken to workaround this problem.

1. One could use ***dynamic DNS***, where every time the instance restarts the new IP address gets dynamically updated in the DNS zone tables. This is an elegant which avoids the need for Elastic IP's altogether. However, these solutions often cost money. Pricing is based upon the number domains managed, the number of requests per month, other services provided (like seconeary DNS services) and number of users, or some combination of all of these.


2. One could use a DevOps tool like Jenkins, Chef, Puppet or Saltstack to do these assignments, based upon a configuration management database. Those are great and capable tools.


3. One could use a simple approach leveraging the **EC2 API Tools** and instance **userData** to do it. 


What is shown here is this last approach for **Ubuntu** servers only at this time.


This solution was adapted from a shell script for **CentOS** written by **Jeffrey M. Hunter**  [http://www.idevelopment.info]


### How It Works


#### Step 1 - Create an Elastic IP (If You Haven't Already)

Go to the AWS console or use the API or CLI tools to create an elastic IP for your instance.


#### Step 2 - Add UserData to Your Instance

You have the ability to add userData to your AWS instances. However, the instances must be shutdown to add/edit this data.

You will need to add the following userData:

*userData {instance_id} elastic-ip={aaa.bbb.ccc.ddd} | hostname={myserver.mycompany.com}*

This will be used to associate both the Elastic IP and hostname you want with your instance.


#### Step 3 - Upload the Script and Template To the Instance(s)

Upload the ***install-remapper.sh*** and the ***remap-it.template*** files to the appropriate instance(s). The service is designed to run in an unprivileged account (e.g., the ubuntu user), using *sudo* when necessary.


#### Step 4 - Configure the Installation Script

You will need to edit the installation script to add your ***IAM User credential*** (an access key and secret key). Within AWS this IAM user does not require much by way of permissions.  Here is a suggested simple policy:

https://github.com/parkmycloud/useful_tools/AWS/RecommendedPolicies/parkmycloud-recommended-policy-with-tagging.json


#### Step 5 - Run the Installation Script

When you run ***install-mapper.sh*** it will perform the following actions:

* Create a directory structure for the service in **/opt/ec2**.

* Create the service script called ***remap-ip.sh***, which will be stored there

* Create an *upstart* file called ***remap-ip.conf***, which will be stored in **/etc/init** and be used to start the service when the instance starts or reboots.

* Generate a self-signed X509 certificate which will need to be associated with the IAM user account, either through the console, API or CLI tools 

* Install ***unzip*** using apt-get, if not alrady installed

* Install ***openssl***, if not already installed

* Install ***AWS EC2 API Tools***

* Install ****Java JDK***, if not already installed.


#### Step 6 - Upload Your Self-Signed X509 Certificate to AWS

You will need to upload the self-signed X509 certificate generated by the install process to AWS.

This certificate is needed for signing the API requests to AWS.  This reference describes this in more detail:

http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/ec2-cli-managing-certs.html?icmpid=docs_iam_console.


### You're Done!

Every time the instance starts up it will assign the Elastic IP and hostname you specified in the userData to instance.

*Copyright 2016. ParkMyCloud, Inc. All rights reserved.*
