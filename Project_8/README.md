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


