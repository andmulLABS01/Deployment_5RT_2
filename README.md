<h1 align="center">Deploy a retail banking application to an EC2 instance<h1> 


# Deployment 5 RT2
November 15, 2023

By: Andrew Mullen

## Purpose:

Practicing my ability to deploy a retail banking application to an EC2 instance and creating infrastructure, using TerraForm, for our Jenkins pipeline
and Flask application with a Guincorn webserver and SQLite database. 

## Steps:

### 1. Create a VPC with Terraform, and the VPC must have only the components listed: 1 VPC, 2 AZs, 2 Public Subnets, 2 EC2s, 1 Route Table, Security Group Ports: 8080, 8000, 22
   - This process is to give us practice using Terraform to create our AWS infrastructure using resource blocks.  This version of main.tf utilizes the default route table instead of creating a second one. Here is the link to the main.tf file: Click [HERE](https://github.com/andmulLABS01/Deployment_5RT_1/blob/main/DPRT1_main.tf)
   - I also ensured that only security group ports 22 and 8080 were applied to instance 1 and security group ports 22 and 8000 were applied to instance 2.

### 2. For the first instance follow the below instructions: Jenkins Server
   - The below instructions create public and private keys that will allow us to SSH into the second instance
     - This step acts as a Jenkins agent which allows us to connect to the second instance when we run the pipeline on the Jenkins server.
   - Create the Jenkins user that we need for our Jenkinsfiles 
   - Install required packages

```
- Install Jenkins
- Create a Jenkins user password and log into the Jenkins user
  - sudo passwd jenkins set an easy password
  - Sign in to the Jenkins user: `sudo su - jenkins -s /bin/bash`
- Create a public and private key on this instance with ssh-keygen
- Copy the public key contents and paste it into the second instance authorized_keys
  - Jenkins server under jenkins user cd into .ssh
  - Run cat authorized_keys file and copy the key
  - Connect to the second instance and paste the key into the authorized_keys file of the second instance
- IMPORTANT: Test the ssh connection
  - ssh unbuntu@ip_address_second_instance from the Jenkins server as the Jenkins user
- Exit the jenkins user
- Now, in the ubuntu user, install the following: {sudo apt install -y software-properties-common, sudo add-apt-repository -y ppa:deadsnakes/ppa, sudo apt install -y python3.7, sudo apt install -y python3.7-venv}
```

###	3. On the second instance, install the following: Webserver
   - Install required packages
    - I did this process manually.  But I could have created a script and deployed it the same way I did for the first instance in my main.tf file.  I may do this for the next retry of the deployment.

```
- Install the following: {sudo apt install -y software-properties-common, sudo add-apt-repository -y ppa:deadsnakes/ppa, sudo apt install -y python3.7, sudo apt install -y python3.7-venv}
```

### 4. In the Jenkinsfilev1 and Jenkinsfilev2, create a command that will ssh into the second instance and download and run the required script for that step in the Jenkinsfile
- This is the step where we will make changes to the files mentioned above. 
  - Update Jenkinsfilev1 Deploy stage with ssh and curl command
  - Update Jenkinsfilev2 Deploy stage with ssh and curl command
    - This acts as a Jenkins agent since we did not deploy an instance to be an agent
  - Update setup.sh with the correct git clone url and cd command
  - Update setup2.sh with the correct git clone url and cd command

- After modifing the Jenkinsfilev1, Jenkinsfilev2, setup.sh, and setup2.sh commit the changes to git repository
		
### 5. Create a Jenkins multibranch pipeline and run the Jenkinsfilev1
- Jenkins is the main tool used in this deployment for pulling the program from the GitHub repository, and then building and testing the files to be deployed to the second EC2 instance.
- Creating a multibranch pipeline gives the ability to implement different Jenkinsfiles for different branches of the same project.
- A Jenkinsfile is used by Jenkins to list out the steps to be taken in the deployment pipeline.

- Steps in the Jenkinsfilev1 are as follows:
  - Build
    - The environment is built to see if the application can run.
  - Test
    - Unit test is performed to test specific functions in the application
  - Deploy
    - SSH into the second instance and runs the setup.sh script to deploy the retail banking site using gunicorn, flask/python, and SQLite 	


### 6. Check the application on the second instance!!
- Here is the screenshot of the application. Click [HERE](https://github.com/andmulLABS01/Deployment_5RT_1/blob/main/Deployment_5RT1a.PNG)
	
### 7. Now make a change to the HTML and then run the Jenkinsfilev2	
	- Steps in the Jenkinsfilev2 are as follows:
	  - Clean
		- SSH into the second instance and runs the pkill.sh script which stops gunicorn and 
	  - Deploy
		- SSH into the second instance and run the setup2.sh script 
		
#### 7a. Make the change to the home.HTML
	- I changed the font color from black to pink in the Welcome paragraph.
#### 7b. View the application after the change
- Click [HERE](https://github.com/andmulLABS01/Deployment_5RT_1/blob/main/Deployment_5RT1b.PNG)

### 8. How did you decide to run the Jenkinsfilev2? 

- I changed the file path in Jenkins configuration section from Jenkinsfilev1 to Jenkinsfilev2.
- You could also have created a second branch put the Jenkinsfilev2 in that branch and run it from there.

### 9. Should you place both instances in the public subnet? Or should you place them in a private subnet? Explain why?

- Well it depends.  Since both instances need to be accessed via the internet it is best to put them on the public subnet with the correct security group settings to secure them. 
However, we could place parts of the second instance, application and database tiers, into the private subnet to enhance security and limit access.


## System Diagram:

To view the diagram of the system design/deployment pipeline, click [HERE](https://github.com/andmulLABS01/Deployment_5RT_1/blob/main/DPRT1_main.tf)

## Problems/Troubleshooting:

The Jenkins pipeline is completed without deploying the application.

Resolution Steps:
- Revied the compile output and found that there was an error `404 command not found` when getting to the build stage.
- I reviewed the Jenkinsfile and found that I put the wrong URL into the groovy script.
- This caused me to review the other files that I needed to make changes to and found that I skipped the step to modify them.
- I made the modifications and the application deployed. 

## Conclusion:

This run was a bit easier to do as some of the pitfalls from the last test were in front of my mind.  I also redid some of the documentation adding some detailed notes to sections to help me remember steps. For this retry, I did not use an existing VPC as I wanted to improve the security groups and felt that should do irritative steps instead.  In the next retry of this deployment, I will use the existing VPC infrastructure in my main.tf file to create the environment.
