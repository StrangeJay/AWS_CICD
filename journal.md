# Setting Up a CI/CD Pipeline with AWS CodeBuild, CodeCommit, and CodeDeploy

## Introduction

Continuous Integration and Continuous Deployment (CI/CD) pipelines are essential for modern software development, enabling automated code testing, building, and deployment. AWS offers a suite of services—AWS CodeBuild, AWS CodeCommit, and AWS CodeDeploy—that streamline the CI/CD process, reducing manual effort and improving deployment efficiency.

In this guide, we will walk through the process of setting up a fully automated CI/CD pipeline to deploy an application to AWS Elastic Beanstalk. The pipeline will be configured to automatically build and deploy application updates whenever changes are pushed to a Bitbucket repository.

Initially, our codebase is hosted on GitHub, but we will migrate it to Bitbucket to align with our pipeline requirements. Bitbucket provides a seamless integration with AWS, allowing us to trigger builds and deployments efficiently. We will use AWS CodeCommit as an intermediate step to facilitate integration between Bitbucket and AWS services.

Additionally, we will create and initialize our own database, ensuring the application has a properly configured data layer before deployment. This step will involve:

- Setting up a managed database service (Amazon RDS)

- Running database migrations and seeding initial data

## What You’ll Learn in This Guide:

- **Migrating Code from GitHub to Bitbucket** – Moving an existing repository while preserving version history.

- **Setting Up AWS CodeCommit** – Acting as a bridge between Bitbucket and AWS services.

- **Configuring AWS CodeBuild** – Automating the build process to compile, test, and package the application.

- **Setting Up AWS CodeDeploy** – Automating deployments to AWS Elastic Beanstalk.

- **Creating a Database and Initializing It** – Provisioning a database and running migrations/seeding.

- **Creating an AWS CodePipeline** – Automating the entire CI/CD workflow.

- **Testing the CI/CD Pipeline** – Ensuring seamless integration and automatic deployments.

By the end of this guide, you will have a fully functional CI/CD pipeline that automates application deployment to AWS Elastic Beanstalk, along with a properly configured database, ensuring a complete and efficient deployment setup.

---

## CI/CD Project

|S/N | Project Tasks                              |
|----|--------------------------------------------|
| 1  |Create a key pair                           |
| 2  |Set up elastic beanstalk                    |
| 3  |Create an RDS database                      |
| 4  |Initialise database                         |
| 5  |Setup Bitbucket                             |
| 6  |Create an s3 bucket to save your artifact   |
| 7  |Setup Code Build                            |
| 8  |Setup Code Pipeline                         |
| 9  |Test Setup                                  |

## Checklist

- [x] Task 1: Creating a keypair
- [x] Task 2: Create an EC2 Instance Profile
- [x] Task 3: Set Up Elasticbeanstalk
- [x] Task 4: Create RDS Database
- [] Task 5: Edit RDS security group to allow inbound traffic from the e2 instances created by ElasticBeanstalk
- [] Task 6: Initialise Database
- [] Task 7: Set up the Code base
- [] Task 8: Set up s3 bucket for artifact storage
- [] Task 9: Set up Code Build
- [] Task 10: Set up code pipeline
- [] Task 11: Test your pipeline

## Documentation

### Create a Key Pair

- Search for key pair in the search field and click on **Key pairs**.

![](img)

- Click on the **Create key pair** button.

![](img)

- Set a **Name** for your key pair and click on **Create key pair**.

![](img)

---

### Create an EC2 Instance Profile

- **Search for iam** and select **IAM** from the list of services that drop down.

![](img)

- Click on **Roles**.

![](img)

- Click on **Create role**.

![](img)

- Click on the **chevron icon** and select **EC2** from the dropdown menu.

![](img)

- Click on **Next**.

![](img)

- **Search for bean**, and select the 4 policies in the image:

**AdministratorAccess-AWSElasticBeanstalk**: 
**AWSElasticBeanstalkCustomPlatformEC2Role**: 
**AWSElasticBeanstalkRoleSNS**: 
**AWSElasticBeanstalkWebTier**:

![](img)

- Click on **Next**.

![](img)

- Give a **Role name** and **Description**.

![](img)

- Click on **Create role**.

![](img)

---

### Set up your Elastic Beanstalk Environment

#### Configure environment

- Search for beanstalk in the search field and select **Elastic Beanstalk** from the results.

![](img)

- Click on **Create application**.

![](img)

- Set your *Application name**, in Environment information set your **Environment name**, **Domain** and **check availability**

![](img)

> [!NOTE]
Make sure the domain name is unique because that'll form the URL.

- Our app runs on Tomcat so click on the **chevron icon** and  choose **Tomcat** as the platform.

![](img)

- In the Presets section, select **custom configuration** and click on **Next**.

![](img)

#### Configure service access

- For service roles select **Create and use new service role**, click on the **chevron down** icon and select your created key pair.

![](img)

- Click on the empty field, select your created **EC2 instance profile** and click **Next**.

![](img)

#### Set up networking, database, and tags

- Click on the **chevron icon** and select the **default vpc**.

![](img)

- Ensure public IP address is **Activated** and select all **Availability Zone**.

![](img)

- Click on **Add new tag**.

![](img)

- Give a **Key** and **Value** pair, and then click on **Next**.

![](img)

#### Configure instance traffic and scaling

- Click on the Root volume type field and select **General Purpose 3(SSD)**

![](img)

- In the Auto scaling group section, select **Load balanced**.

![](img)

- Set the desired minimum and maximum number of **Instances** you want to be provisioned.

![](img)

- Scroll down to Instances types and change it to **t2.micro** to stay within the free tier limit.

![](img)

- In Processes, select the **radio button** and click on **Actions**.

![](img)

- Select **Edit**.

![](img)

- Click on the **Chevron down icon** beside Sessions, ensure Session stickiness is **Enabled**, and click on **Save**.

![](img)

- Click on **Next**.

![](img)

#### Configure updates, monitoring, and logging

- In Application deployments section, click on the Deployment policy field and select **Rolling**.

![](img)

- Choose your **Deployment batch size**.

![](img)

> [!NOTE]
We're using 50% here, but in production you'll have multiple instances and shouldn't select more than 25%, preferably 10% for 1 instance at a time.

- Click on ** Next**.

![](img)

- Review your configurations and click **Submit**.

![](img)

- Your Elastic Beanstalk environment is being created, it might take a while, so while waiting for it, move on to the next step.

![](img)

---

### Create an RDS Database

- **Search for RDS** and select **Aurora and RDS** from the listed services.

![](img)

- Click on **Create database**.

![](img)

- Select **MySQL**.

![](img)

- Select **Free tier** to keep your database resources within the free tier range.

![](img)

- Type in a unique name for your **DB instance identifier** and select **Auto generate password**.

![](img)

- In the VPC section, select **Create new** and type in your **New VPC security group name**.

![](img)

- Click on the **chevron icon** beside Additional configuration.

![](img)

- Type in your **Initial database name**.

![](img)

> [!NOTE]
Ensure you use accounts because the code base has a database named accounts that we will be making use of. But if you're working with your own code base then you can name it whatever name you gave the database you want to load.

![](img)

- Click on **Create database**.

![](img)

- **Close** the pop up message.

![](img)

- Click on **View credential details** to get the credentials for your database.

![](img)

- Copy the details and save them in a note.

![](img)

---












> [!NOTE]

