# DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION: INTRODUCTION TO JENKINS

Continuous integration (CI) is a software development strategy that make sure that the speed and quality of code deployment is achieved. 
In this project, I will be making use of Jenkins to ensure that every changes made to the choosing repository will be automatically updated on the tooling website.

> ### Steps
1. Lauching an EC2 instances
2. Install Java.
3. Install Jenkins.
4. Modify Firewall to Allow Jenkins.
5. Set up Jenkins.
6. Configuration of Jenkins to Retrieve Source Codes from GitHub using Webhooks 
7. Configuration of Jenkins to copy files to NFS Server 

## Step 1: Lauching two EC2 instances

* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch a new EC2 instances of t2.micro family with Ubuntu Server 20.04 LTS (HVM); name it Jenkins

---

## Step 2: Installation of Java

- Update the server
- Upgrade and 
- Install java package with the following code:

> ` sudo apt update && sudo apt upgrade -y && sudo apt install default-jdk-headless -y`
---

## Step 3: Add Jenkins Repository

- All three tasks can be done using a single line of command:

``` 
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \ 
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins -y
```

- To ensure Jenkins is up and running:

> ` sudo systemctl status jenkins `

- Exit the status by entering `Ctrl + C `
---

## Step 4: Modify Firewall to Allow Jenkins

- To allow Jenkins to communicate through the default port 8080, the following code is entered:

> ` sudo ufw allow 8080 `

> `sudo ufw status `

- Also edit the SG of the instance to listen at port 8080

![sg](https://user-images.githubusercontent.com/114786664/201080156-b89aa80d-f3a6-43b5-81a7-1fc502fb9698.png)

---

## Step 5:  Jenkins Set up 

- This is done on the Jenkins web page 

- Visit Jenkins web page on the browser: *public_ip_address:8080*

- The below page will be displayed requesting for thhe administrator's password which can be gotten from this path:

> ` sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![1](https://user-images.githubusercontent.com/114786664/201080368-fe82586f-fd38-429c-8b4d-f85ddb1aefe6.png)

- After which suggested plugin should be installed as shown below.

![2](https://user-images.githubusercontent.com/114786664/201082857-7c590c62-4be2-43a6-986f-32568bba8efa.png)

- After plugin installation is completed, a prompt is displayed to create an admin user and password

- This mark the completion of the setup
---

## Step 6: Configuration of Jenkins to Retrieve Source Codes from GitHub using Webhooks

- The purpose of this is to configure a simple Jenkins job

- This job will be triggered by GitHub webhooks

- Build task will be executed to retrieve codes from GitHub and store it locally on Jenkins server.

- The setup goes thus:

> i. Enabling Webhook in the choosen GIT Hub repository (tooling)

> ii. On the Jenkins web console, click "New Item" and "Freestyle project"

> iii. To connect the GitHub repository, provide its URL at the Jenkins web console

> iv. Save the configuration and try to run the build by clicking the "Build Now" button

> vi. The build should be successful with the below display

![last_1](https://user-images.githubusercontent.com/114786664/201299482-b299be5c-7593-436c-a156-d267b55f1e2b.png)

![jenk_2](https://user-images.githubusercontent.com/114786664/201297196-b5813358-53ce-435a-afcf-dcdcad5594c1.png)

- The build only show up manually. To configure it to respond automatically, I follow the following steps:

> i. Configure triggering the job from GitHub webhook at the "Build Triger" section.

> ii. Also, configure "Post-build Actions" to archive all the files

- We can edit the repository file on GIT Hub, which should reflect automacally on the Jenkins console

![last_6](https://user-images.githubusercontent.com/114786664/201303654-9f4ae7e2-b11f-42da-9ada-ee734eb5d7e9.png)

- The artifacts are stored on Jenkins server locally at this path:

> ` ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/ `

![artif_store](https://user-images.githubusercontent.com/114786664/201304900-9b888c2b-c6c4-40ae-baa3-72a3a4e17e77.png)

---

## Step 7: Configuration of Jenkins to copy files to NFS Server 

- Since the artifact is saved locally, I'll need to configure the Jenkins server to copy to the NFS server to **/user/apps** path

- On the Jenkins web, 

> - Install "Publish Over SSH" plugin

> - Configure the job to copy artifacts over to NFS server

> - Provided that all configuration went well, testing should return **"success"**

![Success](https://user-images.githubusercontent.com/114786664/201308101-a7c4ae02-615e-4c5b-828e-69dfb401ac45.png)

> - Save the configuration
> 
- Open the Jenkins job configuration page and add another  "Post-build Action" 

- Click the "Send Build Artifact Over SSH"

- Save 

- Going to git hub to modify the repo. will trigger a new job, and contents of Jenkins server will have copied its contents to the NFS

![last_n0](https://user-images.githubusercontent.com/114786664/201313669-8cae5f14-ae2b-4f3c-82a5-cdd12d4b3531.png)

![last_n](https://user-images.githubusercontent.com/114786664/201313602-66dd355e-ff0f-49ef-a475-6d25d199f98e.png)

- Contents of the Jenkins in the NFS server as shown below.

![nfs_tooling](https://user-images.githubusercontent.com/114786664/201313975-166265ac-94ff-4867-8c92-98a2c2f924fc.png)


---
__END__