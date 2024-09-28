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

### **Step 1: Set Up Jenkins**

#### **1. Install Java**
Jenkins requires Java to run. Install Java on your Jenkins server:

```bash
sudo yum install java-11-openjdk-devel -y
java --version  # Verify the installation
```

#### **2. Install Jenkins**
Add the Jenkins repository and install Jenkins:

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

#### **3. Access Jenkins UI**
Open your browser and navigate to `http://<your_server_ip>:8080`. Follow the instructions to unlock Jenkins, using the initial admin password located at `/var/lib/jenkins/secrets/initialAdminPassword`. After unlocking, install the recommended plugins.

---

### **Step 2: Set Up PostgreSQL for SonarQube**

#### **1. Add PostgreSQL Repository**
To add the PostgreSQL repository for package management, run:

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
```

#### **2. Install PostgreSQL**
Update your package list and install PostgreSQL:

```bash
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib -y
```

#### **3. Start PostgreSQL**
Enable and start the PostgreSQL service:

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

#### **4. Configure PostgreSQL**
Switch to the `postgres` user to create the necessary database and user:

```bash
sudo -i -u postgres
```

**Create a new PostgreSQL user and database:**
```sql
CREATE USER sonar WITH ENCRYPTED PASSWORD 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
exit
```

---

### **Step 3: Install and Set Up SonarQube**

#### **1. Download SonarQube**
Navigate to the temporary directory and download SonarQube:

```bash
cd /tmp
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip
```

#### **2. Unzip and Move SonarQube**
Extract the downloaded file and move it to the desired location:

```bash
sudo unzip sonarqube-7.9.3.zip -d /opt
sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube
```

#### **3. Create a User and Group for SonarQube**
To run SonarQube safely, create a dedicated user and group:

```bash
sudo groupadd sonar
sudo useradd -c "SonarQube user" -d /opt/sonarqube -g sonar sonar
sudo chown -R sonar:sonar /opt/sonarqube
```

#### **4. Configure SonarQube Database Connection**
Open the SonarQube configuration file and update it with the PostgreSQL connection details:

```bash
sudo vim /opt/sonarqube/conf/sonar.properties
```

Uncomment and set the following properties:

```properties
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
```

#### **5. Start SonarQube**
Switch to the SonarQube user and start the service:

```bash
sudo su sonar
cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh start
```

Check the status to ensure it is running:

```bash
./sonar.sh status
```

---

### **Step 4: Automate Installation with Ansible**

#### **1. Create Ansible Playbook Structure**
Set up your Ansible project structure:

```bash
mkdir -p ~/ansible/roles/sonar/tasks
mkdir -p ~/ansible/roles/postgresql/tasks
mkdir -p ~/ansible/inventory
mkdir ~/ansible/playbook
```

#### **2. Write the Ansible Playbooks**

**PostgreSQL Playbook** (`~/ansible/roles/postgresql/tasks/main.yml`):

```yaml
---
- name: Install PostgreSQL
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - postgresql
    - postgresql-contrib

- name: Start PostgreSQL
  systemd:
    name: postgresql
    state: started
    enabled: yes

- name: Create SonarQube user
  become_user: postgres
  postgresql_user:
    name: sonar
    password: sonar
    encrypted: yes

- name: Create SonarQube database
  become_user: postgres
  postgresql_db:
    name: sonarqube
    owner: sonar
```

**SonarQube Playbook** (`~/ansible/roles/sonar/tasks/main.yml`):

```yaml
---
- name: Download SonarQube
  get_url:
    url: https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip
    dest: /tmp/sonarqube.zip

- name: Extract SonarQube
  unarchive:
    src: /tmp/sonarqube.zip
    dest: /opt
    remote_src: yes

- name: Change ownership of SonarQube
  file:
    path: /opt/sonarqube
    state: directory
    owner: sonar
    group: sonar
    recurse: yes

- name: Configure SonarQube
  lineinfile:
    path: /opt/sonarqube/conf/sonar.properties
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
  with_items:
    - { regexp: '^#sonar.jdbc.username=', line: 'sonar.jdbc.username=sonar' }
    - { regexp: '^#sonar.jdbc.password=', line: 'sonar.jdbc.password=sonar' }
    - { regexp: '^#sonar.jdbc.url=', line: 'sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube' }

- name: Start SonarQube
  become_user: sonar
  command: /opt/sonarqube/bin/linux-x86-64/sonar.sh start
```

**Playbook to Run** (`~/ansible/playbook/site.yml`):

```yaml
---
- hosts: all
  roles:
    - postgresql
    - sonar
```

#### **3. Run Your Ansible Playbook**
Execute the playbook to automate the installation:

```bash
ansible-playbook -i ~/ansible/inventory/ci.yml ~/ansible/playbook/site.yml
```

---

### **Step 5: Configure Jenkins and SonarQube Integration**

#### **1. Install SonarQube Scanner Plugin in Jenkins**
Navigate to `Manage Jenkins > Manage Plugins`, search for "SonarQube Scanner" and install it.

#### **2. Configure SonarQube in Jenkins**
Go to `Manage Jenkins > Configure System`, scroll to the "SonarQube servers" section, and add a new server with the following details:
- **Name**: SonarQube
- **Server URL**: `http://<your_server_ip>:9000`
- **Server Authentication Token**: Generate a token in SonarQube (User > My Account > Security > Generate Tokens).

#### **3. Set Up Jenkins Pipeline**
In your Jenkins project, create or edit the Jenkinsfile to include the SonarQube stage. Below is a sample pipeline stage for the SonarQube Quality Gate:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Your build steps here
            }
        }

        stage('SonarQube Quality Gate') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // Add more stages as needed...
    }
}
```

---

### **Step 6: Test the Complete Pipeline**

#### **1. Push Changes to Your Repository**
Ensure your Jenkinsfile is in your repository and push any changes.

#### **2. Trigger the Jenkins Pipeline**
Go to Jenkins and manually trigger the pipeline or configure a webhook in your Git repository settings to trigger the pipeline on each push.

---

### **Step 7: Set Up Jenkins Agents/Slaves**

#### **1. Add Jenkins Agents**
To distribute your builds, add Jenkins agents:

- **Install Java on Slave Nodes:**
```bash
sudo yum install java-11

-openjdk-devel -y
```

- **Connect Agent Nodes to Jenkins:**
  - Go to `Manage Jenkins > Manage Nodes and Clouds > New Node`.
  - Configure the agent with the necessary details, and set up the agent using the instructions provided by Jenkins.

---

### **Additional Considerations**

- **Backup and Monitoring**: Set up backups for your PostgreSQL database and SonarQube configurations.
- **Security**: Configure security settings in SonarQube and Jenkins, especially for user authentication and permissions.
- **Plugins**: Explore and install additional plugins in Jenkins to enhance functionality as needed.
- **Documentation**: Keep documentation of your setup and configuration changes for future reference.

---

This guide offers a comprehensive approach to integrating SonarQube, Jenkins, and PostgreSQL, with detailed steps and considerations for each component. If you need more information on specific sections or further elaboration on any steps, feel free to ask!
