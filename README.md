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

