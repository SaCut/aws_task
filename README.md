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




- outbount rules


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





