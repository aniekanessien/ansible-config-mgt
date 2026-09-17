# Project 11 - Ansible Configuration Management & Jenkins CI/CD Automation

> Centralized Linux configuration management, SSH automation, infrastructure validation, and GitHub-to-Jenkins CI/CD integration using Ansible.

---

## Project Overview

This project was built as part of my hands-on DevOps learning journey to move beyond simply writing Ansible YAML files and actually build, troubleshoot, validate, and document an automated infrastructure workflow.

The objective was to build a centralized configuration-management environment using **Ansible**, connect it to **Jenkins**, integrate Jenkins with **GitHub**, and eventually create an automated workflow where a GitHub push could trigger Jenkins and execute Ansible against multiple Linux servers

The project evolved into more than just running a playbook. I had to configure SSH access, manage host keys, validate Ansible connectivity, build and test an Ansible playbook, configure Jenkins, troubleshoot Jenkins disk-space and executor problems, configure GitHub integration, create a GitHub webhook, and finally validate that a Git push could automatically trigger a Jenkins build.

The project also became a practical troubleshooting exercise.

During the implementation, I encountered issues involving Jenkins disk space, Jenkins node availability, executors, workspaces, SSH host-key verification, Git authentication, Ansible connectivity and webhook configuration.

Rather than hiding those failures, I documented them as part of the project because troubleshooting is an important part of real-world DevOps work

The final result is a working workflow where:

**GitHub Push → GitHub Webhook → Jenkins → Git Checkout → Ansible Playbook → Managed Linux Servers**

---

# Table of Contents

- [Project Overview](#project-overview)
- [Project Objectives](#project-objectives)
- [Architecture](#architecture)
- [Infrastructure](#infrastructure)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Environment Setup](#environment-setup)
- [AWS Infrastructure](#aws-infrastructure)
- [SSH Configuration](#ssh-configuration)
- [Ansible Configuration](#ansible-configuration)
- [Inventory](#inventory)
- [Playbook](#playbook)
- [Connectivity Validation](#connectivity-validation)
- [Jenkins Integration](#jenkins-integration)
- [Jenkins CI/CD Workflow](#jenkins-cicd-workflow)
- [GitHub Webhook Integration](#github-webhook-integration)
- [Troubleshooting Journey](#troubleshooting-journey)
- [Problems Encountered and Solutions](#problems-encountered-and-solutions)
- [Final Validation](#final-validation)
- [Skills Demonstrated](#skills-demonstrated)
- [Security Considerations](#security-considerations)
- [Lessons Learned](#lessons-learned)
- [Future Improvements](#future-improvements)
- [Project Outcome](#project-outcome)

---

# Project Objectives

The main objectives of this project were to:

* Deploy and configure an Ansible controller.
* Manage multiple Linux servers from one central controller.
* Configure SSH-based communication.
* Build an Ansible inventory.
* Create reusable Ansible playbooks.
* Automate common server configuration.
* Validate connectivity using Ansible.
* Install and configure Jenkins.
* Connect Jenkins to GitHub.
* Configure Jenkins to execute Ansible.
* Configure a GitHub webhook.
* Trigger Jenkins automatically after Git pushes.
* Troubleshoot infrastructure failures.
* Document the complete implementation with evidence.


---

## Project Architecture

The final workflow looks like this:

```text
                         DEVELOPER
                             |
                             | git push
                             v
                      +---------------+
                      |    GitHub     |
                      | Repository    |
                      +-------+-------+
                              |
                              | Webhook
                              v
                      +---------------+
                      |    Jenkins    |
                      |     CI/CD     |
                      +-------+-------+
                              |
                              | Git checkout
                              v
                      +---------------+
                      |    Jenkins    |
                      |   Workspace   |
                      +-------+-------+
                              |
                              | Execute Ansible
                              v
                      +---------------+
                      |    Ansible    |
                      |   Controller  |
                      +-------+-------+
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
       +-----------+    +-----------+    +-----------+
       | Database  |    |    NFS    |    |   Load    |
       |  Server   |    |  Server   |    | Balancer  |
       +-----------+    +-----------+    +-----------+
             |                |                |
             +----------------+----------------+
                              |
                     +--------+--------+
                     |                 |
                     v                 v
               +-----------+     +-----------+
               | WebServer1|     | WebServer2|
               +-----------+     +-----------+
````

The intended automation flow is:

```text
GitHub Push
     ↓
GitHub Webhook
     ↓
Jenkins
     ↓
Git Checkout
     ↓
Ansible Playbook
     ↓
SSH
     ↓
Managed Linux Servers
```
---

# Technology Stack

## Cloud Infrastructure

* Amazon Web Services
* Amazon EC2
* AWS Security Groups

## Operating Systems

* Ubuntu Linux
* RHEL-based Linux systems

## Configuration Management

* Ansible
* YAML
* SSH
* Ansible Inventory
* Ansible Playbooks

## CI/CD

* Jenkins
* Jenkins Executors
* Jenkins Workspaces
* Jenkins Git SCM
* Jenkins Build Jobs
* GitHub Webhooks

## Source Control

* Git
* GitHub

## Supporting Tools

* OpenSSH
* ssh-keyscan
* apt
* yum
* Wireshark
* Linux CLI

---

# Infrastructure

The environment consisted of an Ansible/Jenkins controller and multiple managed Linux servers.

The managed infrastructure included:

| Server             | Role                          |
| ------------------ | ----------------------------- |
| Ansible Controller | Central automation controller |
| Jenkins            | CI/CD automation              |
| Database           | Database server               |
| Load Balancer      | Load-balancing server         |
| NFS Server         | Shared storage server         |
| Web Server 1       | Web server                    |
| Web Server 2       | Web server                    |

---

# AWS Environment

The first stage was confirming that the required EC2 infrastructure was running.

![AWS EC2 Instances](screenshots/screenshots/01-aws-ec2-instances-running.png)

I also reviewed the AWS Security Group configuration to ensure that the required connectivity between the controller, Jenkins and managed servers was possible.

![AWS Security Group Configuration](screenshots/screenshots/02-jenkins-ansible-security-group.png)

This stage reinforced an important infrastructure principle:

> Before troubleshooting an application, verify the underlying network and security configuration.

---

# Jenkins Installation

Jenkins was installed on the Linux environment and accessed through the browser.

![Jenkins Installed](screenshots/screenshots/04-jenkins-installed.png)

![Jenkins-plugins-ready](screenshots/screenshots/05-jenkins-plugins-ready.png)

The Jenkins interface was then verified.

![Jenkins Welcome Interface](screenshots/screenshots/06-jenkins-welcome-interface.png)

The Jenkins service was also checked from the Linux terminal.

```bash
sudo systemctl status jenkins
```

The service was confirmed to be running.

![Jenkins Service Running](screenshots/screenshots/20-jenkins-service-running.png)

---

# Ansible Installation

Ansible was installed on the controller.

I verified the installation using:

```bash
ansible --version
```

![Ansible Version](screenshots/screenshots/07-ansible-version-installed.png)

This confirmed that Ansible was available from the controller and ready for configuration.

---

# Git Installation

Git was installed and verified because Jenkins would eventually use Git to retrieve the project source code.

```bash
git --version
```

![Git Version](screenshots/screenshots/08-git-version-installed.png)

---

# SSH Configuration

SSH was one of the most important components of the project.

Ansible communicates with Linux managed nodes primarily through SSH, so I first had to establish reliable SSH connectivity before attempting automation.

The SSH key configuration was completed on the controller.

![SSH Key Configuration](screenshots/screenshots/09-ssh-key-configuration-successful.png)

---

## SSH Access to Web Server 1

The controller was tested against Web Server 1.

![Ansible SSH Access Web Server 1](screenshots/screenshots/10-ansible-ssh-access-webserver1.png)

---

## SSH Access to Web Server 2

The second web server was also tested.

![Ansible SSH Access Web Server 2](screenshots/screenshots/11-ansible-keyless-ssh-webserver2.png)

---

## SSH Access to NFS Server

The NFS server was also tested from the controller.

![Ansible SSH Access NFS Server](screenshots/screenshots/12-ansible-keyless-ssh-nfs-server.png)

---

# Ansible Inventory

After establishing SSH connectivity, I created an Ansible inventory to represent the managed infrastructure.

```text
inventory/
└── dev.yml
```

The inventory allowed me to address multiple servers centrally rather than manually connecting to every server.

![Ansible Inventory](screenshots/screenshots/13-ansible-jenkins-inventory-tree.png)

---

# Ansible Connectivity Test

Before running configuration tasks, I verified that Ansible could communicate with the managed nodes.

The command used was:

```bash
ansible all -i inventory/dev.yml -m ping
```

Successful hosts returned:

```text
SUCCESS
"ping": "pong"
```

![Ansible Ping Successful](screenshots/screenshots/14-ansible-connectivity-ping-success.png)

This was an important checkpoint.

At this point, the controller could communicate with the infrastructure through Ansible.

---

# Building the Common Playbook

The main configuration playbook was created under:

```text
playbooks/common.yml
```

![Common Playbook Created](screenshots/screenshots/15-common-playbook-created.png)

The playbook contains common configuration tasks that can be applied across the managed infrastructure.

One of the tasks involved installing Wireshark.

The playbook also demonstrated privilege escalation:

```yaml
become: yes
```

This allowed configuration tasks requiring elevated privileges to execute correctly.

---

# Ansible Playbook Execution

The playbook was tested manually before introducing Jenkins automation.

```bash
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

The playbook executed successfully across the infrastructure.

![Ansible Common Playbook Successful](screenshots/screenshots/16-ansible-common-playbook-success.png)

Wireshark installation was also validated.

![Wireshark Installed](screenshots/screenshots/17-wireshark-installed-webserver.png)

---

# SSH Host Configuration

The SSH configuration between the automation environment and managed infrastructure was further validated.

![SSH Host Configuration](screenshots/screenshots/18-ansible-ssh-host-configuration.png)

The resulting configuration was reflected in the common playbook.

![Common YAML Configuration](screenshots/screenshots/19-common-playbook-configuration.png)

---

# Jenkins Configuration

Once Ansible was working manually, I moved to the CI/CD stage.

The Jenkins job was configured to retrieve the project from GitHub.

The repository was configured under:

```text
Jenkins
    ↓
Configure
    ↓
Source Code Management
    ↓
Git
```

The repository was connected to GitHub and the `main` branch was configured.

---

# Jenkins Workspace

Jenkins uses a workspace to clone the repository and execute build steps.

The workspace used for this project was:

```text
/var/lib/jenkins/workspace/ansible
```

This became important later during troubleshooting because a Jenkins job cannot execute normally if the workspace or node is unavailable.

---

# Jenkins Executors

I configured the Built-In Node with two executors.

Navigation:

```text
Jenkins
    ↓
Nodes
    ↓
Built-In Node
    ↓
Configure
```

The executor configuration was:

```text
Number of executors: 2
```

Executors are the resources Jenkins uses to run builds.

This also helped me understand why a build can sometimes remain in:

```text
Waiting for next available executor
```

If the node is offline or the executors are unavailable, the build remains queued.

---

# Jenkins Build Process

The Jenkins job was configured to execute the Ansible automation through a shell build step.

The workflow was:

```text
1. Jenkins starts build
2. Jenkins checks out GitHub repository
3. Jenkins creates/uses workspace
4. Jenkins executes shell command
5. Ansible loads inventory
6. Ansible executes playbook
7. SSH connects to managed servers
8. Ansible returns play recap
9. Jenkins reports build status
```

---

# First Jenkins Build

The first Jenkins build did not succeed.

The console output showed that Jenkins could retrieve the repository but encountered problems during the build process.

![Jenkins Build Error](screenshots/screenshots/41-jenkins-node-build-error.png)

![Jenkins Console Output Error](screenshots/screenshots/43-jenkins-console-output-error..png)

This became the beginning of the troubleshooting phase of the project.

Instead of rebuilding everything from scratch, I started investigating each layer independently.

---

# Troubleshooting Journey

This project gave me practical exposure to troubleshooting several interconnected systems.

The major issues I encountered included:

* Jenkins disk-space threshold
* `/tmp` disk-space limitation
* Jenkins node going offline
* Jenkins executor availability
* Jenkins workspace problems
* SSH host-key verification
* Jenkins SSH permissions
* Git authentication
* Ansible SSH connectivity
* GitHub webhook configuration
* Automated Jenkins builds

---

# Troubleshooting Jenkins Disk Space

One of the major problems was Jenkins reporting insufficient disk space.

The Jenkins node reported that the temporary filesystem had fallen below the configured threshold.

I investigated the filesystem using:

```bash
df -h
```

and:

```bash
df -h /tmp
```

The `/tmp` filesystem was the immediate problem.

![Jenkins Disk Space Threshold](screenshots/screenshots/39-jenkins-disk-threshold-correction.png)

![Jenkins Disk Space Fixed](screenshots/screenshots/44-jenkins-disk-space-fixed.png)

The temporary files were cleaned and the Jenkins service was restarted.

```bash
sudo rm -rf /tmp/*
sudo systemctl restart jenkins
```

The filesystem was then checked again.

![Jenkins Node Disk Space Corrected](screenshots/screenshots/40-jenkins-node-disk-space-correction.png)

This was an important lesson for me because the Jenkins application itself was healthy, but the underlying Linux filesystem prevented Jenkins from executing builds.

---

# Jenkins Node Going Offline

At one point, the Jenkins Built-In Node was marked offline.

The Jenkins interface showed that the node was unavailable.

![Jenkins Node Offline Error](screenshots/screenshots/44-jenkins-node-offline-error.png)

The node status was investigated from:

```text
Jenkins
    ↓
Nodes
    ↓
Built-In Node
```

The monitoring information showed that temporary disk space was still an issue.

After correcting the filesystem condition and restarting Jenkins, the node became available again.

The node eventually showed healthy monitoring information.

---

# Jenkins Service Verification

The Jenkins service was repeatedly verified from the Linux terminal during troubleshooting.

```bash
sudo systemctl status jenkins
```

The important status was:

```text
Active: active (running)
```

![Jenkins Service Status](screenshots/screenshots/20-jenkins-service-running.png)

This helped separate Jenkins application problems from infrastructure-level problems.

---

# Jenkins Workspace Troubleshooting

Another issue was:

```text
Error: no workspace
```

This occurred because Jenkins could not find or use the expected workspace.

The workspace path was checked from Linux:

```bash
sudo ls -la /var/lib/jenkins
```

and:

```bash
sudo ls -la /var/lib/jenkins/workspace
```

The workspace was eventually recreated when Jenkins successfully executed a new build.

This taught me that the Jenkins workflow depends on several layers:

```text
Jenkins Job
    ↓
Jenkins Node
    ↓
Executor
    ↓
Workspace
    ↓
Git Checkout
    ↓
Build Step
```

A problem at any layer can stop the build.

---

# SSH Host-Key Verification Problem

Another significant error occurred when Ansible attempted to connect to the managed servers.

The error was:

```text
Host key verification failed
```

![Ansible SSH Connectivity Error](screenshots/screenshots/42-ansible-ping-connectivity-error.png)

The problem occurred because Jenkins runs under the `jenkins` Linux service account.

The SSH trust configuration therefore had to exist for the Jenkins user rather than only for my normal Ubuntu user.

The Jenkins SSH directory was configured under:

```text
/var/lib/jenkins/.ssh/
```

and:

```text
/var/lib/jenkins/.ssh/known_hosts
```

![Jenkins Known Hosts](screenshots/screenshots/21-jenkins-known-hosts-created.png)

---

# Jenkins SSH Key Configuration

The Jenkins environment also needed the appropriate SSH key configuration.

![Jenkins SSH Key Distribution](screenshots/screenshots/22-jenkins-ssh-key-distributed.png)

The private key was configured for the Jenkins automation workflow.

![Jenkins Private Key Configuration](screenshots/screenshots/23-jenkins-private-key-configured.png)

This reinforced an important concept:

> An SSH connection working from my personal shell does not automatically mean Jenkins can connect.

Jenkins has its own service account, environment, permissions and SSH configuration.

---

# Successful SSH Validation

After correcting the SSH configuration, I tested direct SSH access from the Jenkins environment.

The connection was successfully established with the load-balancer host.

This confirmed that Jenkins had the necessary SSH access to continue with the Ansible automation.

---

# Ansible Connectivity Revalidation

After the SSH troubleshooting, I reran the Ansible connectivity test.

```bash
ansible all -i inventory/dev.yml -m ping
```

The managed nodes returned successful responses.

```text
ping: pong
```

This proved that the problem was no longer with basic Ansible connectivity.

---

# Git Authentication Troubleshooting

During the project, I also encountered a Git authentication problem.

The error included:

```text
Permission denied (publickey)
```

and:

```text
Could not read from remote repository.
```

I investigated the repository configuration using:

```bash
git remote -v
```

The Git remote was initially configured in a way that was not working with the authentication method being used.

I corrected the remote configuration and verified it again.

```bash
git remote -v
```

The repository was then successfully pushed to GitHub.

The final Git state showed:

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

![Final Git Commit Status](screenshots/screenshots/38-final-git-commit-project-status.png)

---

# Jenkins Build Recovery

After correcting the Jenkins node, SSH and workspace issues, the Jenkins builds began executing successfully.

![Jenkins Build Restored](screenshots/screenshots/24-jenkins-node-build-restored.png)

The build console output showed Jenkins successfully checking out the repository and executing the automation workflow.

![Jenkins Build Console](screenshots/screenshots/25-jenkins-build4-console-output.png)

---

# Successful Jenkins Builds

The following builds were successfully executed during the validation stage.

![Jenkins Build 5 Successful](screenshots/screenshots/26-jenkins-build5-successful.png)

The console output was also inspected to confirm that the build was actually executing the expected Ansible tasks.

![Jenkins Build 5 Console Output](screenshots/screenshots/27-jenkins-build5-console-output.png)

---

# GitHub Webhook Integration

The next stage was removing the need to manually click:

```text
Build Now
```

for every change.

I configured a GitHub webhook to notify Jenkins when changes were pushed to the repository.

The webhook was created under:

```text
GitHub Repository
    ↓
Settings
    ↓
Webhooks
```

![GitHub Webhook Created](screenshots/screenshots/28-github-webhook-created.png)

---

# Jenkins Webhook Trigger

The Jenkins job was configured to accept GitHub webhook events.

The objective was to create this workflow:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Jenkins
    |
    | Automatic build
    v
Ansible
    |
    | SSH
    v
Managed Servers
```

---

# Webhook Trigger Test

The GitHub webhook was tested by pushing changes to the repository.

![GitHub Webhook Push Trigger](./screenshots/30-github-webhook-trigger-push.png)

The webhook successfully caused Jenkins to initiate a new build.

![Automated Jenkins Webhook Build](screenshots/screenshots/30-github-webhook-trigger-push.png)

---

# Automated Jenkins Builds

The automated build process was then tested repeatedly.

The Jenkins build history showed multiple builds being generated as repository changes were pushed.

![Jenkins Automated Build](./screenshots/32-jenkins-automated-build5-ansible.png)

The Ansible execution was also visible in the Jenkins console.

![Jenkins Automated Build Validation](screenshots/screenshots/32-jenkins-automated-build5-ansible.png)

---

# Automated Ansible Deployment

The Jenkins job successfully executed the Ansible deployment workflow.

![Jenkins Ansible Deployment Build](screenshots/screenshots/34-jenkins-ansible-deployment-build.png)

This demonstrated that the automation was no longer dependent on manually running the Ansible playbook from my terminal.

---

# Successful Webhook Build

The final webhook-triggered Jenkins build completed successfully.

![Successful Jenkins Webhook Build](screenshots/screenshots/35-jenkins-successful-webhook-build..png)

The webhook activity was also reviewed from Jenkins.

![Jenkins Webhook Activity](screenshots/screenshots/36-jenkins-webhook-activity-log.png)

GitHub also reported successful webhook delivery.

![GitHub Webhook Delivery](screenshots/screenshots/37-github-webhook-recent-delivery-success.png)

---

# Final Ansible Validation

The final Ansible playbook execution returned successful results across the managed infrastructure.

The play recap showed:

```text
database       ok=2   changed=0   unreachable=0   failed=0
loadbalancer   ok=2   changed=0   unreachable=0   failed=0
nfs_server     ok=2   changed=0   unreachable=0   failed=0
web1           ok=2   changed=0   unreachable=0   failed=0
web2           ok=2   changed=0   unreachable=0   failed=0
```

The most important values were:

```text
unreachable=0
failed=0
```
![Final Ansible Validation](screenshots/screenshots/35-jenkins-successful-webhook-build..png)
This confirmed that Ansible could successfully communicate with all managed nodes and execute the configuration tasks.

---

# Final Jenkins Result

The final Jenkins execution reported:

```text
PROJECT 11 COMPLETED SUCCESSFULLY

Finished: SUCCESS
```

This represented the final successful validation of the project.

---

# GitHub Repository Validation

The final repository state was also verified.

```bash
git status
```

The result was:

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

The project documentation was committed and pushed to GitHub.

![Final Git Repository Status](screenshots/screenshots/38-final-git-commit-project-status.png)



---

# Repository Structure

The final project is organized as follows:

```text
ansible-config-mgt/
│
├── inventory/
│   └── dev.yml
│
├── playbooks/
│   └── common.yml
│
├── screenshots/
│   ├── 01-aws-ec2-instances-running.png
│   ├── 02-jenkins-ansible-security-group.png
│   ├── 03-jenkins-controller-ssh-login.png
│   ├── 04-jenkins-installed.png
│   ├── 05-jenkins-plugins-ready.png
│   ├── 06-jenkins-welcome-interface.png
│   ├── 07-ansible-version-installed.png
│   ├── 08-git-version-installed.png
│   ├── 09-ssh-key-configuration-successful.png
│   ├── 10-ansible-ssh-access-webserver1.png
│   ├── 11-ansible-keyless-ssh-webserver2.png
│   ├── 12-ansible-keyless-ssh-nfs-server.png
│   ├── 13-ansible-jenkins-inventory-tree.png
│   ├── 14-ansible-connectivity-ping-success.png
│   ├── 15-common-playbook-created.png
│   ├── 16-ansible-common-playbook-success.png
│   ├── 17-wireshark-installed-webserver.png
│   ├── 18-ansible-ssh-host-configuration.png
│   ├── 19-common-yml-configuration.png
│   ├── 20-jenkins-service-running.png
│   ├── 21-jenkins-known-hosts-created.png
│   ├── 22-jenkins-ssh-key-distributed.png
│   ├── 23-jenkins-private-key-configured.png
│   ├── 24-jenkins-node-build-restored.png
│   ├── 25-jenkins-build4-console-output.png
│   ├── 26-jenkins-build5-successful.png
│   ├── 27-jenkins-build5-console-output.png
│   ├── 28-github-webhook-created.png
│   ├── 29-jenkins-readme-content.png
│   ├── 30-github-webhook-trigger-push.png
│   ├── 31-automated-jenkins-webhook-build.png
│   ├── 32-jenkins-automated-build5-ansible.png
│   ├── 33-jenkins-automated-build-validation.png
│   ├── 34-jenkins-ansible-deployment-build.png
│   ├── 35-jenkins-successful-webhook-build.png
│   ├── 36-jenkins-webhook-activity-log.png
│   ├── 37-github-webhook-recent-delivery-success.png
│   ├── 38-final-git-commit-project-status.png
│   ├── 39-jenkins-disk-threshold-correction.png
│   ├── 40-jenkins-node-disk-space-correction.png
│   ├── 41-jenkins-node-build-error.png
│   ├── 42-ansible-ping-connectivity-error.png
│   ├── 43-jenkins-console-output-error.png
│   ├── 44-jenkins-disk-space-fixed.png
│   └── 45-jenkins-node-offline-error.png
│
└── README.md
```

---

# Skills Demonstrated

This project allowed me to develop practical experience in the following areas.

## Linux Administration

* Linux command-line administration
* Systemd service management
* Filesystem troubleshooting
* Disk-space analysis
* `/tmp` filesystem management
* Linux permissions
* User management
* SSH administration
* Service troubleshooting

## AWS

* EC2
* Security Groups
* Linux server provisioning
* Network connectivity
* Cloud infrastructure troubleshooting

## Ansible

* Ansible installation
* Inventory management
* YAML
* Playbook development
* Package management
* `apt`
* `yum`
* SSH automation
* Privilege escalation
* Connectivity testing
* Multi-server configuration management

## Jenkins

* Jenkins installation
* Jenkins service management
* Jenkins nodes
* Jenkins executors
* Jenkins workspaces
* Git SCM
* Build jobs
* Shell build steps
* Build history
* Console output analysis
* Jenkins troubleshooting

## Git and GitHub

* Git repositories
* Branch management
* Git commits
* Git history
* Git remotes
* Git push
* GitHub repository management
* GitHub Webhooks

## CI/CD

* Event-driven builds
* Source-code-triggered automation
* GitHub-to-Jenkins integration
* Automated Ansible execution
* Build validation
* Infrastructure automation

---

# Troubleshooting Skills Demonstrated

One of the most valuable parts of this project was not simply getting the final green build.

It was learning how to investigate failures systematically.

I worked through problems involving:

```text
Jenkins
   ↓
Node
   ↓
Executor
   ↓
Workspace
   ↓
Git
   ↓
SSH
   ↓
Ansible
   ↓
Managed Infrastructure
```

When something failed, I learned to isolate the affected layer rather than immediately changing everything.

For example, when Jenkins reported:

```text
Waiting for next available executor
```

I learned to investigate the Jenkins node rather than assuming the Ansible playbook was broken.

When Ansible reported:

```text
Host key verification failed
```

I investigated the SSH environment of the Jenkins user rather than only testing SSH from my normal Ubuntu account.

When Jenkins went offline because of disk monitoring, I investigated the Linux filesystem using:

```bash
df -h
```

and:

```bash
df -h /tmp
```

These troubleshooting exercises were as valuable as the successful deployments.

---

# Security Considerations

This is a learning and portfolio environment, but security was still considered throughout the implementation.

Sensitive credentials should never be committed to GitHub.

The following should remain outside the repository:

```text
*.pem
*.key
.env
password files
AWS access keys
private SSH keys
credentials
```

For a production implementation, I would use:

* Jenkins Credentials
* Ansible Vault
* AWS IAM
* AWS Secrets Manager
* Restricted Security Groups
* SSH key-based authentication
* Protected GitHub branches
* Webhook secrets

---

# Production Improvements

There are several improvements I would make before using this architecture for production workloads.

## Ansible

I would introduce:

* Ansible Roles
* Environment-specific inventories
* Group variables
* Host variables
* Ansible Vault
* Molecule testing
* Better idempotency validation

## Jenkins

I would move from a basic Jenkins job to a Jenkins Pipeline using a `Jenkinsfile`.

A future pipeline would look like:

```text
Checkout
   ↓
Validate
   ↓
Ansible Syntax Check
   ↓
Connectivity Test
   ↓
Configuration Deployment
   ↓
Validation
```

I would also use dedicated Jenkins agents instead of relying heavily on the Built-In Node.

## AWS

For a more production-oriented environment:

* Terraform would be used for Infrastructure as Code.
* Managed servers would ideally sit in appropriate private subnets.
* Security Groups would be tightly restricted.
* IAM roles would be used instead of static credentials.
* Monitoring and alerting would be added.

---

# Future Architecture

My next evolution of this project would be:

```text
Developer
     |
     v
GitHub
     |
     v
Pull Request
     |
     v
Jenkins Pipeline
     |
     +----> Ansible Validation
     |
     +----> Security Checks
     |
     +----> Configuration Testing
     |
     v
Jenkins Agent
     |
     v
Ansible
     |
     v
AWS Infrastructure
     |
     v
Monitoring / Logging
```

This would take the project from a practical learning environment toward a more production-oriented DevOps architecture.

---

# What I Learned

This project changed the way I think about DevOps automation.

At first, it looked like a simple Ansible project:

```text
Write YAML
    ↓
Run Playbook
    ↓
Done
```

But the actual implementation showed me that a real automation environment is much more interconnected:

```text
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Node
   ↓
Executor
   ↓
Workspace
   ↓
Git
   ↓
Ansible
   ↓
SSH
   ↓
Linux Servers
```

A failure anywhere in this chain can prevent the final deployment from completing.

The troubleshooting process therefore became one of the most important parts of the project.

I learned to check:

* Service status
* Disk space
* Temporary filesystem space
* Node availability
* Executor availability
* Workspace availability
* SSH authentication
* SSH host keys
* Git remotes
* Ansible connectivity
* Jenkins console output
* GitHub webhook delivery

This is the kind of practical troubleshooting experience I want to continue building throughout my DevOps journey.

---

# Final Validation

The final project was validated at three levels.

## 1. Ansible

```bash
ansible all -i inventory/dev.yml -m ping
```

Expected result:

```text
ping: pong
```

---

## 2. Git

```bash
git status
```

Final state:

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

---

## 3. Jenkins

The final Jenkins build returned:

```text
PROJECT 11 COMPLETED SUCCESSFULLY

Finished: SUCCESS
```

The Ansible play recap also showed:

```text
unreachable=0
failed=0
```

---

# Project Outcome

The final result is a working infrastructure automation workflow in which GitHub changes can trigger Jenkins and Jenkins can execute Ansible configuration management against multiple Linux servers.

The complete workflow is:

```text
                    +-------------+
                    |   GitHub    |
                    +------+------+
                           |
                           | Webhook
                           v
                    +-------------+
                    |   Jenkins   |
                    +------+------+
                           |
                           | Git Checkout
                           v
                    +-------------+
                    |   Workspace |
                    +------+------+
                           |
                           | Execute
                           v
                    +-------------+
                    |   Ansible   |
                    +------+------+
                           |
                           | SSH
          +----------------+----------------+
          |                |                |
          v                v                v
      Database         NFS Server      Load Balancer
          |                                 |
          +----------------+----------------+
                           |
                    +------+------+
                    |             |
                    v             v
                Web Server 1  Web Server 2
```

The project successfully brought together:

```text
AWS
Linux
SSH
Git
GitHub
GitHub Webhooks
Jenkins
Jenkins Executors
Jenkins Workspaces
Ansible
YAML
Configuration Management
CI/CD
Infrastructure Troubleshooting
```

The most important takeaway for me was that DevOps is not only about making automation work.

It is also about understanding **why it fails, how to isolate the failure, how to recover the environment, and how to prove that the final system works.**

---

# Project Evidence

The `screenshots/` directory contains the implementation evidence collected throughout the project, including:

* AWS infrastructure
* Jenkins installation
* Ansible installation
* Git installation
* SSH configuration
* Ansible inventory
* Ansible connectivity
* Playbook development
* Jenkins configuration
* Jenkins failures
* Disk-space troubleshooting
* Node troubleshooting
* SSH troubleshooting
* Git troubleshooting
* GitHub webhook configuration
* Automated Jenkins builds
* Final successful deployment

I intentionally kept the troubleshooting evidence because it demonstrates the actual engineering process rather than showing only the final successful state.

---

# Author

## DevOps / Cloud Infrastructure Portfolio

**Project 11 — Ansible Configuration Management & Jenkins CI/CD Automation**

Built as part of my practical DevOps engineering journey.