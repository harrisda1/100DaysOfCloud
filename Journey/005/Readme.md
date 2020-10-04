![image](https://user-images.githubusercontent.com/71992673/95006647-8565da00-05bb-11eb-85b7-432e5f7d5571.png)

Day 5 - Setup AWS CloudHSM, create CMK, and encrypt an object in a S3 bucket

## Introduction

I’ve heard about KMS before. With taking the CCP exam, I got a high level overview. In studying for the SA exam, I have encrypted a couple of objects in an S3 bucket while following along in Stephane Maarek’s course, but I have not used the CMK tool. I have typically used the default AWS encryption options. 

## Prerequisite

You will need to be familiar with how to encrypt files or objects with an encryption key. It can be as simple as creating a S3 bucket, uploading a file or object, and selecting the option to encrypt it. This mini-tutorial takes it a step further by standing up a HSM, creating a key, and then encrypting an object. 

## Use Case

- ✍️ (Show-Me) Explain in one or two sentences the use case

The most common use cases is to protect data from compromise. If your S3 bucket or even on-premise file servers are breached there is not much a threat actor can do with sensitive data that is encrypted unless they have the key to decrypt the data. With the tutorial talking about setting a CloudHSM cluster there are some other use cases for HSM such as offloading SSL/TLS traffic for web servers or verify private keys are connected to an issuing certificate authority (CA). 

## Cloud Research

Initially I was only going to create an AWS CMK, but then I decide to check out the CloudHSM setup and configuration. A lot of trials and errors, but I learned a lot about how to setup this service, query AWS documentation, and become more efficient in using google to find answers to odd issues that were not addressed in the AWS docs. I note the errors I ran into and how I resolved them in the tutorial. If I have may missed something please let me know so I can update this tutorial. 

## Try yourself

### Step 1 — Access KMS

I logged into my AWS account, clicked the Services link, and typed in CMK. It will list KMS:

![image](https://user-images.githubusercontent.com/71992673/95006879-f1e1d880-05bd-11eb-9a97-a11b15b9c22b.png)

I then clicked on Create a Key:

![image](https://user-images.githubusercontent.com/71992673/95006902-32d9ed00-05be-11eb-9f08-33f231cfa2dc.png)


### Step 2 — Choosing your CMK options

The first option is to choose between symmetric or asymmetric. I went with symmetric.

![image](https://user-images.githubusercontent.com/71992673/95006923-82b8b400-05be-11eb-98f3-d8ea3b3ea05d.png)

I then opened the advanced options and there you pick what type of key material you would like to use. The default is KMS. External will be your key material and there is checkbox for the implications of using an imported key. Custom Key Store (CloudHSM) requires more configuration. It consists of at least two HSMs (Hardware Security Modules). You control the HSMs that generate the key material for the CMKs and any related cryptographic operations.

![image](https://user-images.githubusercontent.com/71992673/95006985-19857080-05bf-11eb-87af-0d6bd8926cbc.png)

### Step 3 — Going down the CloudHSM rabbit-hole...

I wanted to check out how the CloudHSM works so I chose that option and clicked next. Since I do not have a Custom Key Store, I chose the option to create one. 

![image](https://user-images.githubusercontent.com/71992673/95007003-4cc7ff80-05bf-11eb-846b-919e0308724b.png)

### Step 4 — Creating the Cluster

And in order to do that you have to create a CloudHSM cluster. I chose my custom key store name and then clicked the link to create the cluster.

![image](https://user-images.githubusercontent.com/71992673/95007015-73863600-05bf-11eb-8e64-cde5d71b4b36.png)

It opened a new window. I clicked the create cluster button. Below that is the pricing. I thought it was very expensive, but it depends on your region. The cheapest regions are US East Ohio and US West Oregon at $1.45 per hour. Some regions are over $2 per hour. It’s not bad if you only use for a brief demo, but it can add up quickly if you don’t terminate it as soon as you’re done. 

![image](https://user-images.githubusercontent.com/71992673/95007068-1ccd2c00-05c0-11eb-8142-ec0a530ffaac.png)

For the cluster configuration, I went with the default VPC and subnets in each AZ. Since its my first cluster I chose new cluster as the Cluster source. I did not add any tags. Clicked next and it tells you again that you cannot change the VPC or subnets once you create the cluster. I then clicked Create cluster. 

It takes about 2 to 3 minutes for it to go to an uninitialized state. I then selected the cluster, clicked Actions, and clicked Initialize:

![image](https://user-images.githubusercontent.com/71992673/95007079-3e2e1800-05c0-11eb-936a-3d29bf9bf585.png)

To initialize you must create an HSM. I went with the first AZ in the list and clicked create.

![image](https://user-images.githubusercontent.com/71992673/95007085-569e3280-05c0-11eb-8019-276d967c6aa6.png)

The HSM creation took about 7 minutes to complete. 

### Step 5 — Download and sign the CSR

The next step was to download and sign the CSR. There is also an option to verify the certificate. Both have links with additional documentation.

![image](https://user-images.githubusercontent.com/71992673/95008245-fbbf0800-05cc-11eb-91a4-46dcd093d932.png)

I clicked the link and followed the steps to create the self-signed certificate. 

I went [here](https://www.openssl.org/source/) to download the most current version of OpenSSL.

One snag I ran into was getting the Verify failure, User interface error when creating a passphrase for the key. I did a google search and found out that your passphrase cannot have special characters. Once I made that change it took the passphrase. 

Another snag I ran into was signing the Cluster CSR. I was getting the ‘unknown option -CA’ error. In the AWS docs there is an example of the OpenSSL command that is used. It looks like there is a space before the \ and after it. The fix was to remove the space between the \ and the -CA. Initially it looked like this:

.csr \ -CA 100DaysOfCloud_Demo.crt 

and its supposed to look like this:

 .csr \-CA 100DaysOfCloud_Demo.crt

This also applies to the -CAkey, -CAcreateserial, and -out

I then uploaded my files. For the cluster certificate its (cluster ID)_CustomerHsmCertificate.crt. and for the issuing certificate its the customerCA.crt. I then clicked upload and initialize. 

![image](https://user-images.githubusercontent.com/71992673/95008313-8e5fa700-05cd-11eb-897b-169a55bc6ce6.png)

The initialization was complete after about 5 minutes or so. 

### Step 6 — Set the Cluster Password and Activate Cluster

The next step was to set the cluster password.

![image](https://user-images.githubusercontent.com/71992673/95008328-b5b67400-05cd-11eb-8621-f273aebcec56.png)

I clicked the Learn More link and it takes you to the AWS docs to activate the cluster by creating an EC2 instance to set the password. After I created the instance, I tried to run this command:

/opt/cloudhsm/bin/cloudhsm_mgmt_util /opt/cloudhsm/etc/cloudhsm_mgmt_util.cfg

But it did not work. After some poking around, I realized that I needed to connect the EC2 instance to the cluster. I followed the instructions from this AWS [doc](https://docs.aws.amazon.com/cloudhsm/latest/userguide/configure-sg-client-instance.html).

Once that was completed, the next step is to install the AWS CloudHSM client software into my EC2 instance. The AWS doc for this step is located [here](https://docs.aws.amazon.com/cloudhsm/latest/userguide/install-and-configure-client-linux.html). One snag I ran into was I had deleted the AWS CloudHSM cluster Security Group and removed the default Security Group that was pointed to my public IP. I figured this out when I was having trouble downloading the CloudHSM client. It kept timing out. Once I added the SG back the client downloaded immediately. 

I had stopped my EC2 instance the previous night and restarted it again today, but I was having trouble running the CloudHSM client and command line tools. The great thing about the cloud is you can terminate and spin up a new instance and that is what I did. 

I was able to install the client and CLI again. I needed a refresher on getting sudo access and navigating thru the file structure in Linux, so I was back to my friend google. This was to copy the cluster certificate to this path in the instance - opt/cloudhsm/etc/customerCA.crt. I used the scp or secure copy command referenced [here](https://angus.readthedocs.io/en/2014/amazon/transfer-files-between-instance.html). I ran this in a separate terminal session and then checked to see if the file had copied over to the destination directory. It was not working so I tried multiple directories. I then logged out of the EC2 instance and then logged back in and now can see .crt file copied in multiple places LOL. In order to copy the .crt file to the opt/cloudhsm/etc/customerCA.crt I had to switch to root access. 

Next was to activate the cluster. I followed the AWS docs from [here](https://docs.aws.amazon.com/cloudhsm/latest/userguide/activate-cluster.html). Ran into another issue with the .crt file. I named it 100DaysOfCloud_Demo.crt. I thought the customerCA.crt was just an example or demo file that was used. When I ran this command - /opt/cloudhsm/bin/cloudhsm_mgmt_util /opt/cloudhsm/etc/cloudhsm_mgmt_util.cfg - to start the cloudhsm_mgmt_util I got an error stating it could not find the customerCA.crt file so it would not enable encryption. I quit the command and tried it again and got the same error. I then renamed the file to customerCA.crt and that resolved the issue. 

I followed the rest of the steps to set the password for the Crypto Officer and now my cluster is officially active!!

### Step 7 — Create Custom Key Store

The next step was create the custom key store. First I uploaded the trust anchor certificate. I clicked the learn more link and its the customerCA.crt file. This is the 00DaysOfCloud_Demo.crt file. 

![image](https://user-images.githubusercontent.com/71992673/95008483-1befc680-05cf-11eb-97b0-34b1e7bf1b4e.png)

I created a complex password for the kmsuser account and clicked create. I got another error:

![image](https://user-images.githubusercontent.com/71992673/95008492-3aee5880-05cf-11eb-92f7-2c46ed2ffae3.png)

Basically, I had to add another HSM. The fortunate thing is I didn’t have to all the work to setup the initial one. AWS will create it from the backup per the AWS doc located [here](https://docs.aws.amazon.com/cloudhsm/latest/userguide/add-remove-hsm.html). The one step that I did different was use a different AZ. The primary one is in 2a and the new one is in 2b. Below shows both are now active:

![image](https://user-images.githubusercontent.com/71992673/95008518-738e3200-05cf-11eb-96a3-428e661bb9f8.png)

Trying the custom key store creation again. SUCCESS!

![image](https://user-images.githubusercontent.com/71992673/95008529-86a10200-05cf-11eb-8e01-4408307e07a9.png)

Now I have to wait to connect  to the cluster:

![image](https://user-images.githubusercontent.com/71992673/95008544-a20c0d00-05cf-11eb-977d-78b66effc72e.png)

Waiting and waiting and then get this:

![image](https://user-images.githubusercontent.com/71992673/95008553-b8b26400-05cf-11eb-99be-4dbc973120ba.png)

When I hovered over FAIL this is the error I got - KMS cannot connect the custom key store to its CloudHSM cluster. Error code: USER_NOT_FOUND. Back to google and the resolution is to create the kmsuser CU account first in the cluster and then update the key store password value. 

Went back to the terminal and ran this command - 

![image](https://user-images.githubusercontent.com/71992673/95008564-d54e9c00-05cf-11eb-83ed-230de547c55c.png)

I then typed help at the aws-cloudHSM prompt to find the command to create a user. The command is createUser. This is the format:

![image](https://user-images.githubusercontent.com/71992673/95008572-e7c8d580-05cf-11eb-81ea-39493e082107.png)

Once I ran that command I then got this error:

![image](https://user-images.githubusercontent.com/71992673/95008581-f911e200-05cf-11eb-9f60-de89824a8d26.png)

So I have to login with the Admin account. Makes sense. 

![image](https://user-images.githubusercontent.com/71992673/95008592-0b8c1b80-05d0-11eb-90b5-d4691b92fe2d.png)

This time it was successful in creating the account:

![image](https://user-images.githubusercontent.com/71992673/95008599-1a72ce00-05d0-11eb-9dfd-a220858fda51.png)

So even though it failed to connect, you still have to ‘disconnect’ the custom key store from the cluster. That takes about 10 minutes. I wanted to use the same name - 100DaysOfCloud - but I figured, ah instead of waiting I’ll change one character and call it 100DaysofCloud. Well that part worked, but then I got an error you must have active HSMs in multiple AZs. So the custom key store ties up the HSMs even though it fails to connect to the cluster. Once the process was complete (it actually took about 5 minutes) I then deleted the custom key store. 

Once it was deleted I was able to use the same name again with an active kmsuser account and wait for 20 minutes. The dog is restless, time to take him for a walk!

Well I came back and it failed again with the same error. I reread the docs again and tried this command to help with troubleshooting - 

aws kms describe-custom-key-stores --custom-key-store-name 100DaysOfCloud

But it did not give anymore specifics on the error. I then realized that its possible the kmsuser account is not setup on the other HSM since you must have one in at least two AZs. I ran this command - 

sudo /opt/cloudhsm/bin/configure (IP address of the 2nd HSM) 

and then ran this command - 

/opt/cloudhsm/bin/cloudhsm_mgmt_util /opt/cloudhsm/etc/cloudhsm_mgmt_util.cfg

I logged in using the same admin credentials and when I ran the listUsers command sure enough it does not have the kmsuser account. I added that account and tried connecting to the cluster again, gotta wait for 20 more minutes, sigh…

Success!

![image](https://user-images.githubusercontent.com/71992673/95008644-83f2dc80-05d0-11eb-884e-083da4ec841b.png)

So it looks like that was the fix. I did not see anything specific in the docs (its probably there, but overlooked it), but this prompt gave me a clue:

![image](https://user-images.githubusercontent.com/71992673/95008659-9c62f700-05d0-11eb-897d-831b3a4f9f9c.png)

The prompt above comes whenever you reset a password or create a user in the CloudHSM CLI. So you have to do the steps in each HSM node. 

Now we get to create our own CMKs! That step is at the beginning of this mini-tutorial when I decided to go the CloudHSM route instead of using the AWS KMS keys.

### Step 8 — Create CMK (Customer Master Key)

When you choose custom key store now this comes up after you add a custom key store:

![image](https://user-images.githubusercontent.com/71992673/95008705-12675e00-05d1-11eb-9937-a5fdd0cf64be.png)

Add a label:

![image](https://user-images.githubusercontent.com/71992673/95008718-29a64b80-05d1-11eb-83b4-ef14adfb3a67.png)

I picked my AWSAdmin account, CLI Role, and the CloudHSM roles as Key Administrators:

![image](https://user-images.githubusercontent.com/71992673/95008725-3f1b7580-05d1-11eb-82e2-bb3e59faa1b8.png)

I left the key deletion option checked. I picked the same roles for the key usage permissions. The last option is to edit the key policy if you choose to do so. Now I have a CMK!

![image](https://user-images.githubusercontent.com/71992673/95008734-58bcbd00-05d1-11eb-94db-70a915d920ac.png)

### Step 9 — Create S3 Bucket, upload file or object, and encrypt using the CMK you created!

Now to create a S3 bucket so I can upload a file and encrypt with my CMK.

![image](https://user-images.githubusercontent.com/71992673/95008752-8c97e280-05d1-11eb-99d1-46ab75700df9.png)

I went with the defaults with the exception of encryption. I chose the CMK I created:

![image](https://user-images.githubusercontent.com/71992673/95008756-9e798580-05d1-11eb-98d1-bc552bec8217.png)

I uploaded a couple of files and the only setting I changed was enabling versioning. The other settings remained unchanged and the encryption setting is using my CMK:

![image](https://user-images.githubusercontent.com/71992673/95008770-b3eeaf80-05d1-11eb-8090-fd852d548a06.png)

![image](https://user-images.githubusercontent.com/71992673/95008792-d385d800-05d1-11eb-9e8c-8d3e7b223d3d.png)

The uploads were successful. Now when I go to an object’s Properties - Encryption under AWS-KMS I have my CMK and the default S3 encryption options!

![image](https://user-images.githubusercontent.com/71992673/95008810-e9939880-05d1-11eb-9795-25dc4643c6a8.png)

You can also see it listed in the Overview tab of an object:

![image](https://user-images.githubusercontent.com/71992673/95008825-0039ef80-05d2-11eb-809a-915099315759.png)

To complete the demo I performed a teardown of all the resources I used:

- I started with emptying my S3 buckets and then deleting the buckets.
- I then disconnected the custom key store.
- Deleted both HSMs.
- Scheduled the HSM backups to delete in 7 days. 
- Deleted the CloudHSM cluster.
- Terminated the EC2 instance.
- Scheduled the CMK to delete in 7 days (It will not allow you to delete in less than 7 days) (Scroll down to check the box to delete the key in 7 days).
- After the 7 day period delete the custom key store since the CMK is now deleted.


## ☁️ Cloud Outcome

It was satisfying to complete this demo not know much about CloudHSMs. The adage learning by doing definitely applies in this mini-project and I'm looking forward to more deeper dives with the various AWS tools. 

## Next Steps

I think I'm going to try setting up an EFS shared file system - STR03-AWS200

## Social Proof

[Tweet](https://twitter.com/harristha1/status/1312646717511069696?s=20)
