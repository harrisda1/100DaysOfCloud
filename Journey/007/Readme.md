![image](https://user-images.githubusercontent.com/71992673/96214952-90acf400-0f31-11eb-89f5-a07217550174.png)

Create an EFS shared file system

## Introduction

I have played with EFS briefly while studying for the SAA exam. I think this project is a good refresher and I can dig a little deeper on how it works and play with some of the advanced settings. I’m going to create the EFS first and then create the two EC2 instances. 

## Prerequisite

Basic knowledge of how to use the AWS Console, AWS CLI, and basic Linux commands. The console will be used to create the EFS file system and EC2 instances. The AWS CLI will be used to mount the EC2 instances to EFS. The basic Linux commands will be used to create and edit the text file. 

## Use Case

A basic use case would be to setup a home directories for multiple users. EFS can scale to thousands of instances and the storage is persistent so the data you save in one instance can be accessed on another and vice versa. 

## Cloud Research

In my research setting up EFS and the two EC2 instances was the easier part. The trial and errors came about when mounting the instances using IAM authorization. I created an IAM role for each instance and played with the EFS File Security Policy permissions and security groups to figure out how to write to the mount point. 

## Try yourself

Below is my mini-tutorial on based on the STR03-AWS200 - Create an EFS shared file system mini-project. 

### Step 1 — Create the EFS File System

I logged into my AWS account, clicked on the Services tab, and typed in EFS. You will then come to this screen:

![image](https://user-images.githubusercontent.com/71992673/96215538-26954e80-0f33-11eb-8277-458469b93afd.png)

I clicked on Create file system, gave my file system a name, and went with the default VPC:

![image](https://user-images.githubusercontent.com/71992673/96215592-4b89c180-0f33-11eb-8d48-dbfe900a236b.png)

I clicked the Customize button to see what other options are available. 

The first option is Automatic Backups. This is checked by default and you can incur charges. I left it checked.

![image](https://user-images.githubusercontent.com/71992673/96215618-5e9c9180-0f33-11eb-9eea-c9634fde5e64.png)

The second option is Lifecycle Management. AWS will automatically move files you use less frequently so you can save money on storage costs. The default is 30 days since a file was last used. 

![image](https://user-images.githubusercontent.com/71992673/96215665-7542e880-0f33-11eb-80a5-b5db18653451.png)

Next are Performance Mode and Throughput Mode. I stuck to the defaults of General Purpose and Bursting. 

![image](https://user-images.githubusercontent.com/71992673/96215708-8be93f80-0f33-11eb-9d78-d61cd9d04775.png)

Next was encryption. By default the box is checked to enable encryption of data at rest. If you don’t have an external key or your own CMK it will use the default AWS KMS key. When you click the Customize encryption settings tab it will give you the option to select an existing CMK or create a new one. The one I created on my previous project is in the list, but I cannot use because its pending deletion.

![image](https://user-images.githubusercontent.com/71992673/96215730-9e637900-0f33-11eb-99d4-f4a96a0c6908.png)

The last option was to add tags, but I did not add any tags. I then clicked Next. 

Now to pick the Network Access settings. The first option is the VPC. I’m using the same default VPC. 

![image](https://user-images.githubusercontent.com/71992673/96215767-b509d000-0f33-11eb-9a6d-e2e8a3344d08.png)

Next are mount targets. I stuck with the defaults. Each AZ has a default subnet and I used the same Security Group for all the AZs. 

![image](https://user-images.githubusercontent.com/71992673/96215802-c81ca000-0f33-11eb-90c1-6628b8fb3c62.png)

I then clicked Next. The next option was File system policy. I clicked Learn More to familiarize myself with the options. I decided to go with disabling root access and enforcing in-transit encryption for secure access. I did not enable the read-only access as I want to read and write data to EFS.

![image](https://user-images.githubusercontent.com/71992673/96215862-eda9a980-0f33-11eb-9aa1-1b5ca956f7d4.png)

The next window reviews all the settings and then I clicked Create. 

![image](https://user-images.githubusercontent.com/71992673/96215897-03b76a00-0f34-11eb-9244-4885ca24599c.png)

### Step 2 — Create two EC2 instances

The next step was to create two EC2 Instances. I went with the Amazon Linux 2 AMI. I chose the t2.micro which is part of the free tier. The project states to have each instance in a different AZ within the same Region. I’m using us-west-2 (Oregon) region. Under Subnet it defaults to no preference which puts your instance in any of the four AZ’s. I selected us-west-2a for Instance A and us-west-2c for Instance B.

![image](https://user-images.githubusercontent.com/71992673/96216033-3c574380-0f34-11eb-92f1-e24516fb95d8.png)

![image](https://user-images.githubusercontent.com/71992673/96216069-4bd68c80-0f34-11eb-96a9-22d5a190da82.png)

When I was configuring the instance details there is an option to add File Systems.

![image](https://user-images.githubusercontent.com/71992673/96216108-61e44d00-0f34-11eb-8099-c239d0e2841a.png)

However, I decided to not add it there and go thru the CLI as I will mount both instances using IAM authorization. I added a tag called Name and called in Instance A for the first instance and Instance B for the second one. I also created a Security Group called EFS Demo. I set the IP address for SSH to use my IP. 

### Step 3 — Mount EC2 instances to EFS

I launched a terminal window and connected to Instance A remotely. There was a prompt to run updates so I went ahead and did that.

![image](https://user-images.githubusercontent.com/71992673/96216243-ac65c980-0f34-11eb-857a-1fb3a8247867.png)

I opened another terminal window and did the same thing for Instance B.

The next thing I did was install amazon-efs-utils package. This package includes the EFS mount helper that I will need to mount each instance. I ran this command.

![image](https://user-images.githubusercontent.com/71992673/96216281-cd2e1f00-0f34-11eb-8bbb-248c8a9800b4.png)

I then went to documentation for the EFS mount helper [here](https://docs.aws.amazon.com/efs/latest/ug/mounting-fs-mount-helper.html#mounting-IAM-option) and clicked on the mounting with IAM authorization. The command to mount prompts you to enter an efs-mount-point and I was not sure what mine was. I did a google search on ‘where do you find the efs mount point’ and clicked the first link which was another AWS [doc](https://docs.aws.amazon.com/efs/latest/ug/wt1-test.html). I went to Step 3.3 which is where I saw you actually create a directory which becomes the mount point. I then used the command below on each instance to create the mount point.

![image](https://user-images.githubusercontent.com/71992673/96216423-1d0ce600-0f35-11eb-812a-d851c49008d3.png)

I then entered this command for the IAM authorization.

![image](https://user-images.githubusercontent.com/71992673/96216450-30b84c80-0f35-11eb-8b04-eef08735ca83.png)

I found the file-system-id under Amazon EFS - File Systems:

![image](https://user-images.githubusercontent.com/71992673/96216471-43328600-0f35-11eb-9d4a-b971c803e2f7.png)

I then got this message:

![image](https://user-images.githubusercontent.com/71992673/96216504-5b0a0a00-0f35-11eb-841a-34b9376a956e.png)

I then ran the aws configure command and entered my Access Key ID, Secret Access Key, Default region name, and left the output format blank.

I ran the IAM authorization command again, but got the same message. I then realized that I need an instance profile for my EC2 instance, see below:

![image](https://user-images.githubusercontent.com/71992673/96216544-6f4e0700-0f35-11eb-957f-eb444e6cd122.png)

I went to my account menu near the top right of the console and then clicked on My Security Credentials.

![image](https://user-images.githubusercontent.com/71992673/96216570-8260d700-0f35-11eb-9158-dd124b1a90e8.png)

In the menu to the left I clicked on Roles and then the blue Create Roles button.

![image](https://user-images.githubusercontent.com/71992673/96216643-a4f2f000-0f35-11eb-9191-3e7db0f15cca.png)

I chose EC2 as this role will grant us access to EFS. I then clicked Next:Permissions.

![image](https://user-images.githubusercontent.com/71992673/96216684-b9cf8380-0f35-11eb-91d3-11369d1a893c.png)

I did a search for ‘efs’ and had no results. I then did a search under ‘elastic’ and it shows the results for EFS.

![image](https://user-images.githubusercontent.com/71992673/96216719-d075da80-0f35-11eb-85e9-414fc23ce845.png)

I did not want to grant full access to the EC2. I wanted to adhere to the least privilege principal. I ended up going with ‘ClientReadWriteAccess’ policy as I had granted a similar policy within EFS when I set it up. I chose to option to create it without a permission boundary and clicked next.

![image](https://user-images.githubusercontent.com/71992673/96216738-e2f01400-0f35-11eb-9ba8-7c29f425ca6d.png)

I did not add anything under tags. I named the role EC2_Access_To_EFS and added a description and clicked Create role.

![image](https://user-images.githubusercontent.com/71992673/96216770-f8653e00-0f35-11eb-9360-9825ca9169c0.png)

I went back to the EC2 menu, selected an instance, clicked Actions - Instance Settings - Attach/Replace IAM Role.

![image](https://user-images.githubusercontent.com/71992673/96216810-0fa42b80-0f36-11eb-9761-6258d6bedcfc.png)

I clicked the dropdown box and selected the role I created - EC2_Access_To_EFS and then clicked Apply. I did the same thing for the other instance. 

I ran the IAM authorization command again and this time I got a different error:

![image](https://user-images.githubusercontent.com/71992673/96216838-221e6500-0f36-11eb-99ba-1c563e3b7375.png)

I did a google search for ‘efs mount point connection reset by peer’ and found out its related to the security group setup. I have the EC2 instances in the EFS Demo security group and the EFS in the default security group. I followed the steps per this AWS [doc](https://docs.aws.amazon.com/efs/latest/ug/network-access.html) and created a separate security group only for EFS. 

The security group for the EC2 instances has an inbound rule to only SSH to the instances from my IP:

![image](https://user-images.githubusercontent.com/71992673/96216904-49753200-0f36-11eb-8035-a08698d64576.png)

And then there is an outbound rule to the EFS security group called NFS Demo:

![image](https://user-images.githubusercontent.com/71992673/96216924-5b56d500-0f36-11eb-8e95-5de8a6bb7f13.png)

The security group for EFS has an inbound rule to only the EC2 instances security group called EFS Demo:

![image](https://user-images.githubusercontent.com/71992673/96216955-6c9fe180-0f36-11eb-9ffb-22139452c814.png)

I then updated the security group in EFS. I removed the default and added the NFS Demo security group and clicked Save. 

I ran the IAM authorization command again and this time it was successful.

### Step 4 — Create text file and access on both instances

The next step is to create a simple text file on Instance A and then open that same file on Instance B. 

I’m a little rusty on creating files within linux so I used trusty ol’ google. I followed the steps in this [article](https://www.howtogeek.com/199687/how-to-quickly-create-a-text-file-using-the-command-line-in-linux/) I created the file and added some motivating text:

![image](https://user-images.githubusercontent.com/71992673/96217096-c6a0a700-0f36-11eb-87f4-840d73d363ff.png)

Now onto Instance B. I typed in the command below to list the file I created:

![image](https://user-images.githubusercontent.com/71992673/96217131-d7511d00-0f36-11eb-9db8-49f59d1f4376.png)

But, I got the error above stating the file did not exist. Back to google again. I figured out that the file I created was saved in the wrong directory on Instance A, but even when I moved it to the mount point, I still got a permission denied error. I poked around in google trying to figure why I could not write to my text file even though I believed I had granted the correct permissions. Read/Write in the EC2 IAM role and Read/Write in the File System Policy in EFS. I entered this search term ‘unable to write to nfs mounted shares in aws’ and found this article in [stack overflow](https://stackoverflow.com/questions/63495461/cannot-write-files-on-mounted-volume-aws-efs). Basically the Engineer removed the entire File System Policy and then he could write to EFS. 

I tried that option and it worked, but I’d rather have the File System Policy setting enabled. I played around with the combinations of File System Policy settings. I changed a setting, logged out of the Terminal, logged back in, and then tried to write to my text file, rinse and repeat. After doing this for about half an hour I figured out the best possible setting. There is probably a better way to assign least privilege (please feel free to comment if you know of a better way), but settled on this. I changed the EC2 IAM role from Read/Write to Full Access and changed the File System Policy in EFS to only enforce in-transit encryption for all clients.

So with both EC2 instances volumes encrypted and EFS encrypted that covered data at rest. Enforcing in-transit encryption covered data in motion. I had to use the root command to write to the file even when I had no File System Policy settings enabled. When I enabled the disable root access feature in the File System Policy settings that denied write access even with Full Access in the IAM role. It allows me to go to root, but no edits within root. So I only have the enforced in-transit encryption setting enabled and kept Full Access as Read/Write does not have enough permissions to write. I think if I tinkered with a custom policy I might get it to work with having an IAM role with Full Access. 

Here is my sample text file again (notice I’m using root):

![image](https://user-images.githubusercontent.com/71992673/96217220-fea7ea00-0f36-11eb-868e-457561786f6d.png)

I then used the cat command to verify the data is still there:

![image](https://user-images.githubusercontent.com/71992673/96217249-12535080-0f37-11eb-9977-e2526bd920e8.png)

I then went over to Instance B and entered the same command as before (I first navigated to the correct directory):

![image](https://user-images.githubusercontent.com/71992673/96217301-2bf49800-0f37-11eb-9908-5f4d7d737d38.png)

This time it sees the file. I ran the cat command to confirm it is the same data that I wrote when I was editing the file in Instance A:

![image](https://user-images.githubusercontent.com/71992673/96217344-3c0c7780-0f37-11eb-8501-789c1ccfde41.png)

I also made edits while in Instance B, saved them, went back to Instance A, and saw the changes:

Instance B
![image](https://user-images.githubusercontent.com/71992673/96217421-63634480-0f37-11eb-8809-32bc1505a19a.png)

Instance A
![image](https://user-images.githubusercontent.com/71992673/96217463-7d048c00-0f37-11eb-9096-efd596eedfb9.png)

Teardown:
I deleted the EFS Share. 
I deleted both EC2 instances.

Answers to Project Questions:

* What are the use cases for EFS volumes? Use cases for EFS are for media processing, big data analysis, web-hosting, content management, and shared network storage. It is designed for huge amounts of data that big data workloads and analytic apps generate. https://cloud.netapp.com/blog/aws-efs-is-it-the-right-storage-solution-for-you, https://cloud.netapp.com/blog/ebs-efs-amazons3-best-cloud-storage-system
* From how many regions and availability zones can a single EFS volume be accessed? A single EFS volume can be accessed from one region and all the AZ’s within that region. https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html
* What network protocol does EFS use? The network protocol used is Network File System version 4 (NFSv4.1 and NFSv4.0) protocol https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html
* How many instances can an EFS volume be mounted on simultaneously? The minimum is one instance and the maximum is thousands of instances. https://aws.amazon.com/efs/faq/
* How is the throughput [speed] of an EFS volume determined? It is determined by the size of the data stored. The more data stored the higher the throughput and vice versa. https://aws.amazon.com/efs/faq/ 
* What do the performance and throughput mode settings influence? The performance mode settings influence latency and throughput. The General Purpose Performance mode (which is the default) has lower latency and throughput whereas the Max I/O Performance mode has slightly higher latency and higher throughput. The throughput mode settings influence the amount of data stored and throughput. Bursting throughput mode the throughput scales as the size of the file system storage grows whereas Provisioned Throughput mode you can set the throughput independent of the amount of data stored. https://docs.aws.amazon.com/efs/latest/ug/performance.html
* How can access to an EFS volume be limited? Access to an EFS volume can be limited by security group rules, IAM policies, file system policies, and EFS Access Points. https://aws.amazon.com/efs/faq/
* Does EFS support security groups? Yes. https://docs.aws.amazon.com/efs/latest/ug/network-access.html
* Does EFS support encryption at rest and in-transit? Yes. https://docs.aws.amazon.com/efs/latest/ug/encryption.html
* What is EFS Infrequent Access (EFS IA)? EFS IA is a storage class that is at a lower cost and is designed for long-lived infrequently accessed files cost effectively. https://docs.aws.amazon.com/efs/latest/ug/storage-classes.html
* What is the operating system requirement for mounting an EFS volume? It must be a Linux OS. https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html
* What are the key differences between EFS and EBS volumes? The key differences between the two types of file systems are EFS is a managed file system that can be shared across multiple EC2 instances whereas EBS is block storage that can be connected to one EC2 instance. EFS scales, EBS does not scale. EFS data is replicated across multiple AZ’s in the same region whereas EBS data is within one AZ. EFS has no limit of the file system whereas EBS has a max storage size of 16TB. https://medium.com/awesome-cloud/aws-difference-between-efs-and-ebs-8c0d72a348ad


## ☁️ Cloud Outcome

The biggest lesson I learned was regarding permissions. It's actually something that I will continually learn as AWS permissions can be very granular. The majority of time spent on this mini-project was figuring out the right set of permissions, but as I stated in the tutorial, there is probably a better way to assign permissions that follow the least privilege principle. 

## Next Steps

I am going to try this project - COM04-AWS100 - Push a Docker image to Amazon ECR repository. I've read a little about Docker and this will be an opportunity to do a project. 

## Social Proof

[Tweet](https://twitter.com/harristha1/status/1316987699581124608?s=20)
