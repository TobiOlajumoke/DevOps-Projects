 # Continous Integration Pipeline For Tooling Website


## INSTALL AND CONFIGURE JENKINS SERVER
Step 1 – Install the Jenkins server
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
- Install JDK (since Jenkins is a Java-based application)
`sudo apt update`
`sudo apt install default-jdk-headless`

By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
