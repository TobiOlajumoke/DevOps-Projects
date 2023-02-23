# Configure an Apache Load Balancer for Tooling Website solution

![Alt text](Images/3tier%20lb%20diagram.png)

In this project, we will enhance our Tooling Website solution by adding a [Load Balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) to distribute traffic between Web Servers and allow users to access our website using a single URL.

### Task
- Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.
- To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.
### Prerequisites
Make sure that you have the following servers installed and configured within Project-7:
- Two RHEL8 Web Servers
- One MySQL DB Server (based on Ubuntu 20.04)
- One RHEL8 NFS server 


# Configure Apache As A Load Balancer
Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:

- Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in the Security Group.
![Alt text](Images/LB%20port%2080.png)

## Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
### Install apache2
```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev -y
```

### Enable the following modules:
```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```

### Restart apache2 service
`sudo systemctl restart apache2`

Make sure apache2 is up and running

`sudo systemctl status apache2`

# Configure load balancing

