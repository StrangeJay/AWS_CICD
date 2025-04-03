# Build a CI/CD Pipeline with AWS CodeBuild, CodeCommit, and CodeDeploy

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

- [x] Task 1: Creating a key pair
- [x] Task 2: Create an EC2 Instance Profile
- [x] Task 3: Set Up Elasticbeanstalk
- [x] Task 4: Create RDS Database
- [x] Task 5: Edit RDS security group to allow inbound traffic from the e2 instances created by ElasticBeanstalk
- [x] Task 6: Initialise Database
- [x] Task 7: Set up the Code base
- [x] Task 8: Set up s3 bucket for artifact storage
- [x] Task 9: Set up Code Build
- [x] Task 10: Set up code pipeline
- [x] Task 11: Test your pipeline

**Project repo:** [**jprofile**](https://github.com/StrangeJay/jprofile-project/tree/aws-ci)

## Documentation

### Create a Key Pair

- Enter 'key pairs' in the search field and select **Key pairs** from the displayed options.

![](img)

- Click the **Create key pair** button to proceed.

![](img)

- Provide a **Name** for the key pair and click **Create key pair**.

![](img)

---

### Create an EC2 Instance Profile

- Enter **IAM** in the search bar and choose **IAM** from the displayed services.

![](img)

- Navigate to **Roles** by clicking on it.

![](img)

- Click the **Create role** button.

![](img)

- Click the **chevron icon**, then select **EC2** from the dropdown menu.

![](img)

- Click on **Next** to proceed.

![](img)

- In the search field, search for **bean**, and select the four policies shown in the image:

    - **AdministratorAccess-AWSElasticBeanstalk**: This policy grants full administrative access to AWS Elastic Beanstalk and its associated resources. It allows a user or role to create, update, and delete Elastic Beanstalk applications, environments, and configurations. It also provides permissions to manage EC2 instances, load balancers, auto-scaling groups, and other resources Elastic Beanstalk provisions.
    - **AWSElasticBeanstalkCustomPlatformEC2Role**: This policy is designed for EC2 instances that run custom Elastic Beanstalk platforms. It provides the necessary permissions for instances to download platform components, manage logs, and interact with other AWS services needed to run a custom platform.
    - **AWSElasticBeanstalkRoleSNS**: This policy allows Elastic Beanstalk to send notifications via Amazon Simple Notification Service (SNS). It enables Elastic Beanstalk to publish messages related to application and environment events, such as deployment statuses or failures, which can then trigger alerts or automated actions.
    - **AWSElasticBeanstalkWebTier**: This policy is intended for instances in the web tier of an Elastic Beanstalk environment. It grants permissions for web servers to interact with Elastic Beanstalk, manage logs, and perform basic operations required for hosting web applications. It ensures that web-tier instances can function properly within the Elastic Beanstalk-managed infrastructure.

![](img)

- Click **Next** to proceed.

![](img)

- Provide a **Role name** and **Description**.

![](img)

- Click **Create role** to finalize the process.

![](img)

---

### Set up your Elastic Beanstalk Environment

#### Configure environment

- Enter 'beanstalk' in the search field, then select **Elastic Beanstalk** from the search results.

![](img)

- Click **Create application** to begin the process.

![](img)

- Enter your **Application name**, and in the Environment information section, provide your **Environment name** and **Domain**, then **check availability**.

![](img)

> [!NOTE]
It is essential that the domain name is unique, since it will be used to construct the URL.

- Since our app runs on Tomcat, click the **chevron icon** and select **Tomcat** as the platform.

![](img)

- Choose **custom configuration** in the Presets section, then click **Next**.

![](img)

#### Configure service access

- For service roles, select **Create and use new service role**, then click the **chevron down icon** and choose your created key pair.

![](img)

- Click within the empty field to select your created **EC2 instance profile**, and proceed by clicking **Next**.

![](img)

#### Set up networking, database, and tags

- Click the **chevron icon** to open the dropdown menu, then select the **default vpc**.

![](img)

- Ensure public IP address is **Activated** and select all **Availability Zones**.

![](img)

- Click the **Add new tag** button.

![](img)

- Specify a **Key** and **Value** pair, and proceed by clicking **Next**.

![](img)

#### Configure instance traffic and scaling

- Click on the Root volume type field, then choose **General Purpose 3(SSD)** from the options.

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

### RDS Security Group Setup

- Search for ec2 in the search bar and select **EC2** from the services.

![](img)

- Click on **Instances**.

![](img)

- Take note of the 2 servers created by elasticbeanstalk.

![](img)

- Select one of the created servers and click into the **Security group**.

![](img)

- Copy the **Security group ID** of the EC2 instances and then go to **Security Groups** in the Network & Security tab.

![](img)

- Click on the **Security group ID** of the security group created by RDS.

![](img)

- Click on **Edit inbound rules**.

![](img)

- Click the **Add rule** button.

![](img) 

- Search and select **MYSQL/Aurora**.

![](img)

- Paste in the **security group ID** of the instances you copied earlier, and click on **Save rule**.

![](img)

- Return to the **Instances** page.

![](img)

---

### Initialise Database

- Select any of the **instances** and copy the **Public IPv4 address**.

![](img)

- Run the following `ssh -i <"key pair name"> ec2-user@ec2-<Public IP>.compute-1.amazonaws.com`.

![](img)

> [!NOTE]
Ensure you replace "<key pair name>" with the name of your created key pair and <PUBLIC IP> with your public IP address but replace the dots with dashes. For example `ssh -i "cicdbeankey.pem" ec2-user@ec2-3-90-183-88.compute-1.amazonaws.com`.

- Run the following command to search for a MySQL `dnf search mysql` and copy the **mysql client name**.

![](img)

- Run the following command `dnf install mariadb105`.

![](img)

- Run the following command `mysql -h <your rds endpoint> -u <your user> -p accounts` and then type in your password when prompted to log into your database. After confirming that you can access the accounts database, type `exit` to leave the database.

![](img)

- Navigate to the [schema page](https://github.com/StrangeJay/jprofile-project/blob/aws-ci/src/main/resources/db_backup.sql) and click on **Raw**.

![](img)

- Copy the **address** from the address bar.

![](img)

- Return to your terminal and download the sql file by running `wget https://github.com/StrangeJay/jprofile-project/blob/aws-ci/src/main/resources/db_backup.sql`

![](img)

- Run the following command to load the sql file into the accounts database `mysql -h <your rds endpoint> -u <your user> -p accounts < db_backup.sql` and after that run `mysql -h <your rds endpoint> -u <your user> -p accounts` to log into the database.

![](img)

- Run the query `show tables;` to confirm the tables in the image are available on your end.

![](img)

### Set Up Your Code Repository

- Head over to [Bitbucket](https://bitbucket.org/product) and create an account if you don't already have one.

- Click on **Create a workspace**.

![](img)

- **Name** your workspace and click on **Agree and create workspace**.

![](img)

- Select **Create repository**.

![](img)

- Fill in the necessary details and click on **Create repository**.

![](img)

> [!NOTE]
Ensure the repo is empty while creating it, don't add a readme or git ignore file because we'll be migrating our github repository here.

- Return to your terminal and run this command to check if you have ssh keys created already `ls .ssh/`

![](img)

> [!NOTE]
You can either use the existing ssh key or create a new one.

- Run `cat <public key name>` to view your **public key**, and then copy it.

![](img)

> [!NOTE]
Ensure you replace <public key name> with the name of your public key.

- Click on the **gear icon** and select **Personal Bitbucket settings**.

![](img)

- Select **SSH keys** and click on the **Add key** button.

![](img)

- Fill in the necessary **details** Name your SSH key, paste the public key you copied earlier and click on **Add key**.

![](img)

- Return to your terminal and create a config file so when we do a git push or git pull it should use the private key to login to the bitbucket account.

- Create your config file by running the following command `cd ~/.ssh && nano config` and paste the following command in the text editor:
```
# bitbucket.org
Host bitbucket.org
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/<private key name>
```

![](img)

- Test the connection by running `ssh -T git@bitbucket.org` if the connection is working you should see a message similar to that in this image.

![](img)

- Return to the [Code repository](https://github.com/StrangeJay/jprofile-project/tree/main) on github and click on **Code**.

![](img)

- Select SSH and **copy the url**.

![](img)

- Run **`git clone git@github.com:StrangeJay/jprofile-project.git`** to download the code base locally to your device and then cd into the repo with **`cd jprofile-project`**.

![](img)

- Run **`git checkout aws-ci`** to change branches.

![](img)

- Run **`cat .git/config`** to see the origin that's being tracked and run **`git remote rm origin`** to remove github.

![](img)

- Return to Bitbucket, navigate to your repository and **copy the SSH command**, copy everything after git clone, like in the image below.

![](img)

- Return to your terminal and run the command `git remote add origin <copied Bitbucket SSH url>`.

![](img)

- Run `cat .git/config` again and confirm that remote origin has been changed to the **bitbucket url**.

![](img)

- Run **`git push origin --all`** to push all checked out branch (main and aws-ci) upstream and confirm that you see **both branches**.

![](img)

- Return to Bitbucket to confirm that the content was pushed.

![](img)

- Click on the **chevron down icon** close to main and then confirm you can see both **main and aws-ci branches**.

![](img)

---

### Create an S3 Bucket

- Go to your AWS console and search for s3 and select **S3** from the list of services.

![](img)

- Click on **Create bucket**.

![](img)

- Provide a **Bucket name**.

![](img)

- Click on **Create bucket** to create the bucket.

![](img)

> [!NOTE]
Just give your bucket a name and create it, leave every other setting on default.

---

### Set Up Code Build

- Search for cold build and select **CodeBuild** from the list of services.

![](img)

- Click on **Create project**.

![](img)

- Enter a **Project name**.

![](img)

- Click on the **chevron icon** in the source provider section and select **Bitbucket** from the dropdown.

![](img)

- To connect to your Bitbucket account click on **Manage account credentials**.

![](img)

- Click on the **chevron icon** and select **OAuth app** as the Credential type.

![](img)

> [!NOTE]
Bitbucket access token is the best option but a paid feature, so use OAuth as long as you're logged into Bitbucket on the same browser.

- Select **CodeBuild** as the service and then click on **Connect to Bitbucket**.

![](img)

- Click on **Confirm** to confirm the connection.

![](img)

- You should see a message saying your account is successfully connected.

![](img)

- Click on the empty field below Repository and select your **Bitbucket repository** from the dropdown.

![](img)

- Type the name of the branch we're using **`aws-ci`** into the Source version field.

![](img)

- Select **Ubuntu** as your operating system.

![](img)

- Scroll down to Build spec and click on **Switch to editor**.

![](img)

- Download or open up and copy the content of the **buildspec.yml file**, paste it in a text editor and replace the necessary fields.

![](img)

> [!NOTE]
This command consists of three sed (stream editor) operations that modify the application.properties file located at src/main/resources/application.properties. Each command uses the sed -i option to perform an in-place substitution, meaning the file is modified directly without creating a new one.

```
- sed -i 's/jdbc.password=admin123/jdbc.password=nr1mTWY6OvlLBovvmZpD/' src/main/resources/application.properties
- sed -i 's/jdbc.username=admin/jdbc.username=admin/' src/main/resources/application.properties
- sed -i 's/db01:3306/vprodb.c50sgqqusvnr.us-east-1.rds.amazonaws.com:3306/' src/main/resources/application.properties
```

These `sed` commands modify the **`application.properties`** file in **`src/main/resources/`** by updating database credentials and connection details. The first command replaces the database password, changing `jdbc.password=admin123` to `jdbc.password=nr1mTWY6OvlLBovvmZpD`, where `nr1mTWY6OvlLBovvmZpD` is a placeholder and should be replaced with your actual password. The second command attempts to update the database username, but since the replacement value is the same (`admin`), no actual change occurs. The third command updates the database host, replacing `db01:3306` with `vprodb.c50sgqqusvnr.us-east-1.rds.amazonaws.com:3306`, which is an AWS RDS endpoint. All placeholders, including the password and database host, should be replaced with your actual connection details before running these commands. Study this image to see how it should look.

![](img)

- Copy the entire buildspec.yml content and return to your AWS console and then paste it into the Build commands field.

![](img)

- In the Artifact section, select **Amazon S3**.

![](img)

- Click on the Bucket name field and select your created **bucket**.

![](img)

- Set a **Group name** and **Stream name prefix** for cloudwatch logs and click on **Create build project**. 

![](img)

- Now your project has been created, click on **Start build**.

![](img)

- Confirm the success of the build success.

![](img)

---

### Set Up Code Pipeline

- Search for code pipeline and select **CodePipeline** from the list of Services.

![](img)

- Click on **Create pipeline**.

![](img)

- Choose **Build custom pipeline** and click on **Next**. 

![](img)

- Enter a **Pipeline name** and click on **Next**.

![](img)

- Click on the Source provider field and select **Bitbucket**.

![](img)

- Click on the **Connect to Bitbucket** button.

![](img)

- Give the **Connection name** and click on **Connect to Bitbucket**.

![](img)

- Click on **Install a new app**.

![](img)

- Click on the **Grant access** button.

![](img)

- Click on **Connect** to complete the connection.

![](img)

- Click on the repository field and select your **Bitbucket repo**.

![](img)

- Click on the Default branch field and select **aws-ci**.

![](img)

- Click **Next**.

![](img)

- Select **Other build providers**, click on the field below and select **AWS codebuild** from the options.

![](img)

- Click on the Project name field and select your **created build project**.

![](img)

- Click on **Next**.

![](img)

- For Test provider, select **AWS CodeBuild**.

![](img)

> [!NOTE]
You can skip this step, it's optional.

- Click on the Project name field and select your **build project**.

![](img)

- Click on **Next**.

![](img)

> [!NOTE]
There are some inconsistencies between the already built artifact and the Source artifact, so change your input artifact to Source artifact.

![](img)

![](img)

- In the deployment stage, select **AWS Elastic Beanstalk** as the Deploy provider.

![](img)

- Select your **Application name**.

![](img)

- Select your **Environment name**.

![](img)

- Click **Next**.

![](img)

- Review everything and click on **Create pipeline**.

![](img)

- Confirm the successful creation of your pipeline and wait for the process to complete.

![](img)

- If everything was done correctly every stage of the pipeline should succeed. 

![](img)

---

### Test Entire Set Up

- Navigate to your Elastic Beanstalk page and click on the **Domain name** to visit the site.

![](img)

- If your set up was done right, you should see a page like this. Login with **Admin_vp** as Username and Password.

![](img)

- Your App has been successfully deployed.

![](img)

- To test if the pipeline gets triggered as it should, connect to the code repo on your terminal.

![](img)

- Make a minor change and push the change.

![](img)

- Notice the project start building all over again. 

![](img)

---

And that's the end of the project, you have successfully created an AWS pipeline.
