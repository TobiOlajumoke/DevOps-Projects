 # Continous Integration Pipeline For Tooling Website
![Alt text](Images/jenkins.png)

## INSTALL AND CONFIGURE JENKINS SERVER
Step 1 – Install the Jenkins server

- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

- Install JDK (since Jenkins is a Java-based application)

```sh
sudo apt update
sudo apt install default-jdk-headless -y
```

- Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

```
`sudo apt update`
`sudo apt-get install jenkins`


- Make sure Jenkins is up and running
```sh
sudo systemctl start jenkins
sudo systemctl status jenkins
```
- By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

![Alt text](Images/jenkins%20SG.png)

- Perform initial Jenkins setup. From your browser access:
```
 http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
 ```

![Alt text](Images/jenkins%20setup%201.png)


- You will be prompted to provide a default admin password
![Alt text](Images/jenkins%20setup%202.png)
Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Then you will be asked which plugings to install – choose suggested plugins.

![Alt text](Images/jenkins%20setup%203.png)


Once plugin installation is done – create an admin user and you will get your Jenkins server address.
The installation is completed!
![Alt text](Images/jenkins%20setup%204.png)

## Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

- Enable webhooks in your GitHub repository settings

![Alt text](Images/jenkins%20setup%205.png)

- Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![Alt text](Images/jenkins%20setup%206.png)
![Alt text](Images/jenkins%20setup%207.png)

- To connect your GitHub repository, you will need to provide its URL, you can copy it from the repository itself
![Alt text](Images/copy%20gthub%20url%20to%20jenkins.png)


- In the configuration of your Jenkins freestyle project choose Git repository, and provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
![Alt text](Images/configure%20freestyle%20project.png)

Save the configuration and let us try to run the build. For now, we can only do it manually.
Click the "Build Now" button, if you have configured everything correctly, the build will be successful and you will see it under #1

![Alt text](Images/jenkins%20setup%2011.png)
You can open the build and check in "Console Output" if it has run successfully.
If so – congratulations! You have just made your very first Jenkins build!
But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.


## Step 3 - Click "Configure" your job/project and add these two configurations
Configure triggering the job from the GitHub webhook:
![Alt text](Images/jenkins%20setup%2012.png)

Configure "Post-build Actions" to archive all the files – files resulting from a build are called "artifacts".
![Alt text](Images/jenkins%20setup%2013.png)

![Alt text](Images/jenkins%20setup%2014.png)
![Alt text](Images/jenkins%20setup%2015.png)

- Now, go ahead and make some changes in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on the Jenkins server.
![Alt text](Images/jenkins%20setup%2016.png)
![Alt text](Images/jenkins%20setup%2017.png)

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and file transfer is initiated by GitHub). There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.
By default, the artifacts are stored on the Jenkins server locally

```
ls /var/lib/jenkins/jobs/<name of project>/builds/<build_number>/archive/
```
eg mine is:

```
ls /var/lib/jenkins/jobs/ToolingJob/builds/2/archive

```
![Alt text](Images/jenkins%20setup%2018.png)

## Step 4  CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH


- Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.


Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".
Install the "Publish Over SSH" plugin.
On the main dashboard select "Manage Jenkins" and choose the "Manage Plugins" menu item.
On the "Available" tab search for the "Publish Over SSH" plugin and install it
![Alt text](Images/jenkins%20setup%2019.png)

![Alt text](Images/jenkins%20setup%2020.png)
![Alt text](Images/jenkins%20setup%2021.png)

- Configure the job/project to copy artifacts over to the NFS server.
On the main dashboard select "Manage Jenkins" and choose the "Configure System" menu item.
![Alt text](Images/jenkins%20setup%2022.png)
Scroll down to Publish over the SSH plugin configuration section and configure it to be able to connect to your NFS server:
![Alt text](Images/jenkins%20setup%2023.png)

Provide a private key (the content of .pem file that you use to connect to the NFS server via SSH/Putty)
Arbitrary name
Hostname – can be private IP address of your NFS server
Username – ec2-user (since the NFS server is based on EC2 with RHEL 8)
Remote directory – /mnt/apps since our Web Servers use it as a mounting point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![Alt text](Images/jenkins%20setup%2024.png)

![Alt text](Images/jenkins%20setup%2025.png)

Save the configuration

- open your Jenkins job/project configuration page and add another one "Post-build Action"
![Alt text](Images/jenkins%20setup%2026.png)
![Alt text](Images/jenkins%20setup%2027.png)