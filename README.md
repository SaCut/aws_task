# AWS
##### Task:
- Launch an ec2 instance with correct version of ubuntu
- ssh into the instance
- update 
- upgrade
- install nginx 
- access nignx page with public IP
- share the IP in the chat

### Tro-tier deployment on AWS
- ![img](https://i.imgur.com/VGDPtyL.png)

### EC2 instance for our App
- Go to AWS EC2 service
- Select `Launch Instances`
- Choose the right image, in our case Ubuntu 16.04 server, without SQL
- Proceed with the minit2 machine type (preselected)
- In the details, select the correct VPC (in this case DevOpsStudents (default)) and subnet (in our case, DevOpsStudents default, 1c region)
- Enable auto-assigning a public IPv4 address
- Proceed to the next page, then to the next again
- Add a tag to the machine, specifically a sensible name (in this case, `eng84_name_2tier_app`)
- In the next page, change the SSH connection to only be available to your IP address (myIP)
- add another rule, HTTP connections from anywhere
- you can add a sensible description if needed
- Go to the next page, then finally launch the instance
- SSH into the machine from command line with this command: `ssh ~/.ssh/NAME_OF_THE_PEM_FILE.pem NAME_OF_YOUR_INSTANCE@INSTANCE_PUBLIC_IP_ADDRESS`
- transfer a folder to your instance with this command `scp -ri ~/.ssh/NAME_OF_THE_PEM_FILE.pem /FOLDER_TO_BE_COPIED_PATH NAME_OF_YOUR_INSTANCE@INSTANCE_PUBLIC_IP_ADDRESS`
- find your provision file and run it with `sudo /PATH_TO_FILE/PROVISION_NAME.sh`
- check if NginX is running with `systemctl status nginx`
- if it doesn't, debug
- if it does, the app_instance setup is mostly done

### EC2 instance for our database
- Go to AWS EC2 service
- Select `Launch Instances`
- Choose the right image, in our case Ubuntu 16.04 server, without SQL
- Proceed with the minit2 machine type (preselected)
- In the details, select the correct VPC (in this case DevOpsStudents (default)) and subnet (in our case, DevOpsStudents default, 1c region)
- Enable auto-assigning a public IPv4 address
- Proceed to the next page, then to the next again
- Add a tag to the machine, specifically a sensible name (in this case, `eng84_name_2tier_db`)
- In the next page, change the SSH connection to only be available to your IP address (myIP)
- add another rule, custom HTTP connection from the _app instance private IPv4
- add another rule, HTTP connections from anywhere (this will need to be changed later, otherwise the db will be open to the world, which is not what we want)
- add sensible descriptions if needed
- Go to the next page, then finally launch the instance
- SSH into the machine from command line with this command: `ssh -io ~/.ssh/NAME_OF_THE_PEM_FILE.pem ProxyCommand="ssh -i ~/.ssh/NAME_OF_THE_PEM_FILE.pem -W %h:%p NAME_OF_YOUR_INSTANCE@APP_PUBLIC_IP" NAME_OF_YOUR_INSTANCE@INSTANCE_PUBLIC_IP_ADDRESS`
- transfer a folder to your instance with this command `scp -ri ~/.ssh/NAME_OF_THE_PEM_FILE.pem /FOLDER_TO_BE_COPIED_PATH NAME_OF_YOUR_INSTANCE@INSTANCE_PUBLIC_IP_ADDRESS`
- find your provision file and run it with `sudo /PATH_TO_FILE/PROVISION_NAME.sh`
- this should be done

### connect the two instances and seed the db
- to allow users to access the `/posts` address, run `sudo echo "export DB_HOST=mongodb://DB_PRIVATE_IP:27017/posts" >> ~/.bashrc` inside the app terminal
- then run `source ~/.bashrc` to update the current shell
- still in the app instance, run `cd ~/app/seeds/` then run `node seed.js` to fill in the database
- run `cd ..`
- then `node app.js`
- if everithing worked, you should be able to go to `http://APP_PUBLIC_IP/posts` and see the content, from anywhere in the world

### What is a VPC
- VPC virtual private cloud to define and control virtual networ
- VPC enables you to launch AWS resources into a virtual network that you've. This virtual network closely resembles a traditionl network that you'd operate in your data centre
- It allows us to EC2 instances to communicate with each other, we can also create multiple subnets within out VPC
- it benefits us with scalability of infrastruture of AWS

### Internet gateway
- Inetnet is the point which allowed us to connect to Internet
- A gateway that you attach to your VPC to enable communication between resources in your VPC and the internet

### What is a Subnet
- Network inside the VPC, they make network more sufficient 
- A range of IP addressess in your VPC 
- A subnet could have multiple ec2 instances

### Route Table
- Set of rules, called routes
- Route tables are used to determine where external network traffice is directed 

### NACLS
- NACLS are an added layer of defence they work at the network level
- NACLs are stateless, you have to have rules to allow the request to come in and to allow the response to go back out
- Public NACLs Inbound Rules
- 100 Allows inbound HTTP 80 traffic from any IPv4 address.
- 110 Allows inbound SSH 22 traffic from your network over the internet
- 120 allows inbount return traffic from hosts on the internet that are responding to requests originating in the subnet - TCP 1024-65535

#### NACLs outbound Rules
- 100 to allow port 80
- 110 we need the cirdr block and allow 27017 for outbound access to our Mongo DB server in private subnet
- 120 to allow short lived ports between 1024-65535

#### Private NACLs Inbound Rules
- Inbound rules
- 
- outbount rules
- 

### What is Security Group
- Security groups work as a firewall on the instance level
- They are attached to the VPC and subnet
- They have inbound and outbound traffic rules defined 
- Security groups are stateful, if you allowed inbount rule that will automatically be allowed outbound
 
 ### What are the Ephemeral ports
- They are shortly lived ports, they are automatically allocated based on the demand
- Allows outbound responses to clients on the internet
- they range from 1024-65535

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html

#### Bastion
- A bastion host is a server instance itself that you must SSH into before you are able to SSH into any of the other servers in your VPC.
- Bastion allows a more secure way of getting into your servers because they become accessible only from Bastion.
- ![img](https://i.imgur.com/gBWbRe1.png)
- To set up a bastion server:
- Create a server instance the usual way
- Its Security Group needs to be connected with the internet gateway of the VPC
- Its needs to have an SSH connection rule with all the personal IPs of the eventual users
- Allow an SSH inbound connection on the Security Groups of the machines to be connected
- after that, we need a config file on the local machine, with these commands:
- `touch ~/.ssh/config`
- 
```shell
echo "Host bastion
    Hostname PUBLIC_BASTION_IP
    User ubuntu
    IdentityFile ~/.ssh/NAME_OF_THE_PEM_FILE.pem
    ForwardAgent yes" | tee ~/.ssh/config
```
- Then run `ssh bastion`, and you should be allowed inside the bastion instance
- Here, run `touch ~/.ssh/config`
- 
```shell
echo "Host app
    Hostname APP_PRIVATE_IP
    user ubuntu
Host db
    Hostname DB_PRIVATE_IP
    user ubuntu" | tee ~/.ssh/config
```
- Now you should be able to run `ssh app`/`ssh db` to ssh into the shells of the two instances

### S3 setup
- uses cases 
- who is using S3 in the industry
- setting up s3, dependencies
- configure AWSCLI
- how can we get the authentication done to talk with S3
- AWS access and secret
- we will apply crud
- 53: you will have a backup available to apply CRUD in the console of AWS

**we need running EC2 to ssh into the instance and AWS access and secret key**

- S3 is a Simple storage service provided by AWS
- It is used to store and retrieve any amount of data, at anytime, from around the world
- We can also host our static website on 13 
- Create a bucket from AWSCLIT
- upload data
- download data
- delete data
- permissons of the bucket

- in order have AWSCLI we need to install the required dependencies
- python `sudo apt-get install python`
- pip `sudo apt-get install python-pip`
- aws `sudo apt-get install aws`

- `aws configure` to init aws console, you will need the AWS Access Key ID, AWS Secret Access Key, the default version and the default output format 
- `aws s3 mb s3://BUCKET_NAME --region eu-west-1` to create a bucket (no capital letters, no underscores, other similar rules)
- `aws s3 cp README.md s3://BUCKET_NAME` to send files to the bucket
- `aws s3 sync s3://BUCKET_NAME FILE_NAME` to download files from bucket
- `aws s3 rm s3://BUCKET_NAME/FILE_NAME` to delete a file from the bucket
- `aws s3 rb s3://BUCKET_NAME` to remove the bucket

##### available AWS s3 command list
- `cp` copy item
- `ls` list buckets
- `mb` make bucket
- `mv` move item
- `presign`
- `rm` remove item
- `rb` remove bucket
- `sync` pull the content of the bucket to the local machine
- `website`

