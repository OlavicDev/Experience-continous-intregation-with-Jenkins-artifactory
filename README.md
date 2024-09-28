# Experience-continous-intregation-with-Jenkins-artifactory
### Experience Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

This project provides hands-on experience with CI/CD from the application perspective, specifically focusing on PHP. As you progress, you’ll gain expertise by seeing CI/CD in action across different programming languages, starting with PHP and later expanding to Java, Node.js, .NET, and Python. Additionally, future projects will cover platform-related CI/CD concepts, especially when you work with Terraform, Docker, and Kubernetes.

---

### 13 DevOps Success Metrics

1. **Deployment Frequency**: The aim is to have frequent, smaller deployments for easier testing and quicker releases. Track both production and non-production deployments separately.
   
2. **Lead Time**: Measure the time from starting a work item until its deployment. Shorter lead times indicate a faster code release cycle.

3. **Customer Tickets**: Track user-reported bugs or issues as indicators of software quality and performance problems.

4. **Percentage of Passed Automated Tests**: A key DevOps metric that reflects the stability of code changes. High rates of passed tests ensure smoother development.

5. **Defect Escape Rate**: Track how often bugs escape from QA to production. Minimizing this rate ensures code quality before release.

6. **Availability**: Measure system uptime and ensure minimal downtime, including tracking planned maintenance.

7. **Service Level Agreements (SLA)**: Monitor compliance with SLAs or non-functional requirements to meet user expectations.

8. **Failed Deployments**: Track how often deployments cause issues or downtime, aiming to minimize them and plan for easy rollbacks if necessary.

9. **Error Rates**: Errors serve as a signal for both application quality and performance. Proper error handling and monitoring can provide insights into issues post-deployment.

10. **Application Usage & Traffic**: Monitor transaction or user activity post-deployment to identify abnormal spikes or traffic drops that could indicate issues.

11. **Application Performance**: Use tools like DataDog, New Relic, or AppDynamics to monitor performance changes post-deployment. Look for variations in SQL queries, web service calls, etc.

12. **Mean Time to Detection (MTTD)**: Track how quickly issues are detected to minimize the time users experience problems.

13. **Mean Time to Recovery (MTTR)**: Measure how long it takes to resolve failures. A shorter MTTR ensures quick recovery from system issues.

---

### Simulating a CI/CD Pipeline for a PHP Application

As part of your ongoing infrastructure development with Ansible, you will create a CI/CD pipeline that supports continuous integration and delivery. The pipeline will work for PHP applications like Tooling and TODO web apps, which are interpreted languages deployed directly on a server without compilation. However, to better manage releases and versions, Ansible’s `uri` module will be used for deployments instead of directly downloading from Git.

![image](https://github.com/user-attachments/assets/16354e34-301b-4784-81ef-c4bc9f8397b9)

---

### Set Up Environments

We will focus on the following environments:

1. **CI**
2. **Dev**
3. **Pentest**

![image](https://github.com/user-attachments/assets/87fab601-7b56-46ff-90cd-1bac15515136)

<img width="406" alt="flow-chart" src="https://github.com/user-attachments/assets/c4845e4b-a4a1-4f09-ad6d-9a7fb417ec24">

Each will be served by Nginx acting as a reverse proxy for the sites and tools. The inventory setup for Ansible will look as follows:

---

### Ansible Inventory Structure

```
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

#### CI Inventory File

```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_Repository-Private-IP-Address>
```

#### Dev Inventory File

```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

#### Pentest Inventory File

```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-Tooling-Private-IP-Address>
```

### Observations:

- **Pentest Setup**: A new group, `pentest:children`, is introduced to allow flexible execution of Ansible tasks on both `pentest-todo` and `pentest-tooling`. 
- **Group Vars**: The `db` group uses a different Linux distro, necessitating variations in connection settings. For common variables between `pentest-todo` and `pentest-tooling`, you can declare them in `group_vars` to avoid repetition.
- **Group Vars Implementation**: Ansible’s `group_vars` allows shared variables across server groups, providing flexibility in server management.

---

### Common Best Practices of CI/CD

1. **Maintain a Code Repository**: Ensure code is versioned and available to all stakeholders.
   
2. **Automate the Build Process**: Automate builds to avoid manual errors and delays.

3. **Make Builds Self-Tested**: Incorporate testing into the build process.

4. **Commit to the Baseline Daily**: Frequent commits ensure regular updates and feedback on the codebase.

5. **Build Every Commit**: Each commit should trigger a build process.

6. **Test in a Clone of the Production Environment**: Mimicking the production environment ensures testing accuracy.

7. **Automate Deployment**: Aim for automated deployments to ensure speed and reliability.

---

### Project Description

You will be setting up a CI/CD pipeline for a PHP-based application. This project is split into two main repositories:

1. **Ansible-Config-Mgt REPO**: Contains a Jenkinsfile responsible for setting up infrastructure via Ansible roles. This repo focuses on infrastructure.
   
2. **PHP-Todo REPO**: Contains a Jenkinsfile that focuses on application-specific tasks like building, linting, and static code analysis.

---

### Pre-requisites

For this project, you will use AWS VMs and set up the following servers:

1. **Nginx Server**: Acts as a reverse proxy server.
2. **Jenkins Server**: Implements CI/CD workflows.
3. **SonarQube Server**: Analyzes code quality.
4. **Artifactory Server**: Stores build artifacts.
5. **Database Server**: Hosts the TODO application database.
6. **TODO Web Server**: Hosts the TODO web application.

By following these steps, you’ll simulate a typical CI/CD pipeline while practicing best practices for DevOps success.

You're right; the project details provided earlier contain a lot of specific steps and configurations that I summarized to keep things concise. Let’s expand on the integration project with a more detailed, step-by-step guide that thoroughly covers each part while maintaining clarity. Here’s a comprehensive breakdown that aligns with the complexity of the tasks you shared:

---

## Complete Step-by-Step Guide to Integrate SonarQube, Jenkins, and PostgreSQL
### 1. Install Jenkins
Let's lunch a AWS ec2 with an Ubuntu OS instance and configure the jenkins server on it.


**Install jenkins and it's dependencies using the terminal.
```
sudo apt-get update  # Update the instance
```

# Download jenkins key
```
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

# Add jenkins repository
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Add jenkins key
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
```
# Install Java
```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt install openjdk-11-jdk
```

# Install Jenkins
```
sudo apt-get update
sudo apt-get install jenkins -y
```

# Enable and start Jenkins
```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Open TCP port 8080
![image](https://github.com/user-attachments/assets/9748bc0b-4ef2-4363-8b47-4b33a63e2c5b)


### 2. Installing Blue-Ocean Plugin
Install Blue Ocean plugin a Sophisticated visualizations of CD pipelines for fast and intuitive comprehension of software pipeline status.

Follow the navigation below :
  1. Go to manage jenkins > manage plugins > available
  2. Search for BLUE OCEAN PLUGIN and install


### Configure blue ocean pipeline with git repo
Follow the steps below:
1. Click "Open blue oceans" plugin and create a new pipeline
   Select github
   Connect github with jenkins using your github personal access token
![image](https://github.com/user-attachments/assets/0e5204c3-5713-41ea-9c2e-5160ab65e3eb)

   Select the repository
   Create the pipeline Here is our newly created pipeline. It takes the name of your GitHub repository.

![image](https://github.com/user-attachments/assets/1c8add02-627c-4126-a55f-0d5de6d0d944)

At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.

### Let us create our Jenkinsfile
Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.

   Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

```
pipeline {
    agent any


  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```
Now go back into the Ansible pipeline in Jenkins, and select configure, Scroll down to Build Configuration section and specify the location of the Jenkinsfile at `deploy/Jenkinsfile`

Back to the pipeline again, this time click `Build now`

This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.
   
  1. Click on Open Blue Ocean
  2. Select your project
  3. Click on the play button against the branch
![image](https://github.com/user-attachments/assets/f348e963-adc3-4dc6-9a00-2a3af3ae87c1)

**Let us see this in action.
Create a new git branch and name it `feature/jenkinspipeline-stages`

Currently we only have the `Build stage`. Let us add another stage called `Test`. Paste the code snippet below and push the new changes to GitHub.
```
   pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```
Push the new changes to GitHub

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository i. Click on the "Administration" button ii. Navigate to the Ansible project and click on "Scan repository now"

![image](https://github.com/user-attachments/assets/a7ff20ac-a1d3-43dc-ba9c-c85e75f0fb6d)


 Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

![image](https://github.com/user-attachments/assets/1458f76d-b814-42ac-9d52-73605459a180)


 In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

![image](https://github.com/user-attachments/assets/4b7bbbf5-8582-4f1e-9cdc-0c31b784736c)

### Running Ansible Playbook from Jenkins
Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

Installing Ansible on Jenkins
```
sudo apt update

sudo apt install ansible

ansible --version
```

Installing Ansible plugin in Jenkins UI
![image](https://github.com/user-attachments/assets/a806bd75-e3df-4f9d-b1ac-c12fd78f2645)

Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)
```
pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/deploy/ansible.cfg"
    }

    stages {
        stage('Initial cleanup') {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/francdomain/ansible-config-mgt.git'
            }
        }

        stage('Prepare Ansible For Execution') {
            steps {
                sh 'echo ${WORKSPACE}'
                sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'
            }
        }

        stage('Test SSH Connections') {
            steps {
                script {
                    def allHosts = [
                        'ubuntu@172.31.89.253',
                        'ubuntu@172.31.91.227',
                        'ec2-user@172.31.81.236',
                        'ec2-user@172.31.85.118'
                    ]

                    sshagent(['private-key']) {
                        allHosts.each { host ->
                            sh "ssh -o StrictHostKeyChecking=no ${host} exit"
                        }
                    }
                }
            }
        }

        stage('Run Ansible playbook') {
            steps {
                sshagent(['private-key']) {
                    ansiblePlaybook(
                        become: true,
                        credentialsId: 'private-key',
                        disableHostKeyChecking: true,
                        installation: 'ansible',
                        inventory: '${WORKSPACE}/inventory/dev.yml',
                        playbook: '${WORKSPACE}/playbooks/site.yml'
                    )
                }
            }
        }

        stage('Clean Workspace after build') {
            steps {
                cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
            }
        }
    }
}
```

**Note: Ensure that Ansible runs against the Dev environment successfully.
Possible errors to watch out for:

   Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of Master due to Black Lives Matter. You can read more here)
   
   Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set. https://wiki.jenkins.io/display/JENKINS/Building+a+software+project

Possible issues to watch out for when you implement this

   Remember that ansible.cfg must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor sed to update the section roles_path each time there is an execution. You may not have this issue if you run only from the main branch.
   
   If you push new changes to Git so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the Jenkinsfile with a clean up step to always delete the previous workspace before running a new one. Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.
   
   Another possible reason for Jenkins failure sometimes, is because you have indicated in the Jenkinsfile to check out the main git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.

If everything goes well for you, it means, the Dev environment has an up-to-date configuration.

## Install Ansible
confirm your ansible version with
```
ansible --version
```
### Configure ansible on Jenkins
Click on Dashboard > Manage Jenkins > Global Tool Configuration > Add Ansible. Add a name and the path ansible is installed on the jenkins server.

![image](https://github.com/user-attachments/assets/5283cc85-7439-41a5-8645-1f7a06add0d1)


To ensure jenkins properly connects to all servers, install another plugin called ssh agent just like we did in a previous project 
![image](https://github.com/user-attachments/assets/1ec849a9-2aa7-4396-8c70-e7c4c87c244e)

**Then go to manage jenkins > credentials > global > add credentials
Then follow the steps below:

  1. Kind: SSH Username with private key
  2. Scope: Global (Jenkins, nodes, items, all child items, etc)
  3. ID: private-key (or any ID you prefer)
      Username: Leave it blank or set a default value (e.g., defaultuser) # This is because we are using servers of different username i.e ubbuntu and ec2-user. This value won’t be used because the actual usernames will be specified in the Ansible inventory file.
  5. Private Key: Enter the private key directly

#### Update inventory/dev.yml by specifying the private IP address of the servers
![image](https://github.com/user-attachments/assets/e0244dd1-238a-42cf-9074-24b2ed886984)

#### Update the playbook as well
![image](https://github.com/user-attachments/assets/8d841492-fcd7-4ee6-ad53-dd45ba264d56)

Run Ansible against the Dev environment
![image](https://github.com/user-attachments/assets/a7ba5f0e-8e65-4ffa-ae6f-763b36a5ce2c)
   

### Parameterizing Jenkinsfile For Ansible Deployment
To deploy to other environments, we will need to use parameters.

Update `sit` inventory with new servers (inventory/sit.yml)
```
[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>
```

Update Jenkinsfile to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution.
```
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
...
```

##### In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}


   Notice the Build Now has changed to Build with Parameters and this enables us to run differenet environment easily. The default value loads up, but we can now specify which environment we want to deploy the configuration to. Simply type sit and hit Run


View it from Blue Ocean 

![image](https://github.com/user-attachments/assets/2e86a30b-d488-4bfb-9976-457382f22a12)

Add another parameter. This time, introduce tagging in Ansible. You can limit the Ansible execution to a specific role or playbook desired. Therefore, add an Ansible tag to run against webserver only. Test this locally first to get the experience. Once you understand this, update Jenkinsfile and run it from Jenkins.
`Update playbook/site.yml with tags`
![image](https://github.com/user-attachments/assets/f8c02c7a-f866-48d6-a896-1adebb18899c)


   Add another parameter to the jenkinsfile. Name the parameter ansible_tags and the default value webserver
```
string(name: 'ansible_tags', defaultValue: 'webserver', description: 'Ansible tags to run specific roles or tasks')
```

   Update the Ansible execution section to prompt for tag

Click on the play button and update the inventory field to sit and the ansible_tags to webserver
![image](https://github.com/user-attachments/assets/cfbc9bd9-cb3e-4cee-b968-34f639e91477)

Click on Run to run the build
 
##### To avoid hardcoding values in the Jenkinsfile, we can parameterize several more elements. Here are some suggestions:
   SCM (Source Control Management) URL and Branch: Parameterize the repository URL and the branch to allow flexibility in changing repositories and branches without modifying the Jenkinsfile.
   
   SSH Hosts: Parameterize the list of SSH hosts. This will enable you to specify different hosts for different environments or scenarios.
   
   Ansible Playbook Path: Parameterize the playbook path to allow different playbooks to be specified.
   
   Credentials ID: Parameterize the credentials ID used for SSH and Ansible to support different credentials for different environments.
   
   Role Path: The roles path in the Ansible configuration can be parameterized as well.
```
pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/deploy/ansible.cfg"
    }

    parameters {
        string(name: 'inventory', defaultValue: 'dev', description: 'This is the inventory file for the environment to deploy configuration')
        string(name: 'branch', defaultValue: 'main', description: 'Branch to checkout')
        string(name: 'repo_url', defaultValue: 'https://github.com/francdomain/ansible-config-mgt.git', description: 'Repository URL')
        text(name: 'ssh_hosts', defaultValue: 'ubuntu@172.31.89.253\nubuntu@172.31.91.227\nec2-user@172.31.81.236\nec2-user@172.31.85.118', description: 'List of SSH hosts, one per line')
        string(name: 'playbook_path', defaultValue: '${WORKSPACE}/playbooks/site.yml', description: 'Path to the Ansible playbook')
        string(name: 'credentials_id', defaultValue: 'private-key', description: 'Credentials ID for SSH and Ansible')
    }

    stages {
        stage('Initial cleanup') {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: "${params.branch}", url: "${params.repo_url}"
            }
        }

        stage('Prepare Ansible For Execution') {
            steps {
                sh 'echo ${WORKSPACE}'
                sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'
            }
        }

        stage('Test SSH Connections') {
            steps {
                script {
                    def allHosts = params.ssh_hosts.split('\n')

                    sshagent([params.credentials_id]) {
                        allHosts.each { host ->
                            sh "ssh -o StrictHostKeyChecking=no ${host} exit"
                        }
                    }
                }
            }
        }

        stage('Run Ansible playbook') {
            steps {
                sshagent([params.credentials_id]) {
                    ansiblePlaybook(
                        become: true,
                        credentialsId: params.credentials_id,
                        disableHostKeyChecking: true,
                        installation: 'ansible',
                        inventory: "${WORKSPACE}/inventory/${params.inventory}",
                        playbook: params.playbook_path
                    )
                }
            }
        }

        stage('Clean Workspace after build') {
            steps {
                cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
            }
        }
    }
}
```

### CI/CD Pipline for TODO Application
We already have tooling website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

#### Todo Project files path
```
/path/to/your/laravel/project
├── app
│   ├── Console
│   ├── Exceptions
│   ├── Http
│   │   ├── Controllers
│   │   ├── Middleware
│   ├── Models
│   ├── Providers
├── bootstrap
│   ├── cache
├── config
│   ├── app.php
│   ├── database.php
│   └── ...
├── database
│   ├── factories
│   ├── migrations
│   ├── seeders
├── public
│   ├── index.php
│   ├── css
│   ├── js
│   ├── ...
├── resources
│   ├── js
│   ├── lang
│   ├── views
│   └── ...
├── routes
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   ├── web.php
├── storage
│   ├── app
│   ├── framework
│   ├── logs
├── tests
│   ├── Feature
│   ├── Unit
├── vendor
├── .env
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── phpunit.xml
└── webpack.mix.js
```

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). Configure Artifactory on Ubuntu 20.04

### Create an Ansible role for Artifactory
Prerequests
   Ensure port 8082 is opened in artifactory server
Install Artifactory role using Ansible galaxy collection
```
ansible-galaxy collection install jfrog.platform
```

Update Artifactory role in `roles/artifactory/tasks/main.yml` to install jfrog Artifactory

`Update playbook/site.yml`

`Update inventory/ci.yml`


Run the playbook against ci.yml to install `jfrog artifactory`
 

Access the artifactory GUI on a browser with http://<server public IP address>:8082. Use the default authentication credentials: admin and password to login.

![image](https://github.com/user-attachments/assets/2f605f3c-4a8f-41ec-9cbc-2b1ecd2551d0)

Create a local repository `Todo-artifact-local`

![image](https://github.com/user-attachments/assets/d9241f96-b6cd-4e83-b20f-c810b07ff8f5)


### Phase 1 – Prepare Jenkins
1. Fork the repository below into your GitHub account
```
https://github.com/StegTechHub/php-todo.git
```

2. On your Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)
```
sudo apt update

# Install dependancies
sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}

# Download the installer to the current directory
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Install Composer Globally
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# Remove the Installer
php -r "unlink('composer-setup.php');"

# Verify Installation version
php -v
composer -v
```

The php version installed by the composer is 8.3.6 while the php version of the todo application is 7.4. Ensure to remove the current version and install php 7.4 and its dependencies to avoid error.

#### 3. Install Jenkins plugins
   Plot plugin
![image](https://github.com/user-attachments/assets/0274f67b-8022-4f5e-a27f-90b43f1365e6)

   Artifactory plugin
![image](https://github.com/user-attachments/assets/87d6c0d6-0241-4e7c-9a1d-64f920b852bb)

   We will use plot plugin to display tests reports, and code coverage information.
   The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.
   
#### 4. In Jenkins UI configure Artifactory
Configure the server ID, URL and Credentials, run Test Connection.

![image](https://github.com/user-attachments/assets/5fad2c0f-d0e1-4e4e-93dc-2e1389921e40)

### Phase 2 - Integrate Artifactory repository with Jenkins
   Create a dummy Jenkinsfile in the repository
   Using Blue Ocean, create a multibranch Jenkins pipeline
![image](https://github.com/user-attachments/assets/8155f525-ffeb-47eb-8112-ac92ddcc6c21)


### 3. On the database server, create database and user
In jenkins server Install my sql client
```
sudo apt install mysql-client -y
```
Create database and user on database server
```
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```
Update mysql `role in roles/mysql/vars/main.yml`
![image](https://github.com/user-attachments/assets/1b5eaeb6-cf91-4593-9066-892d64ff1cc5)

Run ansible with jenkins to create the database
![image](https://github.com/user-attachments/assets/aff1b91f-3794-4fad-8363-03ce5df30330)
 
Verify that database has benn created
![image](https://github.com/user-attachments/assets/e533cba6-d689-44d4-a8e9-f06662ccc9b0)


### 4. Update the database connectivity requirements in the file .env.sample
```
APP_ENV=local
APP_DEBUG=true
LOG_LEVEL=debug
APP_KEY=SomeRandomString
APP_URL=http://localhost

DB_HOST=172.31.91.227
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=sePret^i
DB_CONNECTION=mysql
BD_PORT=3306

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

### 5. Update Jenkinsfile with proper pipeline configuration
```
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/StegTechHub/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```

Notice the `Prepare Dependencies` section

   The required file by PHP is .env so we are renaming .env.sample to .env
   
   Composer is used by PHP to install all the dependent libraries used by the application
   
   php artisan uses the .env file to setup the required database objects - (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)
   
   Update the Jenkinsfile to include Unit tests step
   
      Ensure that all neccesary php extensions are already installed.
      Run the pipeline build , you will notice that the database has been populated with tables using a method in laravel known as migration and seeding.
      
      
### Install phpunit
```
wget -0 phpunit.phar https://phar.phpunit.de/phpunit-9.5.10.phar
```
Install phploc
```
phpunit --version
```
**Update Jenkinsfile to include Unit tests
```
stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      }
```
**Run the pipieline
![image](https://github.com/user-attachments/assets/410d4f22-2400-4266-a4ae-053172b36b94)

After successful run, login to the database.
```
sudo mysql
```
**Run show tables and you will see the tables being created.
```
show databases;
```

Set the bind address 
   configure the bind address to `0.0.0.0`
   
### Phase 3 - Code Quality Analysis
This is one of the areas where developers, architects and many stakeholders are mostly interested in as far as product development is concerned. As a DevOps engineer, you also have a role to play. Especially when it comes to setting up the tools.

For PHP the most commonly tool used for code quality analysis is phploc. Read the article here for more

The data produced by phploc can be ploted onto graphs in Jenkins.

   Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.
```
stage('Code Analysis') {
      steps {
            sh 'phploc app/ --log-csv build/logs/phploc.csv'

      }
    }
```

Plot the data using plot Jenkins plugin. This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots' data series latest values are pulled from the CSV file generated by phploc.

**Run the pipeline and View the Plot chart in Jenkins
![image](https://github.com/user-attachments/assets/90b5ffdc-0e99-4d85-8f49-b21b6db42448)



**3. Bundle the application code for into an artifact (archived package) upload to Artifactory
```
stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
```
**4. Publish the resulted artifact into Artifactory
```
stage ('Upload Artifact to Artifactory') {
          steps {
            script {
                 def server = Artifactory.server 'artifactory-server'
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }"""

                 server.upload spec: uploadSpec
               }
            }

        }
```
**5. Deploy the application to the dev environment by launching Ansible pipeline
```
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
```

Create a task to set up the Dev environment and deploy artifact to webserver (todo server)
```
# # tasks file for webserver - php-todo
---
# Install Apache
- name: Install Apache
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name: httpd
    state: present

# Install EPEL release
- name: Install EPEL release
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name: https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    state: present

# Install dnf-utils and Remi repository
- name: Install dnf-utils and Remi repository
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name:
      - dnf-utils
      - http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    state: present

# Reset PHP module
- name: Reset PHP module
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: dnf module reset php -y

# Enable PHP 7.4 module
- name: Enable PHP 7.4 module
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: dnf module enable php:remi-7.4 -y

# Install PHP and extensions
- name: Install PHP and extensions
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name:
      - php
      - php-opcache
      - php-gd
      - php-curl
      - php-mysqlnd
      - php-common
      - php-mbstring
      - php-intl
      - php-xml
      - php-fpm
      - php-json
    state: present
    enablerepo: remi-7.4

# Ensure PHP-FPM service is started and enabled
- name: Ensure PHP-FPM service is started and enabled
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: php-fpm
    state: started
    enabled: true

# Set SELinux boolean for httpd_execmem
- name: Set SELinux boolean for httpd_execmem
  remote_user: ec2-user
  become: true
  ansible.builtin.seboolean:
    name: httpd_execmem
    state: true
    persistent: yes

# Ensure httpd service is started and enabled
- name: Ensure httpd service is started and enabled
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

# Download the artifact
- name: Download the artifact
  remote_user: ec2-user
  become: true
  ansible.builtin.get_url:
    url: http://54.88.110.0:8082/artifactory/Todo-artifact-local/php-todo.zip
    dest: /home/ec2-user/php-todo.zip
    url_username: admin
    url_password: Admin123$

- name: Ensure unzip is installed
  become: yes
  ansible.builtin.yum:
    name: unzip
    state: present
  when: ansible_os_family == "RedHat"

- name: Unzip the artifacts
  ansible.builtin.unarchive:
    src: /home/ec2-user/php-todo.zip
    dest: /var/www/html/
    remote_src: yes

# Unzip the artifacts
- name: Unzip the artifacts
  ansible.builtin.unarchive:
    src: /home/ec2-user/php-todo.zip
    dest: /home/ec2-user/
    remote_src: yes
  become: true

# Run chown on /var/www/html
- name: Run chown on /var/www/html
  ansible.builtin.command:
    cmd: chown -R $USER:$USER /var/www/html
  become: true

# Deploy the code
- name: Deploy the code
  remote_user: ec2-user
  become: true
  ansible.builtin.copy:
    src: /home/ec2-user/var/lib/jenkins/workspace/php-todo-pipeline/.
    dest: /var/www/html/
    force: yes
    remote_src: yes

# Delete the zip file after unzipping
- name: Delete the zip file after unzipping
  ansible.builtin.file:
    path: /home/ec2-user/php-todo.zip
    state: absent
  become: true

# Delete the unzipped files after copying
- name: Delete the unzipped files after copying
  ansible.builtin.file:
    path: /home/ec2-user/php-todo/
    state: absent
  become: true

# Remove Apache default welcome page on RedHat-based systems
- name: Remove Apache default welcome page
  remote_user: ec2-user
  become: true
  ansible.builtin.file:
    path: /etc/httpd/conf.d/welcome.conf
    state: absent
  when: ansible_os_family == "RedHat"

# Restart httpd
- name: Restart httpd
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: httpd
    state: restarted

# Verify PHP version
- name: Verify PHP version
  remote_user: ec2-user
  become: true
  ansible.builtin.command: php -v
  register: php_version

# Display PHP version
- name: Display PHP version
  remote_user: ec2-user
  become: true
  ansible.builtin.debug:
    msg: "{{ php_version.stdout }}"

# Set SELinux context for web directory and PHP files
- name: Set SELinux context for web directory
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chcon -R -t httpd_sys_content_t /var/www/html

- name: Set SELinux context for PHP writable files
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chcon -R -t httpd_sys_rw_content_t /var/www/html

# Generate a new APP_KEY if missing or invalid
- name: Generate a new APP_KEY if missing or invalid
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: php artisan key:generate
  args:
    chdir: /var/www/html

# Clear the configuration cache
- name: Clear the configuration cache
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: php artisan config:cache
  args:
    chdir: /var/www/html

# Ensure .env file and config directory have correct permissions
- name: Ensure .env file and config directory have correct permissions - chown
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chown -R apache:apache /var/www/html/

- name: Ensure .env file and config directory have correct permissions - chmod
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chmod -R 775 /var/www/html/

# Restart the web server and PHP-FPM
- name: Restart the web server
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: httpd
    state: restarted

- name: Restart PHP-FPM
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: php-fpm
    state: restarted
```

Run the pipeline
![image](https://github.com/user-attachments/assets/ebfa4aaf-aa4b-44d3-854f-a882c6262fa6)

**Check the artifactory for the uploaded artifact (php-todo.zip)
![image](https://github.com/user-attachments/assets/fb62b883-69b6-4a40-8443-8b8dfe3a84c1)


Visit a browser using the publis IP address to access the todo application

### SonarQube Installation
**Install SonarQube on Ubuntu 20.04 With PostgreSQL as Backend Database
Here is a manual approach to installation. Ensure that you can to automate the same with Ansible.

Below is a step by step guide how to install SonarQube 7.9.3 version. It has a strong prerequisite to have Java installed since the tool is Java-based. MySQL support for SonarQube is deprecated, therefore we will be using PostgreSQL.

We will make some Linux Kernel configuration changes to ensure optimal performance of the tool - we will increase vm.max_map_count, file discriptor and ulimit.

**Tune Linux Kernel
This can be achieved by making session changes which does not persist beyond the current session terminal.
```
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
**To make a permanent change, edit the file /etc/security/limits.conf and append the below
```
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```
**Before installing, let us update and upgrade system packages
```
sudo apt-get update
sudo apt-get upgrade
```
**Install wget and unzip packages
``
sudo apt-get install wget unzip -y
``
**Install OpenJDK and Java Runtime Environment (JRE) 11
```
sudo apt-get install openjdk-11-jdk -y
sudo apt-get install openjdk-11-jre -y
```
**Set default JDK - To set default JDK or switch to OpenJDK enter below command:
```
sudo update-alternatives --config java
```
If you have multiple versions of Java installed, you should see a list like below:

Output
```
Selection    Path                                            Priority   Status

------------------------------------------------------------

  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode

  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode

  2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

* 3            /usr/lib/jvm/java-8-oracle/jre/bin/java          1081      manual mode
```
Type "1" to switch OpenJDK 11

**Verify the set JAVA Version:
```
java -version
```
#### Install and Setup PostgreSQL 10 Database for SonarQube
**The command below will add PostgreSQL repo to the repo list:
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
```
**Download PostgreSQL software
```
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```
**Start PostgreSQL Database Server
```
sudo systemctl start postgresql
```
Enable it to start automatically at boot time
```
sudo systemctl enable postgresql
```
Change the password for default postgres user (Pass in the password you intend to use, and remember to save it somewhere)
```
su - postgres
```
Create a new user by typing
```
createuser sonar
```
Switch to the PostgreSQL shell
```
psql
```

Set a password for the newly created user for SonarQube database
```
ALTER USER sonar WITH ENCRYPTED password 'sonar';
Create a new database for PostgreSQL database by running:
CREATE DATABASE sonarqube OWNER sonar;
Grant all privileges to sonar user on sonarqube Database.
grant all privileges on DATABASE sonarqube to sonar;
Exit from the psql shell:
\q
Install SonarQube on Ubuntu 20.04 LTS
Navigate to the tmp directory to temporarily download the installation files
cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip
Unzip the archive setup to /opt directory
sudo unzip sonarqube-7.9.3.zip -d /opt
Move extracted setup to /opt/sonarqube directory
sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube
Configure SonarQube
We cannot run SonarQube as a root user, if you run using root user it will stop automatically. The ideal approach will be to create a separate group and a user to run SonarQube

Create a group sonar
sudo groupadd sonar
Now add a user with control over the /opt/sonarqube directory
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
Open SonarQube configuration file using your favourite text editor (e.g., nano or vim)
sudo vim /opt/sonarqube/conf/sonar.properties
Find the following lines:
#sonar.jdbc.username=
#sonar.jdbc.password=
Uncomment them and provide the values of PostgreSQL Database username and password:


Edit the sonar script file and set RUN_AS_USER
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh


Now, to start SonarQube we need to do following:
Switch to sonar user
sudo su sonar


Move to the script directory
cd /opt/sonarqube/bin/linux-x86-64/
Run the script to start SonarQube
./sonar.sh start
Expected output shall be as:
$./sonar.sh status

SonarQube is running (176483).
To check SonarQube logs, navigate to /opt/sonarqube/logs/sonar.log directory
tail /opt/sonarqube/logs/sonar.log


Configure SonarQube to run as a systemd service
Stop the currently running SonarQube service
cd /opt/sonarqube/bin/linux-x86-64/
Run the script to start SonarQube
./sonar.sh stop


Create a systemd service file for SonarQube to run as System Startup.
sudo nano /etc/systemd/system/sonar.service
Add the configuration below for systemd to determine how to start, stop, check status, or restart the SonarQube service.
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target


Save the file and control the service with systemctl
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar


Visit sonarqube config file and uncomment the line of sonar.web.port=9000
sudo vi /opt/sonarqube/conf/sonar.properties


Access SonarQube
To access SonarQube using browser, type server's IP address followed by port 9000
http://server_IP:9000 OR http://localhost:9000
Login to SonarQube with default administrator username - admin and password - admin
  

