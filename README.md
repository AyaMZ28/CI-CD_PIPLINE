# Automated Build and Test with Jenkins on EC2 Instance

This project focuses on configuring Jenkins to automate the building and testing of code changes sourced from a GitHub repository, using a Jenkinsfile. The development process adheres to the Git Flow branching model.

## Table of Contents
- [Create EC2 Instance](#create-ec2-on-aws)
- [Create GitHub Repository](#create-github-repository)
- [Install Git & Git Flow](#install-git--git-flow)
- [Create Jenkinsfile](#create-jenkinsfile)
- [Create Jenkins Container on EC2](#create-jenkins-container-on-ec2)
- [Configure Jenkins for GitHub Integration](#configure-jenkins-for-github-integration)
- [Create a New Pipeline Job in Jenkins](#create-a-new-pipeline-job-in-jenkins)
- [Implement Webhooks for CI](#implement-webhooks-for-ci)
- [Testing and Validation](#testing-and-validation)

## Create EC2 On AWS
- Launch an EC2 instance on AWS.
- Add an inbound rule to the security group for port 8080.

## Create GitHub Repository
- Create a new GitHub repository or select an existing one.
- Initialize the repository with a README.md, and ensure it contains at least two branches:
    - `develop` for ongoing development.
    - `master` (or `main`) for stable releases.

## Install Git & Git Flow
```bash
# Install git
$ sudo yum install git
$ git --version

# Install Git Flow
$ git clone --recursive https://github.com/petervanderdoes/gitflow-avh.git
$ cd gitflow-avh/contrib/
$ chmod +x gitflow-installer.sh
$ sudo ./gitflow-installer.sh install stable
$ sudo yum update
$ git flow init
$ git branch
```
- Clone GitHub Repo:
```bash
$ git clone "repo_url"
$ cd <path to cloned repo folder>
```

## Create Jenkinsfile
- Create a Jenkinsfile:
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```
- Save and exit.
- Push the file to the GitHub Repo:
```bash
$ git add Jenkinsfile
$ git commit -m "Add Jenkinsfile"
$ git push "repo_url" develop
```
If authentication fails, create a token and use it instead of a password.

## Create Jenkins Container on EC2
```bash
$ sudo yum update -y
$ sudo yum install docker -y
$ sudo systemctl start docker
$ sudo docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```
- Accessing Jenkins:
    - In your web browser, go to: `http://(ec2-public-ip):8080`
    - Retrieve the Jenkins password:
        1. Run the following command inside your container:
        ```bash
        $ cat /var/jenkins_home/secrets/initialAdminPassword
        ```
        2. Alternatively, run:
        ```bash
        $ sudo docker volume inspect jenkins_home
        ```
        Copy the path of the mount point and run:
        ```bash
        $ ls secrets
        ```
    - Copy the password to the Jenkins tab and sign in.

## Configure Jenkins for GitHub Integration
- Install necessary plugins in Jenkins such as Git and Pipeline.
- Connect Jenkins to GitHub:
    - Go to "Manage Jenkins" > "Manage Plugins" > "Available" and install "GitHub Integration Plugin".
    - Set up credentials in Jenkins for GitHub (username and token).

## Create a New Pipeline Job in Jenkins
- Select "New Item", name your pipeline (e.g., "GitHubpipeline"), and choose "Pipeline" as the type.
- In the pipeline configuration, select "Pipeline script from SCM" and choose "Git" as the SCM.
- Enter the repository URL and credentials.
- Specify the branch to build (e.g., */develop).
- In Build Triggers, check **"GitHub hook trigger for GITScm polling"**.
- Save.

## Implement Webhooks for CI
- In GitHub, go to your repository settings and select **"Webhooks"**.
- **Add a new webhook:**
    - Payload URL: `http://<your-jenkins-url>:8080/github-webhook/`
    - Content type: **application/json**
    - Select **"Just the push event"**.
    - Ensure the webhook is active.

## Testing and Validation
- Push a change to the 'develop' branch and verify Jenkins triggers a build.
- Check the Jenkins dashboard for build status and output.
```
      
