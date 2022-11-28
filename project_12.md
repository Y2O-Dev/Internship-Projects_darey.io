## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

Code refactoringg simply means the process of changing the structure of an already existing code without changing the external functionality of the code. It main purpose is to improve the design, functionality etc of the software.
In this project, we'll continue our journey on Ansible to include the use of imports and roles.

At the end of this configuration, the Ansible architecture will looks like this:

![Architec](https://user-images.githubusercontent.com/114786664/204156423-a20b25fe-3551-4b11-b187-0ac7c2070aa4.png)

---

### Step 1: Jenkins job enhancement

Some changes need be make to the Jenkins job because presently every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Apart from that, it consumes space on Jenkins serves with each subsequent change.

To do this, follow the below 7 steps:

1.  SSH into the Jenkins server and create another directory and a name of your choice. In my case, **ansible-config-artifact**

> `sudo mkdir /home/ubuntu/ansible-config-artifact `

2. Change permissions to this directory, so Jenkins could save files in the dir.

> `chmod -R 0777 /home/ubuntu/ansible-config-artifact `

![1](https://user-images.githubusercontent.com/114786664/204143260-62e85d1f-2cde-4f38-9fb4-c3924c487811.png)

---

3. Install *Copy Artifact* from Maneged Plugins on the Jenkins web console

4. Create a new Freestyle project: give it any name but in my case **save_artifacts.**

5. This project will be triggered by completion of your existing ansible project if configured it correctly

6. For **save_artifacts** project to save  artifact into **/home/ubuntu/ansible-config-artifact** directory, *Build* steps will be created, and choose *Copy artifacts from other project*, specify *ansible* as a source project and /**home/ubuntu/ansible-config-artifact** as a target directory.

7. Test the set up by making some change in README.MD file inside your *ansible-config-mgt* repository. Specifically on the master branch 

- If all set up went well, you should see your files inside the 
**/home/ubuntu/ansible-config-artifact** directory

![2_jenkin_artifact](https://user-images.githubusercontent.com/114786664/204143261-38efd42b-eda7-4cfc-9ecc-efdab7d6b779.png)

---
Now the Jenkins pipeline is more clean

### Step 2: Refactor Ansible code by importing other playbooks into site.yml

1. Within **playbooks** directory, create a new file and give name of your choice: **site.yml** – This file will now be considered as an entry point into the entire infrastructure configuration.

2. Create a new dirctory in root of the repository and name it **static-assignments**. The **static-assignments** folder is where all other children playbooks will be stored.

3. Move common.yml file into the newly created static-assignments folder.

4. Inside *site.yml* file, import *common.yml* playbook as shown below: 

![6_import_playbook](https://user-images.githubusercontent.com/114786664/204143292-7110a0ff-5e66-40b7-908d-81678151825e.png)

---

The folder structure should look like this:

![3_tree](https://user-images.githubusercontent.com/114786664/204143263-bae953f1-8d4b-4ad2-98a1-de2f5dacab1d.png)

---

5. Run ansible-playbook command against the dev environment. 

- This will not display any changes on the servers since all taks in the playbook was already installed

- To see the impart of import, we'll write another playbook in *static-assignments* and name it *common-del.yml*  to delete wireshark utility from the servers

![6c_common_del_yml](https://user-images.githubusercontent.com/114786664/204151182-e7cba2af-c7a5-4400-b706-2b81ef63155e.png)

---

update *site.yml* with: 

Import_playbook: ../static-assignments/*common-del.yml* instead of common.yml and run it against dev servers:

``` 
cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```

![5_import_del_top](https://user-images.githubusercontent.com/114786664/204143288-1f638590-467d-4369-9453-b9ff74e29d21.png)

---

![5a_import_del_tail](https://user-images.githubusercontent.com/114786664/204143287-e64de33f-bb7d-4ad5-96d4-dca104e7c9f6.png)

---

- To ensure that wireshark is deleted on all the servers by running:

> `wireshark --version `

![6a_lb_delwireshrk](https://user-images.githubusercontent.com/114786664/204143289-340ce93a-b946-4bf1-9caf-1c757bb94a90.png)

---

![6b_db_delwireshrk](https://user-images.githubusercontent.com/114786664/204143290-72e47fff-7655-405d-bdaa-f7d7a4180e68.png)

---

### Step 3: Configure UAT Webservers with a role ‘Webserver’

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – *Web1-UAT* and *Web2-UAT*.

2. To create a role, you must create a directory called **roles/**

- Use an Ansible utility called ansible-galaxy inside **ansible-config-artifact/roles** directory to create other Roles dependent directories.

```
mkdir roles
cd roles
ansible-galaxy init webserver
```
- Although the directory/files structure can be created manually

- The entire folder structure should look like below, 

![7_galax_init_tree](https://user-images.githubusercontent.com/114786664/204143295-87316f8b-17dd-4544-bcd8-4d2f4aab247f.png)

---

- we can remove the tests, files, and vars directory, after which the tree  structure should look like this:

![7b_post_del_tree](https://user-images.githubusercontent.com/114786664/204143294-6b0c8e7d-c621-4d84-b570-7e0665de5d59.png)

---

3. Update your inventory *ansible-config-artifact/inventory/uat.yml* file with IP addresses of your 2 UAT Web servers

![7c_uat_inventory](https://user-images.githubusercontent.com/114786664/204153990-454209b6-4de3-412b-a899-93f947333fc0.png)

---

4. In */etc/ansible/ansible.cfg* file uncomment roles_path string and provide a full path to your roles directory 

> `roles_path    = /home/ubuntu/ansible-config-artifact/roles`

5. It is time to start adding some logic to the webserver role. 

- Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

- Install and configure Apache (httpd service)

- Clone Tooling website from GitHub

- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.

- Make sure httpd service is started

![8_playbook](https://user-images.githubusercontent.com/114786664/204154656-f242c618-a3e0-4f21-903c-6b16cce185aa.png)

---

### Step 4: Reference ‘Webserver’ role

- Within the **static-assignments** folder, create a new assignment for uat-webservers *uat-webservers.yml*. This is where you will reference the role.

![8_uat_yml](https://user-images.githubusercontent.com/114786664/204155039-588d4f2c-d6f0-4480-aa1a-952df009720d.png)

---

- the entry point to our ansible configuration is the *site.yml* file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.

![8_site_yml](https://user-images.githubusercontent.com/114786664/204155204-696a4a97-cc4e-4719-9059-db75039ee481.png)

---

### Step 5: Commit & Test

- Commit your changes, create a Pull Request and merge them to master branch

- Run the playbook against the uat inventory

> ` ansible-playbook -i ansible-config-artifact/inventory/uat.yml ansible-config-artifact/playbooks/site.yml `

![8a_playbook_out_top](https://user-images.githubusercontent.com/114786664/204143298-b69d4ec2-e8e5-4c68-8dde-aead75e1ffe9.png)

![8a_playbook_out_tail](https://user-images.githubusercontent.com/114786664/204143296-19318953-95f0-4164-9da9-b52c275b0f3b.png)

---

- To confirm UAT Webserver configuration, we'll try to reach both from our browser:

> `http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php `

![8b_web1_display](https://user-images.githubusercontent.com/114786664/204143329-1ac1ad9e-a029-4de0-bb8a-3530aabfa16c.png)

---

![8b_web_display](https://user-images.githubusercontent.com/114786664/204143331-9105701d-a6a9-4684-9a6c-bf584c69b7e5.png)

---

### END