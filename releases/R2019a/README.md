# MATLAB Production Server on Amazon Web Services

# Requirements

Before starting, you need the following:

-   A MATLAB® Production Server™ license. For more information, see [Configure MATLAB Production Server Licensing on the Cloud](https://www.mathworks.com/help/licensingoncloud/matlab-production-server-on-the-cloud.html). In order to configure the license in the cloud, you will need the MAC address of the license server on the cloud. You can get the license server MAC address only after deploying the solution to the cloud. For more information, see [Get License Server MAC Address](/releases/R2019a/doc/cloudConsoleDoc.md#get-license-server-mac-address).
-   An Amazon Web Services™ (AWS) account.
-   A Key Pair for your AWS account in the US East (N. Virginia) or EU (Ireland) region. For more information, see [Amazon EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).

# Costs
You are responsible for the cost of the AWS services used when you create cloud resources using this guide. Resource settings, such as instance type, will affect the cost of deployment. For cost estimates, see the pricing pages for each AWS service you will be using. Prices are subject to change.


# Introduction
The following guide will help you automate the process of running MATLAB
Production Server on the Amazon Web Services (AWS) cloud. The automation is
accomplished using an AWS CloudFormation template. The template is a JSON
file that defines the resources needed to deploy and manage MATLAB Production
Server on AWS. Once deployed, you can manage the server using the
MATLAB Production Server Cloud Console&mdash;a web-based interface to
configure and manage server instances on the cloud. For more information, see [MATLAB Production Server Cloud Console User's Guide](/releases/R2019a/doc/cloudConsoleDoc.md#matlab-production-server-cloud-console-users-guide).
For information about the architecture of this solution, see [Architecture and Resources](#architecture-and-resources). For information about AWS templates, see [AWS CloudFormation Templates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html). 

# Prepare Your AWS Account
1. If you don't have an AWS account, create one at https://aws.amazon.com by following the on-screen instructions.
2. Use the regions selector in the navigation bar to choose **US-EAST (N. Virginia)** or **EU (Ireland)** as the region where you want to deploy MATLAB Production Server.
3. Create a [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) in that region.  The key pair is necessary as it is the only way to connect to the instance as an administrator.
4. If necessary, [request a service limit increase](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-) for the Amazon EC2 instance type or VPCs.  You might need to do this if you already have existing deployments that use that instance type or you think you might exceed the [default limit](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-resource-limits.html) with this deployment.

# Deployment Steps

## Step 1. Launch the Template
Click the **Launch Stack** button to deploy resources on AWS. This will open the AWS Management Console in your web browser.

| Release | Windows Server 2016 VM | Ubuntu 16.04 VM |
|---------------|------------------------|-----------------|
| MATLAB R2019a | <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://s3.amazonaws.com/matlab-production-server-templates/MatlabProductionServer_Windows_R2019a_New.template" target="_blank">     <img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"/> </a> | <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://s3.amazonaws.com/matlab-production-server-templates/MatlabProductionServer_Linux_R2019a_New.template" target="_blank">     <img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"/> </a> |

For other releases, see [How do I launch a template that uses a previous MATLAB release?](#how-do-i-launch-a-template-that-uses-a-previous-matlab-release)
<p><strong>Note:</strong> Creating a stack on AWS can take at least 20 minutes.</p>

## Step 2. Configure the Stack
1. Provide values for parameters in the **Create Stack** page:

    | Parameter Name                         | Value                                                                                                                                                                                                                                                                                                                                                 |
    |----------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | **Stack name**                         | Choose a name for the stack. This will be shown in the AWS console. <p><em>*Example*</em>: Boston</p>                                                                                                                                                                                                                                                                       |
    | **Name of existing Key Pair**          | Choose the name of an existing EC2 Key Pair to allow access to all the VMs in the stack. For information about creating an Amazon EC2 key pair, see [Amazon EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair). <p><em>*Example*</em>: boston-keypair</p>                                                                                   |
    | **ARN of SSL Certificate**             | Provide the Amazon Resource Name (ARN) of an existing certificate in the AWS Certificate Manager to enable secure HTTPS communication to the HTTPS server endpoint. This field is optional and may be left blank to use HTTP communication instead. For more information, see [Create Self-signed Certificate](/README.md#create-self-signed-certificate) and [Upload Self-signed Certificate to AWS Certificate Manager](/README.md#upload-self-signed-certificate-to-aws-certificate-manager).<p><em>*Example*</em>: 123456789012</p>                                                                                        |
    | **Number of worker nodes**             | Choose the number of AWS instances to start for the server. <p><em>*Example*</em>: 6</p><p>If you have a standard 24 worker MATLAB Production Server license and select `m5.xlarge` (4 cores) as the **Instance type for the worker nodes**, you will need 6 worker nodes to fully utilize the workers in your license.</p><p>You can always under provision the number instances. In which case you may end up using fewer workers than you are licensed for.</p>                                                                                                                                                                                                                                                                        |
    | **Instance type for the worker nodes** | Choose the AWS instance type to use for the server instances. All AWS instance types are supported. For more information, see [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/). <p><em>*Example*</em>: m5.xlarge</p> |
    | **Allow connections from** | This is the IP address range that will be allowed to connect to the cloud console that manages the server. The format for this field is IP Address/Mask. <p><em>Example</em>: </p>10.0.0.1/32 <ul><li>This is the public IP address which can be found by searching for "what is my ip address" on the web. The mask determines the number of IP addresses to include.</li><li>A mask of 32 is a single IP address.</li><li>Use a [CIDR calculator](https://www.ipaddressguide.com/cidr) if you need a range of more than one IP addresses.</li><li>You may need to contact your IT administrator to determine which address is appropriate.</li></ul></p> |
    | **Use public IP addresses** | Choose 'Yes' if you want to make your solution available over the internet. |
    | **Create Redis ElastiCache** | Choose whether you want to create an Redis ElastiCache service. Creating this service will allow you to use the persistence functionality of the server. Persistence provides a mechanism to cache data between calls to MATLAB code running on a server instance. |

    >**Note**: Make sure you select US East (N.Virginia) or EU (Ireland) as your region from the navigation panel on top. Currently, US East and EU (Ireland) are the only supported regions.

2. Tick the box to accept that the template uses IAM roles. These roles allow server instances to write log files to the S3 bucket. For more information about IAM, see [IAM FAQ](https://aws.amazon.com/iam/faqs). 
  
3. Click the **Create** button. The CloudFormation service will start creating the resources for the stack.

## Step 3. Get Password to the Cloud Console
1. After clicking **Create** you will be taken to the *Stack Detail* page for your stack. Wait for the Status to reach **CREATE\_COMPLETE**. This may take up to 10 minutes.
1. In the Stack Detail for your stack, expand the **Outputs** section.
1. Look for the key named `MatlabProductionServerInstance` and click the corresponding URL listed under value. This will take you to the server instance (`matlab-production-server-vm`) page. 
1. Click the **Connect** button at the top.
1. In the *Connect To Your Instance* dialog box, choose **Get Password**.
1. Click **Choose File** to navigate and select the private key file (`.pem` file) for the key pair that you used while creating the stack in [Step 2](#step-2-configure-the-stack).
1. Click **Decrypt Password**. The console displays the password for the instance in the *Connect To Your Instance* dialog box, replacing the link to *Get Password* shown previously with the actual password.
1. Copy the password to the clipboard.


## Step 4. Connect to the Cloud Console
> **Note**: The Internet Explorer web browser is not supported for interacting with the cloud console.

1. In the Stack Detail for your stack, expand the **Outputs** section. 
1. Look for the key named `MatlabProductionServerVM` and click the corresponding URL listed under value. This is the HTTPS endpoint to the MATLAB Production Server Cloud Console. 



## Step 5. Log in to the Cloud Console
The username to the cloud console is **Administrator**. For the password, paste the password you copied to the clipboard by completing [Step 3](#step-3-get-password-to-the-cloud-console). The cloud console provides a web-based interface to configure and manage server instances on the cloud. For more information on how to use the cloud console, see [MATLAB Production Server Cloud Console User's Guide](/releases/R2019a/doc/cloudConsoleDoc.md#matlab-production-server-cloud-console-users-guide).  

![MATLAB Production Server Cloud Console](/releases/R2019a/images/cloudConsoleLogin.png?raw=true)

**Accept Terms and Conditions**: Access to and use of the MATLAB Production Server Cloud Console is governed by license terms in the file `C:\MathWorks\Cloud Console License.txt` (Linux: `/MathWorks/Cloud Console License.txt`) available on the `servermachine` in the resource group for this solution. 


>**Note**: The cloud console uses a self-signed certificate which can be changed. For information on changing the self-signed certificates, see [Change Self-signed Certificates](/releases/R2019a/doc/cloudConsoleDoc.md#change-self-signed-certificates).

## Step 6. Upload the License File
> **Note**: You will need a fixed MAC address to get a license file from the MathWorks License Center. For more information, see [Configure MATLAB Production Server Licensing on the Cloud](https://www.mathworks.com/support/cloud/configure-matlab-production-server-licensing-on-the-cloud.html).

1.  In the cloud console, go to **Administration** \> **Manage License**.

2.  Click **Browse License File** to select the license file you want to upload
    and click **Open**.

3.  Click **Upload**.


You are now ready to use MATLAB Production Server on AWS. 

To run applications on MATLAB Production Server, you will need to create applications using MATLAB Compiler SDK. For more information, see [Deployable Archive Creation](https://www.mathworks.com/help/mps/deployable-archive-creation.html) in the MATLAB Production Server product documentation.

# Additional Information

## Delete Your Stack

Once you have finished using your stack, it is recommended that you delete all resources to avoid incurring further cost. To delete the stack, do the following:
1. Log in to the AWS Console.
2. Go to the AWS S3 bucket page and select the S3 bucket which contains your logs.
3. Click the button labeled **Empty bucket** to delete the files in the bucket.  Follow the on-screen instructions.
4. Click the button labeled **Delete bucket** to delete the bucket itself.  Follow the on-screen instructions.
3. Go to the AWS Cloud Formation page and select the stack that you created.
3. Click the **Actions** button and click **Delete Stack** from the menu that appears.

If you do not want to delete the entire deployment but want to minimize the cost you can bring the number of instances in the Auto Scaling Group down to 0 and then scale it back up when the need arises.

## Security
When you run MATLAB Production Server on the cloud you get two HTTP/HTTPS endpoints. 

1. A HTTP/HTTPS endpoint to the application gateway/load balancer that connects the server instances. This endpoint is displayed in the home page of the cloud console and is used to make requests to the server. Whether the endpoint is HTTP or HTTPS depends on whether you provided a certificate during the creation of the stack.

1. A HTTPS endpoint to the cloud console. This endpoint is used to connect to the cloud console. The cloud console comes with a self-signed certificate.  

For information on changing the self-signed certificates, see [Change Self-signed Certificates](/releases/R2019a/doc/cloudConsoleDoc.md#change-self-signed-certificates). 

### Create Self-signed Certificate
For information on creating a self-signed certificate, see [Create and Sign an X509 Certificate](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/configuring-https-ssl.html).

### Upload Self-signed Certificate to AWS Certificate Manager

1. Open the AWS Certificate Manager.
2. Click the button at the top of the page to **Import a certificate**.
3. Copy the contents of the `.crt` file containing the certificate into the field labeled **Certificate body***.
4. Copy the contents of the `.pem` file containing the private key into the field labeled **Certificate private key***.
5. Leave the field labeled **Certificate chain** blank.
6. Click the button labeled **Review and import**.
7. Review the settings and click the **Import** button.
8. Copy the value of the Amazon Resource Name (ARN) field from the **Details** section of the certificate.

The ARN value that you copied should be pasted into the **ARN of SSL Certificate** parameter of the template in [Step 2](#step-2-configure-the-stack).

# Architecture and Resources
Deploying this reference architecture will create several resources in your
resource group.


![Architecture](/releases/R2019a/images/FinalArchitecture60.png?raw=true)

*Architecture on AWS*

### Resources

| Resource Type                                                              | Number of Resources | Description                                                                                                                                                                                                                                                                                                                        |
|----------------------------------------------------------------------------|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AWS EC2 Instance                                                           | 1                   | Virtual machine(VM) that hosts the MATLAB Production Server Cloud Console and license manager. Use the cloud console to: <ul><li> Upload your license file and start using the server</li><li>Get HTTP/HTTPS endpoint to make requests</li><li> Upload applications (.ctf files) to the server</li><li> Manage server configurations</li><li> Manage the HTTPS certificate</li></ul><p>For more information, see [MATLAB Production Server Cloud Console User's Guide](/releases/R2019a/doc/cloudConsoleDoc.md#matlab-production-server-cloud-console-users-guide).   |
| Auto Scaling Group                                                         | 1                   | Manages the number of identical VMs to be deployed. Each VM runs an instance of MATLAB Production Server which in turn runs multiple MATLAB workers.                                                                                                                                                                               |
| Load Balancer                                                              | 1                   | Provides routing and load balancing service to MATLAB Production Server instances. The MATLAB Production Server Cloud Console retrieves the HTTP/HTTPS endpoint for making requests to the server from the load balancer resource.<p>**NOTE**: Provides HTTPS endpoint to the server for making requests.</p>                                                                                           |
| S3 Bucket                                                                  | NA                  | S3 storage bucket created during the creation of the stack where logs for the reference architecture are stored.                                                                                                                                                                                                  |
| Virtual Private Cluster (VPC)                                              | 1                   | Enables resources to communicate with each other.                                           |
| Redis ElastiCache | 1 | Enables caching of data between calls to MATLAB code running on a server instance. |

# FAQ
## How do I use an existing VPC to deploy MATLAB Production Server?
You can launch the reference architecture within an existing VPC and subnet using the following template:

| Release | Windows Server 2016 VM  | Ubuntu 16.04 VM |
|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| R2019a | <a  href ="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://s3.amazonaws.com/matlab-production-server-templates/MatlabProductionServer_Windows_R2019a_Existing.template"  target ="_blank" >      <img  src ="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" />  </a> | <a  href ="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://s3.amazonaws.com/matlab-production-server-templates/MatlabProductionServer_Linux_R2019a_Existing.template"  target ="_blank" >      <img  src ="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" />  </a> |

In addition to the parameters specified in the section [Configure the Stack](#step-2-configure-the-stack), you will need to specify the following parameters in the template to use your existing VPC.

| Parameter  | Value |
|----------------------------------|--------------------------------------------------------------------------------|
| Existing VPC ID | ID of your existing VPC. |
| IP address range of existing VPC | IP address range from the existing VPC. |
| Subnet 1 ID | ID of an existing subnet that will host the cloud console and other resources. |
| Subnet 2 ID | ID of an existing subnet that will host the application gateway. |

You will also need to open the following ports in your VPC:

| Port | Description |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `443` | Required for communicating with the cloud console. |
| `8000`, `8002`, `9910` | Required for communication between the cloud console and workers within the VPC.  These ports do not need to be open to the internet. |
| `27000`, `50115` | Required for communication between network license manager and the workers. |
| `3389` | Required for Remote Desktop functionality. This can be used for troubleshooting and debugging. |

## What versions of MATLAB Runtime are supported?

| Release | MATLAB Runtime | MATLAB Runtime | MATLAB Runtime | MATLAB Runtime | MATLAB Runtime | MATLAB Runtime | MATLAB Runtime | MATLAB Runtime |
|---------------|----------------|----------------|----------------|----------------|----------------|----------------|----------------|----------------|
| MATLAB R2018a | R2015b | R2016a | R2016b | R2017a | R2017b | R2018a |  |  |
| MATLAB R2018b |  | R2016a | R2016b | R2017a | R2017b | R2018a | R2018b |  |
| MATLAB R2019a |  |  | R2016b | R2017a | R2017b | R2018a | R2018b | R2019a |


## Why do requests to the server fail with errors such as “untrusted certificate” or “security exception”?  

These errors result from either CORS not being enabled on the server or due to the fact that the server endpoint uses a self-signed 
certificate. 

If you are making an AJAX request to the server, make sure that CORS is enabled in the server configuration. You can enable CORS by editing the property `--cors-allowed-origins` in the config file. For more information, see [Edit the Server Configuration](/releases/R2019a/doc/cloudConsoleDoc.md#edit-the-server-configuration).

Also, some HTTP libraries and Javascript AJAX calls will reject a request originating from a server that uses a self-signed certificate. You may need to manually override the default security behavior of the client application. Or you can add a new 
HTTP/HTTPS endpoint to the application gateway. For more information, see [Create a Listener](/releases/R2019a/doc/cloudConsoleDoc.md#create-a-listener). 

# Enhancement Request
Provide suggestions for additional features or capabilities using the following link: https://www.mathworks.com/cloud/enhancement-request.html

# Support
Email: `cloud-support@mathworks.com`

