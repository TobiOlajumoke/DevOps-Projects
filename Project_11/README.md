## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

You have been implementing some interesting projects up until now, and that is awesome.
In Projects 7 to 10 you had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy your web application.
This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time you will become confident at writing code using declarative language such as YAML.
Let us get started!


Ansible Client as a Jump Server (Bastion Host)

A [Jump Server](https://en.wikipedia.org/wiki/Jump_server) (sometimes also referred as [Bastion Host](https://en.wikipedia.org/wiki/Bastion_host)) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces [attack surface](https://en.wikipedia.org/wiki/Attack_surface).



When you reach Project 15, you will see a Bastion host in proper action. But for now, we will develop Ansible scripts to simulate the use of a Jump box/Bastion host to access our Web Servers.

### Task
- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration


## STEP 1

INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
![Alt text](Images/jenkins-ansible.png)
- In your GitHub account create a new repository and name it ansible-config-mgt.
- Install Ansible
```sh
sudo apt update
```

```sh
sudo apt install ansible
```

Check your Ansible version by running `ansible --version`
![Alt text](Images/jenkins-ansible.png)


- Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.
   - Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
![Alt text](Images/jenkns%20build%201.png)

   - Configure Webhook in GitHub and set webhook to trigger ansible build.
![Alt text](Images/jenkns%20build%201.5.png)

   - Configure a Post-build job to save all (**) files, like you did it in Project 9.
![Alt text](Images/jenkns%20build%202.png)

- Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder.


![Alt text](Images/tesr%201.png)
![Alt text](Images/tesr%202.png)

```sh
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
![Alt text](Images/ls%20command.png)
![Alt text](Images/ls%20jenkins%20artifact.png)
Note: Trigger Jenkins project execution only for /main  branch.
Now your setup will look like this:

![Alt text](Images/looks%20like%20now.png)




 Every time you stop/start your Jenkins-Ansible server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10).
 
 Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.



## Step 2 – Prepare your development environment using Visual Studio Code
First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – [Visual Studio Code (VSC)](https://en.wikipedia.org/wiki/Visual_Studio_Code), you can get it [here](https://code.visualstudio.com/download).

- After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.
- Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance
`git clone <ansible-config-mgt repo link>`


### BEGIN ANSIBLE DEVELOPMENT

In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the
 name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)
- Checkout the newly created feature branch to your local machine and start building your code and directory structure
- Create a directory and name it playbooks – it will be used to store all your playbook files.
- Create a directory and name it inventory – it will be used to keep your hosts organised.
- Within the playbooks folder, create your first playbook, and name it common.yml
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

## Step 4 – Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

- Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:
```sh
eval `ssh-agent -s`
```
```sh
ssh-add <path-to-private-key>
```

- Confirm the key has been added with the command below, you should see the name of your key
```sh
ssh-add -l
```

- Now, ssh into your Jenkins-Ansible server using ssh-agent
```sh
ssh -A ubuntu@public-ip
```



- Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.


Update your inventory/dev.yml file with this snippet of code:

```sh
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

```


## CREATE A COMMON PLAYBOOK
### Step 5 – Create a Common Playbook
It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.
In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.
Update your playbooks/common.yml file with following code:
```sh
---
- name: update web and nfs 
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server and DB server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

```

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

Feel free to update this playbook with following tasks:
Create a directory and a file inside it
Change timezone on all servers
Run some shell script
…
For a better understanding of Ansible playbooks – [watch this video](https://youtu.be/ZAdJ7CdN7DY) from RedHat and read [this article](https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook).


