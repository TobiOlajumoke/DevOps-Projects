## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

By now you have learned what Load Balancing is used for and have configured an LB solution using Apache, but a DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. That is why, in this project we will configure an [Nginx](https://www.nginx.com/) Load Balancer solution.
It is also extremely important to ensure that connections to your Web solutions are secure and information is [encrypted in transit](https://security.berkeley.edu/data-encryption-transit-guideline) – we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.
When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.
This threat is real – users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.
SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.
SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.
There are different types of SSL/TLS certificates – you can learn more about them here. You can also watch a tutorial on how SSL works here or an additional resource here

In this project you will register your website with LetsEnrcypt Certificate Authority, to automate certificate issuance you will use a shell client recommended by LetsEncrypt – cetrbot.


This project consists of two parts:
- Configure Nginx as a Load Balancers

- Register a new domain name and configure secured connection using SSL/TLS certificates

Your target architecture will look like this:
![Alt text](Images/P10%201.png)

## CONFIGURE NGINX AS A LOAD BALANCER
You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.

- To uninstall and use apache on the instance we just finished using recently 
run this :
```sh
sudo systemctl stop apache2
sudo apt-get remove apache2 -y
```
```sh
sudo apt-get purge apache2 -y
```
```sh
sudo apt-get autoremove -y
```
```sh
sudo apt-get clean 

```
#                                      OR
- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)\

![Alt text](Images/P10%202.png)

- Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

from  the terminal:
```sh
sudo vi /etc/hosts

```
![Alt text](Images/etc%20hosts.png)

- Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

```sh
sudo apt update -y
sudo apt install nginx -y
```


- Configure Nginx LB using Web Servers’ names defined in /etc/hosts
Hint: Read this [blog[(https://linuxize.com/post/how-to-edit-your-hosts-file/)] to read about /etc/host

Open the default nginx configuration file

```sh
sudo vi /etc/nginx/sites-available/load_balancer.conf

```
#insert following configuration

```sh
upstream web {
    server Web1 weight=5;
    server Web2 weight=5;
}


server {
            listen 80;
            server_name www.domain.com;
            location / {
                
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://web;
            }
}

 
```
- edit server name and domain name
eg tobi.com www.tobi.com  


- run the following command next
we'll delete the default configuration file 
```sh
sudo rm -f /etc/nginx/sites-enabled/default
```
- will parse and validate the configuration files:
```sh
sudo nginx -t
```
![Alt text](Images/sudo%20nginx%20-t.png)

- cd into this directory
```sh
cd /etc/nginx/sites-enabled/
```
- run the ls command it's empty 
- we are gonna create a symbolic link in the current directory to the configuration file `load_balancer.conf` located in the `sites-available` directory :
```sh
sudo ln -s ../sites-available/load_balancer.conf .
```
![Alt text](Images/symlink.png)

- Restart Nginx and make sure the service is up and running
```sh
sudo systemctl restart nginx
sudo systemctl status nginx
```



## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
Let us make necessary configurations to make connections to our Tooling Web Solution secured!

- In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any [Domain name registrar](https://en.wikipedia.org/wiki/Domain_name_registrar) – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.
Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

I'm making use of my Godaddy domain 
 - Go to route 53 on aws portal
 ![Alt text](Images/domain%20name%20setup%201.png)
 - now click create hosted zone
 ![Alt text](Images/domain%20name%20setup%202.png)
 - Add your domain name and create
 ![Alt text](Images/Domain%20name%20setup%203.5.png)
 - copy all 4 name servers under records
 ![Alt text](Images/Domain%20name%20setup%204.png)
 - paste them in your domain name setting, Under name server
 ![Alt text](Images/Domain%20name%20setup%203.png)
 - Now we create a record 
 ![Alt text](Images/Domain%20name%20setup%205.png)




- Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server [on this page](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html).







  

cd /etc/nginx/sites-available/
.load_balancer.conf.swp