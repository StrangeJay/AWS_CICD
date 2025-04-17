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

![](img/keypair1.png)

- Click the **Create key pair** button to proceed.

![](img/keypair2.png)

- Provide a **Name** for the key pair and click **Create key pair**.

![](img/keypair4.png)

---

### Create an EC2 Instance Profile

- Enter **IAM** in the search bar and choose **IAM** from the displayed services.

![](img/iam1.png)

- Navigate to **Roles** by clicking on it.

![](img/iam2.png)

- Click the **Create role** button.

![](img/iam3.png)

- Click the **chevron icon**, then select **EC2** from the dropdown menu.

![](img/iam4.png)

- Click on **Next** to proceed.

![](img/iam5.png)

- In the search field, search for **bean**, and select the four policies shown in the image:

    - **AdministratorAccess-AWSElasticBeanstalk**: This policy grants full administrative access to AWS Elastic Beanstalk and its associated resources. It allows a user or role to create, update, and delete Elastic Beanstalk applications, environments, and configurations. It also provides permissions to manage EC2 instances, load balancers, auto-scaling groups, and other resources Elastic Beanstalk provisions.
    - **AWSElasticBeanstalkCustomPlatformEC2Role**: This policy is designed for EC2 instances that run custom Elastic Beanstalk platforms. It provides the necessary permissions for instances to download platform components, manage logs, and interact with other AWS services needed to run a custom platform.
    - **AWSElasticBeanstalkRoleSNS**: This policy allows Elastic Beanstalk to send notifications via Amazon Simple Notification Service (SNS). It enables Elastic Beanstalk to publish messages related to application and environment events, such as deployment statuses or failures, which can then trigger alerts or automated actions.
    - **AWSElasticBeanstalkWebTier**: This policy is intended for instances in the web tier of an Elastic Beanstalk environment. It grants permissions for web servers to interact with Elastic Beanstalk, manage logs, and perform basic operations required for hosting web applications. It ensures that web-tier instances can function properly within the Elastic Beanstalk-managed infrastructure.

![](img/iam6.png)

- Click **Next** to proceed.

![](img/iam7.png)

- Provide a **Role name** and **Description**.

![](img/iam8.png)

- Click **Create role** to finalize the process.

![](img/iam9.png)

---

### Set up your Elastic Beanstalk Environment

#### Configure environment

- Enter 'beanstalk' in the search field, then select **Elastic Beanstalk** from the search results.

![](img/ebean1.png)

- Click **Create application** to begin the process.

![](img/ebean2.png)

- Enter your **Application name**, and in the Environment information section, provide your **Environment name** and **Domain**, then **check availability**.

![](img/ebean3.png)

> [!NOTE]
It is essential that the domain name is unique, since it will be used to construct the URL.

- Since our app runs on Tomcat, click the **chevron icon** and select **Tomcat** as the platform.

![](img/ebean4.png)

- Choose **custom configuration** in the Presets section, then click **Next**.

![](img/ebean5.png)

#### Configure service access

- For service roles, select **Create and use new service role**, then click the **chevron down icon** and choose your created key pair.

![](img/ebean6.png)

- Click within the empty field to select your created **EC2 instance profile**, and proceed by clicking **Next**.

![](img/ebean7.png)

#### Set up networking, database, and tags

- Click the **chevron icon** to open the dropdown menu, then select the **default vpc**.

![](img/ebean8.png)

- Ensure public IP address is **Activated** and select all **Availability Zones**.

![](img/ebean9.png)

- Click the **Add new tag** button.

![](img/ebean10.png)

- Specify a **Key** and **Value** pair, and proceed by clicking **Next**.

![](img/ebean11.png)

#### Configure instance traffic and scaling

- Click on the Root volume type field, then choose **General Purpose 3(SSD)** from the options.

![](img/ebean12.png)

- Within the Auto Scaling group section, choose the **Load balanced** option.

![](img/ebean13.png)

- Specify the desired minimum and maximum number of **Instances** to be provisioned.

![](img/ebean14.png)

- Scroll down to Instances types and change it to **t2.micro** to remain within the free tier limits.

![](img/ebean15.png)

- Within the Processes section, click the **radio button** to select it, then click **Actions**.

![](img/ebean16.png)

- Choose **Edit**.

![](img/ebean17.png)

- Click the **Chevron down icon** next to Sessions, verify that Session stickiness is **Enabled**, and then click **Save**.

![](img/ebean18.png)

- Click the **Next** button.

![](img/ebean19.png)

#### Configure updates, monitoring, and logging

- Within the Application deployments section, click on the Deployment policy field, then choose **Rolling** from the options.

![](img/ebean20.png)

- Specify the **Deployment batch size** you want to use.

![](img/ebean21.png)

> [!NOTE]
For this example, we are using a Deployment batch size of 50%. However, in a production environment with multiple instances, it's recommended to select no more than 25%, ideally around 10% to deploy to one instance at a time.

- Click the **Next** button.

![](img/ebean22.png)

- Take a moment to review your settings, and then click **Submit**.

![](img/ebean23.png)

- Your Elastic Beanstalk environment is currently being created. This process may take some time, so while you wait, proceed to the next step.

![](img/ebean24.png)

---

### Create an RDS Database

- Enter **RDS** in the search bar and choose **Aurora and RDS** from the displayed services.

![](img/rds1.png)

- Click **Create database** to begin setting up a new database.

![](img/rds2.png)

- Select **MySQL**.

![](img/rds3.png)

- Choose the **Free tier** option to keep your database usage within the free tier.

![](img/rds4.png)

- Provide a unique **DB instance identifier**, then select the **Auto generate password** option.

![](img/rds5.png)

- In the VPC section, select **Create new** and type in your **New VPC security group name**.

![](img/rds6.png)

- Click on the **chevron icon** next to Additional configuration.

![](img/rds7.png)

- Enter your desired **Initial database name**.

![](img/rds8.png)

> [!NOTE]
It's important for this exercise that you name your initial database 'accounts' to match the configuration in our codebase.

- Click on **Create database**.

![](img/rds9.png)

- **Close** the pop up message.

![](img/rds10.png)

- Click **View credential details** to get the credentials for your database.

![](img/rds11.png)

- Copy the displayed details and save them in a secure note.

![](img/rds12.png)

---

### RDS Security Group Setup

- Search for ec2 in the search bar and then select **EC2** from the services.

![](img/ec2-1.png)

- Click on **Instances**.

![](img/ec2-2.png)

- Make a note of the two servers provisioned by Elastic Beanstalk.

![](img/ec2-3.png)

- Choose one of the servers created and then click the link to its **Security group**.

![](img/ec2-4.png)

- Copy the **Security group ID** of the EC2 instances and then go to **Security Groups** under the Network & Security tab.

![](img/ec2-5.png)

- Click the **Security group ID** of the security group created by RDS.

![](img/ec2-6.png)

- Click **Edit inbound rules**.

![](img/ec2-7.png)

- Click the **Add rule** button.

![](img/ec2-8.png) 

- Search and select **MYSQL/Aurora**.

![](img/ec2-9.png)

- Paste in the **security group ID** of the instances you copied earlier, and click **Save rule**.

![](img/ec2-10.png)

- Return to the **Instances** page.

![](img/ec2-11.png)

---

### Initialise Database

- Select any of the **instances** and copy its **Public IPv4 address**.

![](img/init-db.png)

- Execute the following command in your terminal **`ssh -i <"key pair name"> ec2-user@ec2-<Public IP>.compute-1.amazonaws.com`**.

![](img/init-db2.png)

> [!NOTE]
Make sure to replace <key pair name> with the exact name of your key pair file and <PUBLIC IP> with your Public IP address. When replacing the Public IP, substitute the dots (.) with dashes (-). For example: `ssh -i "cicdbeankey.pem" ec2-user@ec2-3-90-183-88.compute-1.amazonaws.com`.

- Run the command `dnf search mysql` to search for MySQL packages, and then copy the name of the **mysql client name** package from the results.

![](img/init-db3.png)

- Run the following command **`dnf install mariadb105`**.

![](img/init-db4.png)

- Execute this command in your terminal: **`mysql -h <your rds endpoint> -u <your user> -p accounts`**. Making sure to substitute your RDS endpoint for `<your rds endpoint>` and your username for `<your user>`. When prompted, enter your password to log in to the 'accounts' database, and once you've verified that you can access it, type `exit` to disconnect.

![](img/init-db5.png)

- Go to the [schema page](https://github.com/StrangeJay/jprofile-project/blob/aws-ci/src/main/resources/db_backup.sql) and click **Raw**.

![](img/init-db6.png)

- Copy the **URL** from your browser's address bar.

![](img/init-db7.png)

- Return to your terminal window and execute this command to download the 'db_backup.sql' file: **`wget https://github.com/StrangeJay/jprofile-project/blob/aws-ci/src/main/resources/db_backup.sql`**.

![](img/init-db8.png)

- Execute the first command below in your terminal to import the 'db_backup.sql' file into your 'accounts' database. Make sure to replace `<your rds endpoint>` and `<your user>` with your RDS endpoint and username: **`mysql -h <your rds endpoint> -u <your user> -p accounts < db_backup.sql`**. Then, run the second command to log in: **`mysql -h <your rds endpoint> -u <your user> -p accounts`**.

![](img/init-db9.png)

- Run the SQL command **`show tables;`** in your MySQL prompt. This will list the tables in your current database. Compare this list with the tables shown in the image to confirm they match.

![](img/init-db10.png)

### Set Up Your Code Repository

- Go to [Bitbucket](https://bitbucket.org/product) and create a Bitbucket account if you don't have one yet.

- Click on **Create a workspace**.

![](img/bb1.png)

- Provide a **Name** for your workspace in the designated field, and then click the **Agree and create workspace** button.

![](img/bb2.png)

- Select **Create repository**.

![](img/bb3.png)

- Fill in the necessary details and click on **Create repository**.

![](img/bb4.png)

> [!NOTE]
When creating the repository, ensure it's empty. Do not add a README file or a .gitignore file, as we will be migrating our existing GitHub repository here.

- To check for existing SSH keys on your system, go to your terminal and run the following command: **`ls .ssh/`**.

![](img/bb5.png)

> [!NOTE]
You can either use the existing ssh key or create a new one.

- Run the command **`cat <public key name>`** to display your **public key**, and then copy the displayed key.

![](img/bb6.png)

> [!NOTE]
Make sure to replace `<public key name>` in the command with the actual filename of your public key.

- Click the **gear icon** and then select **Personal Bitbucket settings** from the menu.

![](img/bb7.png)

- Select **SSH keys** from the left-hand menu, and then click the **Add key** button.

![](img/bb8.png)

- In the form provided, fill in the necessary **details**. Give your SSH key a descriptive name, paste the public key you copied earlier into the designated field, and then click **Add key**.

![](img/bb9.png)

- Return to your terminal and create a configuration file. This setup will allow Git to use your private SSH key for authentication when you perform `git push` or `git pull` operations with your Bitbucket repository.

- In your terminal, run the following commands to create and edit the SSH config file: **`cd ~/.ssh && nano config`**. Once the nano text editor opens, paste the following configuration, making sure to replace `~/.ssh/<private key name>` with the actual path to your private key file:
```
# bitbucket.org
Host bitbucket.org
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/<private key name>
```

![](img/bb10.png)

- Run the command **`ssh -T git@bitbucket.org`** in your terminal to test your SSH connection to Bitbucket. If the connection is successful, you should see a message similar to the one shown in the image.

![](img/bb11.png)

- Go back to the [Code repository](https://github.com/StrangeJay/jprofile-project/tree/main) on github and click the **Code** button.

![](img/bb12.png)

- Select the SSH option and then **copy** the displayed URL.

![](img/bb13.png)

- Run the following command to download the codebase to your local machine: **`git clone git@github.com:StrangeJay/jprofile-project.git`**. Once the download is complete, navigate into the repository directory by running: **`cd jprofile-project`**.

![](img/bb14.png)

- Run the command **`git checkout aws-ci`** to switch to the 'aws-ci' branch.

![](img/bb15.png)

- Run the command **`cat .git/config`** to view the remote repository that your local repository is currently tracking. Then, run **`git remote rm origin`** to remove the connection to the GitHub repository.

![](img/bb16.png)

- Go back to your repository on Bitbucket. Locate the **SSH clone command** and copy the portion of the URL that comes after git clone, as shown in the image below.

![](img/bb17.png)

- Go back to your terminal and run the following command, replacing `<copied Bitbucket SSH url>` with the SSH URL you just copied from Bitbucket: **`git remote add origin <copied Bitbucket SSH url>`**.

![](img/bb18.png)

- Run the command `cat .git/config`  again in your terminal. Verify that the [remote "origin"] section now shows the **bitbucket url** you just added.

![](img/bb19.png)

- Run the command **`git push origin --all`** to push all local branches (including 'main' and 'aws-ci') to your Bitbucket repository. After running the command, confirm in your Bitbucket repository that **both branches** ('main' and 'aws-ci') have been successfully pushed.

![](img/bb20.png)

- Go back to your Bitbucket repository to verify that the code and branches have been successfully pushed.

![](img/bb21.png)

- On Bitbucket, click the **chevron down icon** located near the 'main' branch name. A dropdown menu should appear, where you should confirm that you see both the **main and aws-ci branches** listed.

![](img/bb22.png)

---

### Create an S3 Bucket

- Go to your AWS console, search for 'S3' in the search bar, and select **S3** from the list of services.

![](img/s3-1.png)

- Click on **Create bucket**.

![](img/s3-2.png)

- Provide a **Bucket name**.

![](img/s3-3.png)

- Click on **Create bucket** to create the bucket.

![](img/s3-4.png)

> [!NOTE]
Just give your bucket a name and create it, leave every other setting on default.

---

### Set Up Code Build

- In your AWS console, use the search bar to find 'CodeBuild', and then choose **CodeBuild** from the displayed services.

![](img/cb1.png)

- Click on **Create project**.

![](img/cb2.png)

- Enter a **Project name** in the designated field.

![](img/cb3.png)

- In the Source provider section, click the **chevron icon** and select **Bitbucket** from the dropdown menu that appears.

![](img/cb4.png)

- Click on **Manage account credentials** to connect your AWS CodeBuild project to your Bitbucket account.

![](img/cb5.png)

- Click the **chevron icon** and then select **OAuth app** as the Credential type from the dropdown menu.

![](img/cb6.png)

> [!NOTE]
While using a Bitbucket access token is the recommended and most secure method, it's a paid feature. Therefore, as long as you are logged into Bitbucket in the same web browser, using OAuth app is a viable alternative.

- Choose **CodeBuild** and then click the **Connect to Bitbucket** button.

![](img/cb7.png)

- Click the **Confirm** button to finalize the connection.

![](img/cb8.png)

- You should see a confirmation message indicating that your Bitbucket account has been successfully connected.

![](img/cb9.png)

- Click in the empty field below 'Repository'. A dropdown menu will appear; select your **Bitbucket repository** from the list.

![](img/cb10.png)

- In the 'Source version' field, type the branch name **`aws-ci`**.

![](img/cb11.png)

- Choose **Ubuntu** as the operating system for your build environment.

![](img/cb12.png)

- Scroll down the page until you find the 'Buildspec' section, and then click the **Switch to editor** button.

![](img/cb13.png)

- Either download the [**buildspec.yml**](https://github.com/StrangeJay/AWS_CICD/blob/master/buildspec.yml) file to your computer or open it directly in your browser. Once you have access to its contents, copy everything and paste it into the text editor on the AWS CodeBuild page. Next, find and replace the placeholders or fields that need your specific configuration.

![](img/cb14.png)

> [!NOTE]
This command consists of three sed (stream editor) operations that modify the application.properties file located at `src/main/resources/application.properties`. Each command uses the `sed -i` option to perform an in-place substitution, meaning the file is modified directly without creating a new one.

```
- sed -i 's/jdbc.password=admin123/jdbc.password=nr1mTWY6OvlLBovvmZpD/' src/main/resources/application.properties
- sed -i 's/jdbc.username=admin/jdbc.username=admin/' src/main/resources/application.properties
- sed -i 's/db01:3306/vprodb.c50sgqqusvnr.us-east-1.rds.amazonaws.com:3306/' src/main/resources/application.properties
```

These `sed` commands modify the **`application.properties`** file in **`src/main/resources/`** by updating database credentials and connection details. The first command replaces the database password, changing `jdbc.password=admin123` to `jdbc.password=nr1mTWY6OvlLBovvmZpD`, where `nr1mTWY6OvlLBovvmZpD` is a placeholder and should be replaced with your actual password. The second command attempts to update the database username, but since the replacement value is the same (`admin`), no actual change occurs. The third command updates the database host, replacing `db01:3306` with `vprodb.c50sgqqusvnr.us-east-1.rds.amazonaws.com:3306`, which is an AWS RDS endpoint. All placeholders, including the password and database host, should be replaced with your actual connection details before running these commands. Study this image to see how it should look.

![](img/cb15.png)

- Copy the complete content of the buildspec.yml file. Then, go back to your AWS console and paste this entire content into the 'Build commands' field.

![](img/cb16.png)

- Scroll down to the 'Artifacts' section. Under 'Type', select **Amazon S3** from the available options.

![](img/cb17.png)

- Click in the 'Bucket name' field. A dropdown menu will appear; select the **bucket** you created earlier from that list.

![](img/cb18.png)

- "Enter a **Group name** and a **Stream name prefix** for your CloudWatch logs. Once you've done this, click the **Create build project** button.

![](img/cb19.png)

- Now that your project is created, click the **Start build** button to begin the build process.

![](img/cb20.png)

- Check the build status to confirm that it shows 'Succeeded'.

![](img/cb21.png)

---

### Set Up Code Pipeline

- Search for code pipeline and select **CodePipeline** from the list of Services.

![](img/cp1.png)

- Click on **Create pipeline**.

![](img/cp2.png)

- Choose **Build custom pipeline** and click on **Next**. 

![](img/cp3.png)

- Enter a **Pipeline name** and click on **Next**.

![](img/cp4.png)

- Click on the Source provider field and select **Bitbucket**.

![](img/cp5.png)

- Click on the **Connect to Bitbucket** button.

![](img/cp6.png)

- Give the **Connection name** and click on **Connect to Bitbucket**.

![](img/cp7.png)

- Click on **Install a new app**.

![](img/cp8.png)

- Click on the **Grant access** button.

![](img/cp9.png)

- Click on **Connect** to complete the connection.

![](img/cp10.png)

- Click on the repository field and select your **Bitbucket repo**.

![](img/cp11.png)

- Click on the Default branch field and select **aws-ci**.

![](img/cp12.png)

- Click **Next**.

![](img/cp13.png)

- Select **Other build providers**, click on the field below and select **AWS codebuild** from the options.

![](img/cp14.png)

- Click on the Project name field and select your **created build project**.

![](img/cp15.png)

- Click on **Next**.

![](img/cp16.png)

- For Test provider, select **AWS CodeBuild**.

![](img/cp17.png)

> [!NOTE]
You can skip this step, it's optional.

- Click on the Project name field and select your **build project**.

![](img/cp18.png)

- Click on **Next**.

![](img/cp19.png)

> [!NOTE]
There are some inconsistencies between the already built artifact and the Source artifact, so change your input artifact to Source artifact.

![](img/cp19-2.png)

![](img/cp19-3.png)

- In the deployment stage, select **AWS Elastic Beanstalk** as the Deploy provider.

![](img/cp20.png)

- Select your **Application name**.

![](img/cp21.png)

- Select your **Environment name**.

![](img/cp22.png)

- Click **Next**.

![](img/cp23.png)

- Review everything and click on **Create pipeline**.

![](img/cp24.png)

- Confirm the successful creation of your pipeline and wait for the process to complete.

![](img/cp25.png)

- If everything was done correctly every stage of the pipeline should succeed. 

![](img/cp26.png)

---

### Test Entire Set Up

- Navigate to your Elastic Beanstalk page and click on the **Domain name** to visit the site.

![](img/test1.png)

- If your set up was done right, you should see a page like this. Login with **Admin_vp** as Username and Password.

![](img/test2.png)

- Your App has been successfully deployed.

![](img/test3.png)

- To test if the pipeline gets triggered as it should, connect to the code repo on your terminal.

![](img/test4.png)

- Make a minor change and push the change.

![](img/test5.png)

- Notice the project start building all over again. 

![](img/test6.png)

---

And that's the end of the project, you have successfully created an AWS pipeline.
