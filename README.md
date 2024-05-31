# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project we will we need to do the following:

- Refactor the Ansible code
- Create assignments
- Use the imports functionality.

Imports allow to effectively re-use previously created playbooks in a new playbook. This helps to organize tasks and reuse them when needed.

To better understand the Ansible artifact re-use, [click here](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html).


[Code Refactoring](https://en.wikipedia.org/wiki/Code_refactoring) means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity and add proper comments without affecting the logic.

### JENKINS JOB ENHANCEMENT

In our previous jobs, Jenkins was configured to create a directory for every change in the code which consumes space in the Jenkins server. To fix this we will be making changes using copy artifact plugin.

Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.
```
sudo mkdir /home/ubuntu/ansible-config-artifact
```
Change permissions to this directory, so Jenkins could save files there 
```
sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact
```

![Alt text](images/12.1.png)

Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![Alt text](images/12.2.png)

Create a new Freestyle project (you have done it in [Project 9](https://github.com/Olaminiyi/Project-9)) and name it save_artifacts.
In the general configuration: check discard old builds
put maximum number of build to 2

![Alt text](images/12.3.png)
   
Under Build Triggers: check build after other project are built; in project to watch put 'ansible'
Then save

![Alt text](images/12.4.png)

The main idea of save_artifacts project is to save artifacts into **/home/ubuntu/ansible-config-artifact** directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and **/home/ubuntu/ansible-config-artifact** as a target directory.
Go to configuration of the new project 'save_artifacts'
Under build steps
- type ansibe as the project name
- under artifacts to copy type '**'
- under target directory; put - `/home/ubuntu/ansible-config-artifact` 
- Save

![Alt text](images/12.5.png)

Test your set up by making some change in `README.MD` file inside your `ansible-config-mgt repository` (right inside master branch).
If both Jenkins jobs have completed one after another – you shall see your files inside `/home/ubuntu/ansible-config-artifact` directory and it will be updated with every commit to your master branch.
   
![Alt text](images/12.14.png) 

# Step 2 – Refactor Ansible code by importing other playbooks into site.yml
```
git checkout main
```
```   
git branch refactor
```
```
git checkout refactor
```
![Alt text](images/12.6.png)

Within playbooks folder, create a new file and name it `site.yml` – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including `common.yml` that you created previously. Dont worry, you will understand more what this means shortly.

![Alt text](images/12.7.png)

Create a new folder in root of the repository and name it `static-assignments`. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

![Alt text](images/12.8.png)

Move common.yml file into the newly created static-assignments folder.

![Alt text](images/12.9.png)

![Alt text](images/12.10.png)

Inside site.yml file, import common.yml playbook.

![Alt text](images/12.11.png)


### Run ansible-playbook command against the dev environment
Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under `static-assignments` and name it `common-del.yml`. In this playbook, configure deletion of wireshark utility.

![Alt text](images/12.12.png)

Update site.yml with - `import_playbook: ../static-assignments/common-del.yml` instead of common.yml and run it against dev servers:

![Alt text](images/12.13.png)

```   
cd /home/ubuntu/ansible-config-mgt/
```
```
ansible-playbook -i inventory/dev.yml playbooks/site.yml
```
![Alt text](images/12.15.png)

Make sure that wireshark is deleted on all the servers by running wireshark --version

![Alt text](images/12.16.png)

### Step 3 – Configure UAT Webservers with a role ‘Webserver’

Launch 2 fresh EC2 instances using `RHEL 8 image`, we will use them as our uat servers, so give them names accordingly – `Web1-UAT` and `Web2-UAT`.

![Alt text](images/12.17.png)

To create a role, you must create a directory called `roles/`, relative to the playbook file or in `/etc/ansible/` directory.
There are two ways how you can create this folder structure:
Use an Ansible utility called [ansible-galaxy](https://galaxy.ansible.com/ui/) inside ansible-config-mgt/roles directory (you need to create roles directory upfront).
```
mkdir roles
```
```
cd roles
```    
After removing unnecessary directories and files, the roles structure should look like this

└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates

![Alt text](images/12.18.png)

Update your inventory `ansible-config-mgt/inventory/uat.yml` file with IP addresses of `your 2 UAT Web servers`

![Alt text](images/12.19.png)

In `/etc/ansible/ansible.cfg` file uncomment roles_path string and provide a full path to your roles directory `roles_path  = /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.
ssh into ansible jenkins wit (ssh -A)
```
cd ansible-config-artifact
```
```
sudo vi /etc/ansible/ansible.cfg 
```   
Uncomment the roles_path and add the path to the roles to it. i.e `/home/ubuntu/ansible-config-artifact/roles`

![Alt text](images/12.20.png)

It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:
Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
```
git clone https://github.com/Olaminiyi/tooling
```
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started

![Alt text](images/12.21.png)

### Reference ‘Webserver’ role

Within the static-assignments folder, create a new assignment for uat-webservers.yml. This is where you will reference the role.

![Alt text](images/12.22.png)

Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.

![Alt text](images/12.23.png)

### Commit & Test

Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into `/home/ubuntu/ansible-config-mgt/` directory.

Now run the playbook against your uat inventory and see what happens
```
ansible-playbook -i inventory/uat.yml \playbooks/site.yml
```
![Alt text](images/12.24.png)

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:
1. web uat 1

![Alt text](images/12.25.png)
   
2. web uat 2
  
![Alt text](images/12.26.png)

The Ansible architecture now looks like this

![alt text](images/project12_architecture.png)

