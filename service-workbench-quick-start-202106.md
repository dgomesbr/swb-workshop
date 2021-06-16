# Service Workbench Quick Start Guide
June, 2021

## 1. Prerequisites

Before getting started with the installation of Service Workbench on AWS, you should ensure that you have the requisite expertise, and have completed the pre-installation steps outlined below.
1. Practical knowledge of core AWS services (EC2, IAM) and Linux command line.
1. Have administrator access to an AWS account into which you can deploy Service Workbench on AWS.


### 1.1 Enable AWS Cost Explorer

1. Enable AWS Cost Explorer by visiting the AWS Cost Management service for the account that you will be deploying Service Workbench on AWS into. 
1. (After 24h) Activate the following cost allocation tags in the AWS Billing and Cost Management Dashboard to enable billing information within Service Workbench on AWS.  These tags appear after enabling the AWS Cost Explorer.

    • `Proj`, `Env`, `CreatedBy`

### 1.2	Create a Development Environment using AWS Cloud9

To deploy and configure Service Workbench on AWS, you will need a development environment. To avoid any confusion with your local environment, we will use AWS Cloud9, which is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. 

1. Within your AWS Console, go to the [AWS Cloud9 service](https://console.aws.amazon.com/cloud9/home/product)
1. Choose **Create Environment**
1. Fill the Name field with: `c9-swb-dev`
1. Add a meaningful description
1. Click on **Next Step**
1. Leave the Environment Type as _Create a new EC2 instance for environment (direct access)_
1. Under **Instance type**, Choose **Other instance type** - choose **m5.xlarge** (you can use a t3.medium with customers)
1. In Platform, choose Amazon Linux 2
1. Scroll down to tags and include:
    1. Key: **Owner**, Value: Your loginname, alias, or another unique identifier
    2. Key: **Env**, Value: `dev`
    3. Key: **Proj**, Value: `swb-workshop`
1. Click on **Next Step**
1. Review the configuration, and click on **Create Environment**

1.3	Installing required tools and utilities
The last of the requirements is to install the libraries and tools required to complete the installation of the Service Workbench on AWS. Additionally, by default, the AWS Cloud9 instance bundles a 10Gb EBS volume. This won’t be enough for our development environment. The script will also resize our AWS Cloud9 instance disk:

1.	Clone the Service Workbench on AWS using Cloud9 boostrap repository
2.	Read and execute the contents of the init script 
(Optional) Access platform reference documentation
The documentation is available via Docusaurus; In order for you to be able to access it, please modify the AWS Cloud9 instance security group to include your IP address. If you need additional guidance, follow the documentation.

After modifying the security group, you can build and serve the documentation:

 
2	Installing and configuring the Service Workbench on AWS
This section describes how to deploy Service Workbench on AWS using the AWS Cloud9 instance that we created in the pre-requisites section. Alternatively, it is possible to deploy from a local computer using local AWS CLI credentials. Keep in mind that tools and libraries installed in the previous section might be incompatible with your local system.
2.1	Creating a new Service Workbench Configuration

In this section, you will install Service Workbench components into your AWS account. Once the last step is underway, you can proceed to the next section (Install AMIs for EC2-based workspaces) to run both processes simultaneously.

1.	In the terminal, export the Stage Name that is going to be used in the deployment process, for this guide we will be using demo as the Stage Name.
Note: The stage name is included in the name of the Amazon S3 storage bucket, so must be Amazon S3-compatible (lower-case characters, numbers, periods, and dashes)
2.	Create a copy of the example configuration that comes bundled with the repository
3.	Open the “demo.yml” configuration file on the on AWS Cloud9, uncomment, and set values for:
3.1	solutionName: The solutionName is used in Amazon S3 bucket names so must be Amazon S3-compatible (lower-case characters, numbers, periods, and dashes)
3.2	 awsRegion: The region you will be using for the deployment. Make sure to use the same region when you are using the AWS Console.
4.	Back to the terminal, run the following script to complete the installation.
Note: This takes up to 15 minutes and can be ran in parallel with the AMI Installation step.
5.	Once the above step has completed, copy the root account password and website URL for later use.
Note: To retrieve this information again, run this command:
6.	Using the URL and root password from above, visit the Service Workbench on AWS website and log in using user “root”.


2.2	Install AMIs for EC2-based workspaces

In order to use EC2-based workspaces, you must ﬁrst install Amazon EC2 AMIs for these workspaces. This process may be run concurrently with the previous section (while environment-deploy.sh is running). In order to run both simultaneously, use the menu Windows > New Terminal

1.	In the terminal, run the following command to start the Amazon EC2 AMI generation
 
2.	Once the process has been completed, you can verify that the Amazon EC2 AMI were created by running:  
Note: This alias is defined in your ~/.bashrc file as a shortcut to query the Amazon EC2 API. You should see AMIs for EC2-LINUX, EC2-RSTUDIO, EC2-WINDOWS, and EMR.

 
3	Conﬁguring Accounts
In this section, you will set up your Service Workbench instance with accounts, workspaces, and other features, that can be used for evaluating the main Service Workbench features. You can perform the steps in this section after logging into your Service Workbench web interface as root for the ﬁrst time (the veriﬁcation step of the Install the Service Workbench on AWS platform section above).

3.1	Hosting account setup

Hosting accounts are the accounts in which research compute resources are deployed, and which are responsible for the billing of those resources. In this deployment, the hosting account will be the same as the deployment account.
1.	Access the host account directory by:
 
2.	In the terminal, run the following command, it will create an AWS CloudFormation stack named “aws-hosting-account-${ACCOUNT_ID}-stack”, where ACCOUNT_ID, is the 12-digit account number that  was passed to the script.
 
Note1: This step is dependent of $STAGE_NAME environment variable to be set. This was done on Step 2.1.1
Note2: If you want to use the same account as the AWS Cloud9 is running, retrieve the account id running:
 
3.	The parameters passed to the stack are described below, but can also be seen on the “hosting-account-cfn-args-${STAGE_NAME}.json” inside the cloud9 directory.

Parameter Name	Value
Namespace	Stage name
CentralAccountId	For the quickstart, this will be the current AWS Cloud9 AccountId
ExternalId	Workbench
VpcCidr	default (10.0.0.0/16)
VpcPublicSubnet1Cidr	default (10.0.0.0/19)
ApiHandlerArn	ApiHandlerRoleArn created in the first step
LaunchConstraintPolicyPreﬁx	default (*)
LaunchConstraintRolePrefix	default (*)
WorkﬂowRoleArn	WorkﬂowLoopRunnerRoleArn created in the first step
 


4.	Deploy the stack. The Outputs of the stack will contain values similar to:

Key	Value
CrossAccountEnvMgmtRoleArn	arn:aws:iam::0000:role/stagename-stage-xacc-env-mgmt
CrossAccountExecutionRoleArn	arn:aws:iam::0000:role/ stagename -stage-cross-account-role
EncryptionKeyArn	arn:aws:kms:us-east-2:0000:key/f00-f00-f00
VPC	vpc-f00f00
VpcPublicSubnet1	subnet-f00f00







5.	In the website of your Service Workbench deployment, select “Accounts” (left navigation), “AWS Accounts” (tab), “Add Account” (button). Fill in values as follows:
Field	Value
Account Name	As desired
AWS Account ID	12-digit ID of imported account
Role ARN	CrossAccountExecutionRoleArn value
AWS Service Catalog Role Arn	CrossAccountEnvMgmtRoleArn value
External ID	As speciﬁed (default: workbench)
Description	As desired
VPC ID	VPC value
Subnet ID	VpcPublicSubnet1 value
KMS Encryption Key ARN	EncryptionKeyArn value


Verify that the account now appears under ‘AWS Accounts’.

3.2	Create default index, project, and admin account

Service Workbench supports a three-tier hierarchy for managing how research resources are deployed and billed.

●	At the top level, Service Workbench supports one or more host accounts. Each of these accounts hosts, and is billed for, the compute resources deployed in the account.
●	Each host account is linked to by one or more Indexes.
●	Each Index is linked to by one or more Projects.
●	Service Workbench users are associated with Projects.

In this section, you will create an index, project, and a local user to be the administrator of the project. Users are associated with hosting accounts through projects and indexes, so the AWS account that hosts a research resource will be determined by the project and index that the user belongs to.
1.	Navigate to Accounts (left navigation), Indexes (tab), Add Index (button)
a.	Enter a unique name for the index (e.g. “index01”)
b.	Select the AWS account that you added to Service Workbench on AWS in the last section
c.	Enter a description
d.	Press the Add Index button
2.	Select the Projects tab in the Accounts interface, select Add Project (button)
a.	Enter a unique name for the project (e.g. “project01”)
b.	Select the index that you created in the last step
c.	Enter a description
d.	Leave the project admin blank (for now)
e.	Press the Add Project button
3.	Navigate to Users (left navigation), select Add Local User (button)
a.	Enter a username in email format (this does not have to be a real email address)
b.	Enter ﬁrst and last names of the user
c.	Enter a secure password for the user
d.	Select “admin” for the user role
e.	Select the project that you created in the last section for Projects
f.	Select Active for Status
g.	Press the Add Local User button
4.	Repeat this step to create a user with Researcher role, for demonstration purposes
5.	Go back to Projects under Accounts
a.	Press the Detail button next to the project that you created
b.	Select Edit
c.	For Project Admins, select the admin user you created in the last step
d.	Select Save
6.	Log out of the root account and log back in using your new admin user

3.3	Conﬁguring Workspaces and Studies
In this section, we will make one of the ﬁve default workspace types available for researchers. Workspace types appear in Service Workbench on AWS after they have been conﬁgured in AWS Service Catalog (done during the installation process). A workspace type is made available for deployment by a user by creating a Conﬁguration, which deﬁnes the size of the resource deployed as well as which roles that can deploy the conﬁguration.

3.4	Create a workspace conﬁguration

1.	Navigate to Workspace Types
a.	Select SageMaker Notebook
b.	Select Import
2.	Add a workspace conﬁguration:
a.	Select Edit in the SageMaker Notebook workspace type
b.	Select Conﬁgurations then Add Conﬁguration
c.	Fill in the Basic Information:
i.	All ﬁelds are required
ii.	Id may not contain spaces
iii.	Description and Estimate Costs are free text supporting MarkDown
d.	Fill in the Access Control:
i.	In Roles Allowed, select Admin and Researcher
e.	Fill in Input Parameters:
i.	For most ﬁelds, begin typing the name of the ﬁeld, and select the autocomplete option of that name.
ii.	For Instance type, use ml.t3.medium
f.	Add tags, if required, and select Done to create the conﬁguration
3.	Under workspace types, select Approve to make this product available for deployment by users
 
3.5	Create a study

Studies are storage locations, appearing to users as ﬁle systems mounted within their workspace. Each study is implemented as a policy-protected path in the single S3 bucket storing all data in this Service Workbench on AWS deployment. There are three classes of Studies: My Study, which are private to the user; Organization, which may be shared with other users, and Open Data, which is amount of the AWS Open Data dataset.
1.	In Service Workbench on AWS, open Studies and select Create Study
a.	ID must not contain spaces
b.	Select Organization Study type
c.	Name and Description are required
d.	Select your project from Project ID
2.	In the Organization tab, use Upload Files to store some ﬁles in your Organization Study
3.	Modify the permissions to share the study with the second user, created earlier

Note: Studies created using My Study cannot be shared with other users. Therefore, it’s recommended that users create Organization studies, which can be single-user access, or shared, if there is a possibility that the study data would be shared in the future.

3.6	Create and deploy a workspace

Workspaces can be attached to a study from the Studies interface, follow the procedures in this section. Alternatively, workspaces may be deployed directly from the Workspaces tab, however, workspaces created in this manner will not have access to study data.

1.	Open Studies, Find & Select Studies
a.	Select the Organization tab
b.	Select your study
i.	Note ‘Selected Studies: 1’ is displayed. You may attach multiple Studies to a Workspace.
c.	Select Next
2.	In Select Compute, select SageMaker Notebook
3.	From Create Workspace, launch the workspace
a.	The Name may not contain spaces
b.	The default CIDR may be widened
c.	Select your Project ID
d.	Select your conﬁguration
e.	Select Create Research Workspace
4.	Open Workspaces in the left sidebar
a.	Select Connect to open a new tab with a Jupyter Notebook running on the SageMaker workspace
 
4. Post-Deployment Tasks
Once your basic installation is complete, you can stop or terminate the AWS Cloud9 instance that you used to deploy Service Workbench on AWS, as the instance is only needed to for the deployment process. Stopping the instance, rather than terminating it, is recommended if you intend to update your Service Workbench on AWS deployment as the platform is updated.

For creating a Service Workbench on AWS deployment that can be demonstrated to researchers, it’s recommended that you create workspace conﬁgurations for all of the default workspace types (EC2 Windows, EC2 Linux, EMR, and R Studio on EC2).