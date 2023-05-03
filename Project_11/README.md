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


