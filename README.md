# VPC_lab  
#### This readme will document my learning process of virtual private cloud (VPC) on AWS  

I will be following this [video](https://www.youtube.com/watch?v=g2JOHLHh4rI)  

<!-- ![alttext](/images/filename) -->

Using an existing subscription to acloudguru (ACG), I create a sandbox AWS environment.  
  
This free account allows me to follow the video closely and create the resources.  

![](/images/acg_aws.png)

![](/images/aws_credentials.png)

###  Exercise 1. Creating a VPC with subnets  

## 1.1 VPC, Subnets
- Login to AWS console  
- Go to VPC dashboard  
- Create VPC  
- Refer to [custom-vpc.md](/Code/Amazon%20VPC/custom-vpc.md) to copy and paste the values into the forms  
- Created VPC, `Name: MyVPC IPv4 CIDR Block: 10.0.0.0/16`  

![](/images/vpc_cidr.png)

- Now to create Subnets, recall that subnets must have IPs within the original `IPv4 CIDR Block: 10.0.0.0/16`  
- VPC dashboard, left column
- Subnets -> Create subnet
- Enter the details and create all 4 subnets

![](/images/vpc_create_subnet.png)

- Select public subnets
- Go to Actions, modify auto-assign IP settings to `Enable`

## 1.2 Route table
- Create route table for private subnets
- Left column, click route tables, create route tables with name and select your VPC
- Once done, go to subnet associations, select private1a and private 1b and save
- Rename the main route table to MAIN
- Verify that the main route table does not have any subnet association but does have public1a and public1b listed

![](/images/vpc_route_table.png)
![](/images/vpc_route_table_2.png)

## 1.3 Internet gateways
- Left column, click internet gateways
- Create internet gateway with the provided name
- Attach to VPC
- Go to MAIN route table -> Routes -> Edit Routes -> Add route with the following

![](/images/vpc_route_table_3.png)

Done! Next exercise will test the setup  

### Exercise 2 Launch instances and test VPC 

## 2.1 Create NAT gateway
- Left column, NAT gateways, create NAT gateway with name MyNATGW, select public1a subnet, generate elastic IP and create
- go to route tables -> Private-RT -> Routes -> Edit route -> Add route, 0.0.0.0/0, select the created nat gateway
- now our private subnets will have access to the internet


## 2.2 Create security group
- Left column, security groups, create security group

![](/images/security_group.png)

## 2.3 Run instances, populate aws_cli variables
- Go through the video and find the variables to end up with `aws ec2 run-instances --image-id ami-090fa75af13c156b4 --instance-type t2.micro --security-group-ids sg-0357d8e60abd96b29 --subnet-id subnet-0539005e4f26f40f3 --key-name cloud-labs-nv --user-data file://user-data-subnet-id.txt`
- run `aws configure` to set up the necessary credentials
- cd to Code/Amazon VPC/ otherwise the file://user-data-subnet-id.txt cannot be read
- output should be a nice json formatted text
- verify in aws console

![](/images/ec2_1.png)

- repeat for public 1b with public 1b subnet ID

![](/images/ec2_2.png)

- repeat for private 1b with private1b subnet ID

![](/images/ec2_3.png)

- Go to your browser and key in public 1A public IP

![](/images/ec2_4.png)
![](/images/ec2_5.png)

- Nice, we are able to connect to our instaces within the VPC

- Go to Instance dashboard, select Public-1A and click connect, use ec2 instance connect and click connect, let it run

![](/images/ec2_6.png)

- get private address of public 1b

![](/images/ec2_7.png)

- go back to ec2 instance connect, you are now in public1a
- ping the private IP of public1b

![](/images/ec2_8.png)

- Great! we have two way connectivity between our instances inside the VPC. Working as intended

- Now lets try connecting to our private1b subnet

![](/images/ec2_9.png)
![](/images/ec2_10.png)

- Great! we are able to connect to private1b from public1a
- We try a `curl` command and we are able to see the header of the webservice that was successfully ran via the aws cli

![](/images/ec2_11.png)




