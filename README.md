# REQUIREMENTS

Before we get started, there are a few things that you’ll need to put in place:

An AWS account with admin or power user privileges. Since we’ll be creating, modifying, and deleting things in this exercise, the account should be a sandbox account that does not have access to production VMs, files, or databases.
Access to a Linux shell environment with an active internet connection.
Some experience working with Python and the Bash command line interface.

# GETTING CONFIGURED

Let’s get our workstation configured with Python, Boto3, and the AWS CLI tool. While the focus of this tutorial is on using Python, we will need the AWS CLI tool for setting up a few things.

Once we’re set up with our tools on the command line, we’ll go to the AWS console to set up a user and give permissions to access the services we need to interact with.

# PYTHON AND PIP

First, check to see if Python is already installed. You can do this by typing which python in your shell. If Python is installed, the response will be the path to the Python executable. If Python is not installed, go to the Python.org website for information on downloading and installing Python for your particular operating system.

We will be using Python 2.7.10 for this tutorial. Check your version of Python by typing python -V. Your install should work fine as long as the version is 2.6 or greater.

The next thing we’ll need is pip, the Python package manager. We’ll use pip to install the Boto3 library and the AWS CLI tool. You can check for pip by typing which pip. If pip is installed, the response will be the path to the pip executable. If pip is not installed, follow the instructions at pip.pypa.io to get pip installed on your system.

Check your version of pip by typing "pip -V". Your version of pip should be 9.0.1 or newer.

Now, with Python and pip installed, we can install the packages needed for our scripts to access AWS.

# AWS CLI TOOL AND BOTO3

Using the pip command, install the AWS CLI and Boto3:



```pip install awscli boto3 -U --ignore-installed six```

Please Note: This command may need to be run with sudo to allow for installation with elevated privileges. The -U option will upgrade any packages if they are already installed. On some systems, there may be issues with a package named "six"; using the "--ignore-installed six" option can work around those issues.
We can confirm the packages are installed by checking the version of the AWS CLI tool and loading the boto3 library.

Run the command "aws --version" and something similar to the following should be reported:

aws-cli/1.11.34 Python/2.7.10 Darwin/15.6.0 botocore/1.4.91

Finally, run the following to check boto3: python -c “import boto3”. If nothing is reported, all is well. If there are any error messages, review the setup for anything you might have missed.

At this point, we’re ready for the last bit of configuration before we start scripting.

USERS, PERMISSIONS, AND CREDENTIALS

Before we can get up and running on the command line, we need to go to AWS via the web console to create a user, give the user permissions to interact with specific services, and get credentials to identify that user.

Open your browser and navigate to the AWS login page. Typically, this is https://console.aws.amazon.com/console/home.

Once you are logged into the console, navigate to the Identity and Access Management (IAM) console. Select “Users” -> “Add user."


 ![Image](https://raw.githubusercontent.com/managedkaos/AWS-Python-Boto3/master/images/image1.png)

On the “Add user” page, give the user a name and select “Programmatic access." Then click “Next: Permissions." In this example, I’ve named the user “python-user-2." Note that your user name should not have any spaces or special characters.

![image2](https://raw.githubusercontent.com/managedkaos/AWS-Python-Boto3/master/images/image2.png)

On the “Permissions” page, we will set permissions for our user by attaching existing policies directly to our user. Click “Attach existing policies directly." Next to “Filter”, select “AWS managed." Now search for 



AmazonEC2FullAccess. After entering the search term, click the box next to the listing for “AmazonEC2FullAccess." Repeat this step for S3 and RDS, searching for and selecting AmazonS3FullAccess and AmazonRDSFullAccess. Once you’ve selected all three, click “Next: Review."



![image3](https://raw.githubusercontent.com/managedkaos/AWS-Python-Boto3/master/images/image3.png)



On the review screen, check your user name, AWS access type, and permissions summary. It should be similar to the image below. If you need to fix anything, click the “Previous” button to go back to prior screens and make changes. If everything looks good, click “Create user."



![image4.png](https://raw.githubusercontent.com/managedkaos/AWS-Python-Boto3/master/images/image4.png)



On the final user creation screen, you’ll be presented with the user’s access key ID and secret access key. Click the “Download .csv” button to save a text file with these credentials or click the “Show” link next to the secret access key. IMPORTANT: Save the file or make a note of the credentials in a safe place as this is the only time that they are easily captured. Protect these credentials like you would protect a username and password!

Now that we have a user and credentials, we can finally configure the scripting environment with the AWS CLI tool.

Back in the terminal, enter aws configure. You’ll be prompted for the AWS access key ID, AWS secret access key, default region name, and default output format. Using the credentials from the user creation step, enter the access key ID and secret access key.

For the default region name, enter the region that suits your needs. The region you enter will determine the location where any resources created by your script will be located. You can find a list of regions in the AWS documentation. In the example below, I’m using us-west-2.

Options for the default output format are text, json, and table. Enter “text” for now.

```AWS Access Key ID [None]: AKIAJFUD42GXIN4SQRKA 
AWS Secret Access Key [None]: LLL1tjMJpRNsCq23AXVtZXLJhvYkjHeDf4UO9zzz
Default region name [None]: us-west-2
Default output format [None]: text
Now that your environment is all configured, let’s run a quick test with the AWS CLI tool before moving on. In the shell, enter:
 ```


```aws ec2 describe-instances```


If you already have instances running, you’ll see the details of those instances. If not, you should see an empty response. If you see any errors, walk through the previous steps to see if anything was overlooked or entered incorrectly, particularly the access key ID and secret access key.


# SCRIPTING EC2

The Elastic Compute Cloud (EC2) is a service for managing virtual machines running in AWS. Let’s see how we can use Python and the boto3 library with EC2.

# LIST INSTANCES

For our first script, let’s list the instances we have running in EC2. We can get this information with just a few short lines of code.

First, we’ll import the boto3 library. Using the library, we’ll create an EC2 resource. This is like a handle to the EC2 console that we can use in our script. Finally, we’ll use the EC2 resource to get all of the instances and then print their instance ID and state. Here’s what the script looks like:
```
#!/usr/bin/env python
import boto3
ec2 = boto3.resource('ec2')
for instance in ec2.instances.all():
    print instance.id, instance.state
```
Save the lines above into a file named list_instances.py and change the mode to executable. That will allow you to run the script directly from the command line. Also note that you’ll need to edit and chmod +x the remaining scripts to get them running as well. In this case, the procedure looks like this:

```
$ vi list_instances.py
$ chmod +x list_instances.py
$ ./list_instances.py
```
If you haven’t created any instances, running this script won’t produce any output. So let’s fix that by moving on to the the next step and creating some instances.

# CREATE AN INSTANCE

One of the key pieces of information we need for scripting EC2 is an Amazon Machine Image (AMI) ID. This will let us tell our script what type of EC2 instance to create. While getting an AMI ID can be done programmatically, that's an advanced topic beyond the scope of this tutorial. For now, let’s go back to the AWS console and get an ID from there.

In the AWS console, go the the EC2 service and click the “Launch Instance” button. On the next screen, you’re presented with a list of AMIs you can use to create instances. Let’s focus on the Amazon Linux AMI at the very top of the list. Make a note of the AMI ID to the right of the name. In this example, its “ami-1e299d7e." That’s the value we need for our script. Note that AMI IDs differ across regions and are updated often so the latest ID for the Amazon Linux AMI may be different for you.



# GETTING AN AMI ID

![image5.png](https://raw.githubusercontent.com/managedkaos/AWS-Python-Boto3/master/images/image5.png)


Now with the AMI ID, we can complete our script. Following the pattern from the previous script, we’ll import the boto3 library and use it to create an EC2 resource. Then we’ll call the create_instances() function, passing in the image ID, max and min counts, and the instance type. We can capture the output of the function call which is an instance object. For reference, we can print the instance’s ID.

```
#!/usr/bin/env python
   import boto3
   ec2 = boto3.resource('ec2')
   instance = ec2.create_instances(
       ImageId='ami-1e299d7e',
       MinCount=1,
       MaxCount=1,
       InstanceType='t2.micro')
   print instance[0].id
```
While the command will finish quickly, it will take some time for the instance to be created. Run the list_instances.py script several times to see the state of the instance change from pending to running.

# TERMINATE AN INSTANCE

Now that we can programmatically create and list instances, we also need a method to terminate them.

For this script, we’ll follow the same pattern as before with importing the boto3 library and creating an EC2 resource. But we’ll also take one parameter: the ID of the instance to be terminated. To keep things simple, we’ll consider any argument to the script to be an instance ID. We’ll use that ID to get a connection to the instance from the EC2 resource and then call the terminate() function on that instance. Finally, we print the response from the terminate function. Here’s what the script looks like:

```#!/usr/bin/env python
import sys
import boto3
ec2 = boto3.resource('ec2')
for instance_id in sys.argv[1:]:
    instance = ec2.Instance(instance_id)
    response = instance.terminate()
    print response
```
Run the list_instances.py script to see what instances are available. Note one of the instance IDs to use as input to the terminate_instances.py script. After running the terminate script, we can run the list instances script to confirm the selected instance was terminated. That process looks something like this:

```
$ ./list_instances.py
i-0c34e5ec790618146 {u'Code': 16, u'Name': 'running'}
```
```
$ ./terminate_instances.py i-0c34e5ec790618146
{u'TerminatingInstances': [{u'InstanceId': 'i-0c34e5ec790618146', u'CurrentState': {u'Code': 32, u'Name': 'shutting-down'}, u'PreviousState': {u'Code': 16, u'Name': 'running'}}], 'ResponseMetadata': {'RetryAttempts': 0, 'HTTPStatusCode': 200, 'RequestId': '55c3eb37-a8a7-4e83-945d-5c23358ac4e6', 'HTTPHeaders': {'transfer-encoding': 'chunked', 'vary': 'Accept-Encoding', 'server': 'AmazonEC2', 'content-type': 'text/xml;charset=UTF-8', 'date': 'Sun, 01 Jan 2017 00:07:20 GMT'}}}
```
```
$ ./list_instances.py
i-0c34e5ec790618146 {u'Code': 48, u'Name': 'terminated'}
```
