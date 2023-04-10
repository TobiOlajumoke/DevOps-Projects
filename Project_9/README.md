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
Enable webhooks in your GitHub repository settings

