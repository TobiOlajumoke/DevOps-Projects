## LOAD BALANCER SOLUTION WITH APACHE
From our project 7
# STEP 1
## Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.

# Prerequisites
- Make sure that you have the following servers installed and configured within Project-7:
- Two RHEL8 Web Servers
- One MySQL DB Server (based on Ubuntu 20.04)
- One RHEL8 NFS server

# CONFIGURE APACHE AS A LOAD BALANCER
- Configure Apache As A Load Balancer
   - Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:


   - Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in the Security 


dorcas ma kpa mi na