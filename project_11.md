## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

A bastion host is a server whose purpose is to provide access to a private network from an external network, such as the Internet. 

## Ansible Installation and Configuration on EC2 instance

- Either launch a new instance or rename the previous instance used for Jenkins

- Install Ansible on the instance after updating the server with the following command:

> ` sudo apt update; sudo apt install ansible`

- Verfy that ansible is installed by checking the version: ` ansible --version`

![ansible_version](https://user-images.githubusercontent.com/114786664/203035685-799fb5f0-aca9-4710-855e-52618ad490d6.png)

---

## Configure Jenkins build job to save your repository content every time you change it

- Create a new Freestyle project named ansible in Jenkins and point it to your *ansible-config-mgt* repository.

- Configure Webhook in GitHub and set webhook to trigger ansible build

- Configure a Post-build job to save all (**) files

## Prepare your development environment using Visual Studio Code

- Installation of of te VS Code 

- Configure to ensure its connection to github

- Clone the the newly created Ansible repository to the VSCode

## Ansible Developnment

- Creation oof a new branch in the repository named *proj_11*

- Create a directory and name it *playbooks*  that will be used to store all the playbook files.
- Create a directory named inventory which will be used to keep your hosts organised.

- Within the playbooks folder, create your first playbook, and name it _common.yml_

- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) _dev, staging, uat,_ and _prod_ respectively as shown in the below figure:

![dir_files](https://user-images.githubusercontent.com/114786664/203584878-6fc5aab7-0c1a-4523-87c9-b6bc8979a624.png)

---

## Setting up Ansible Inventory

- Set up the inventory file as shown below

![inventory](https://user-images.githubusercontent.com/114786664/203585765-b39ce3e5-33e5-4487-ba59-c2c5fec441cc.png)

---

### Using SSH Agent to log into the Jenkins_Ansible Server

- We'll need to copy the instance rsa key to the host system using the ssh-add command as shown below:

> `ssh-agent -s`
> `ssh-add <path-to-private-key> `

![ssh_1](https://user-images.githubusercontent.com/114786664/203588266-90961633-89ec-44e7-aaa2-51d931e3de62.png)

## Creating a Playbook

- In _common.yml_ playbook file, you will write configuration tasks to be executed

- The _playbooks/common.yml_ file should be updated with following code:

![playbook_config](https://user-images.githubusercontent.com/114786664/203598269-afc81112-bcf8-4826-9c8b-20d8c1575e90.png)

---

## Updating the GIT with the latest changes

- Next is to update the GIT repo with the changes made to the _common.yml_ file by:

- using git commands to add, commit and push your branch to GitHub.

> `git status`

> ` git add <selected files>`

> `git commit -m "commit message" `

- Create a Pull request

- And subsequently merge the code to the master branch.

- Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to 

`/var/lib/jenkins/jobs/Ansible_Config/builds/4/archive/playbooks/common.yml` directory on Jenkins-Ansible server as shown below:

![playbook_1](https://user-images.githubusercontent.com/114786664/203602729-4d77bf42-e1f5-42e0-8b45-bf391e32a30f.png)

---

## Running the First Ansible Test

- It is time to execute ansible-playbook command and verify if playbook works:

> `cd ansible-config-mgt `

> `ansible-playbook -i inventory/dev.yml playbooks/common.yml`

- The below diagram was display as a summary of the run playbook

![playreslt1](https://user-images.githubusercontent.com/114786664/203035717-d90e41c4-b2b7-4646-8bf4-8bb6dfa89019.png)

---

## Optional Step

- In this optional steps, I installed httpd on the RHEL (web & NFS) servers, while i installed its equivalent on the Ubuntu (LB & DB) servers 

- Both installation is done using the _builtin package_ module instead of the specific package managers (yum, apt..)

- Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to 

`/var/lib/jenkins/jobs/Ansible_Config/builds/5/archive/playbooks/common.yml` directory on Jenkins-Ansible server as shown below:

![playbook_2](https://user-images.githubusercontent.com/114786664/203608229-21f60249-ad3b-4909-9fd6-994b0d4bace7.png)

---

- once the installation was done, the playbook was modified to restart the apache on the LB & DB server as shown the last line of the below diagram

![playbook_n](https://user-images.githubusercontent.com/114786664/203609116-65348911-98c5-4e77-9e9d-92348869d5bb.png)

---

- Testing the display of the installed apache on the web browser gives the default page of Apache as shown in the below diagram:

![2nd_inst](https://user-images.githubusercontent.com/114786664/203035678-0e6ba91e-9faa-43a4-8ad3-8407da58fc41.png)

---

![apache_display](https://user-images.githubusercontent.com/114786664/203035696-04485f7b-41fb-46d9-8190-72373667968f.png)

---

__END__