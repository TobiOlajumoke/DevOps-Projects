# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY
WARNING: This infrastructure set up is NOT covered by AWS free tier. Therefore, ensure to DELETE ALL the resources created immediately after finishing the project. Monthly cost may be shockingly high if resources are not deleted. Also, it is strongly recommended to set up a budget and configure notifications when your spendings reach a predefined limit. Watch this video to learn how to configure AWS budget.

### Let us Get Started
You have been doing great work so far – implementing different Web Solutions and getting hands-on experience with many great DevOps tools. In previous projects you used basic Infrastructure as a Service (IaaS) offerings from AWS such as EC2 (Elastic Compute Cloud) as rented Virtual Machines and EBS (Elastic Block Store), you have also learned how to configure Key pairs and basic Security Groups.
But the power of Clouds is not only in being able to rent Virtual Machines – it is much more than that. From now on, you will start gradually study different Cloud concepts and tools on example of AWS, but do not be worried, your knowledge will not be limited to only AWS specific concepts – overall principles are common across most of the major Cloud Providers (e.g., Microsoft Azure and Google Cloud Platform).
**NOTE**: The next few projects will be implemented manually. Before you begin to automate infrastructure in the cloud, it is very important that you can build the solution manually. Otherwise, programming your automation may become frustrating very quickly.
You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.
Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![alt text](<images/architexture daigram.png>)

## Starting Off Your AWS Cloud Project

There are few requirements that must be met before you begin:
1. Properly configure your AWS account and Organization Unit Watch How To Do This Here
- Create an AWS Master account. (Also known as Root Account)
- Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
- Move the DevOps account into the Dev OU.
- Login to the newly created AWS account using the new email address.


2. Create a free domain name for your fictitious company at Freenom domain registrar here.


3. Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that here














