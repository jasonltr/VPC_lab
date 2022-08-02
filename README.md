# VPC_lab  
## This readme will document my learning process of virtual private cloud (VPC) on AWS  

I will be following this [video](https://www.youtube.com/watch?v=g2JOHLHh4rI)  

<!-- ![alttext](/images/filename) -->

Using an existing subscription to acloudguru (ACG), I create a sandbox AWS environment.  
  
This free account allows me to follow the video closely and create the resources.  

![](/images/acg_aws.png)

![](/images/aws_credentials.png)

##  Exercise 1. Creating a VPC with subnets  

### 1.1 VPC, Subnets
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

### 1.2 Route table
- Create route table for private subnets
- Left column, click route tables, create route tables with name and select your VPC
- Once done, go to subnet associations, select private1a and private 1b and save
- Rename the main route table to MAIN
- Verify that the main route table does not have any subnet association but does have public1a and public1b listed

![](/images/vpc_route_table.png)
![](/images/vpc_route_table_2.png)

### 1.3 Internet gateways
- Left column, click internet gateways
- Create internet gateway with the provided name
- Attach to VPC
- Go to MAIN route table -> Routes -> Edit Routes -> Add route with the following

![](/images/vpc_route_table_3.png)

Done! Next exercise will test the setup  

## Exercise 2 Launch instances and test VPC 

### 2.1 Create NAT gateway
- Left column, NAT gateways, create NAT gateway with name MyNATGW, select public1a subnet, generate elastic IP and create
- go to route tables -> Private-RT -> Routes -> Edit route -> Add route, 0.0.0.0/0, select the created nat gateway
- now our private subnets will have access to the internet


### 2.2 Create security group
- Left column, security groups, create security group

![](/images/security_group.png)

### 2.3 Run instances, populate aws_cli variables
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

- Nice, we are able to connect to our instances within the VPC

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


## Exercise 3 Set up Security Groups and Network Access Control List

- Recall Security groups are at instance level
- Security groups are stateful firewall, allows the return traffic automatically
- Network ACL are stateless firewall, checks for an allow rule for both connections (both ways)

![](/images/sg_best.png)
- We will be trying to do this

### 3.1 Test various SG configurations  
- Go to EC2, 
- Create new SG

![](/images/sg_1.png)

- This new SG will only allow inbound traffic from the public web service that we created
- Now we need to assign it to our instance
- EC2 -> Instances -> Private-1B -> Actions -> Security -> Change Security group -> Remove public web and add private-app

![](/images/sg_2.png)

- EC2 -> Security Groups -> Public-Web -> Edit inbound rules

![](/images/sg_3.png)

- Only my machine (IP) will be able to access the public web service now, we can test this using a vpn that masks your IP.

- Change the inbound rule to any ipv4, as this is a webservice, we want anyone to be able to access it.

![](/images/sg_5.png)

- Go to instances and connect to public1A via ec2 instance connect, but this will fail
- Go to public web security group, add inbound rule, type SSH port 22, any ipv4 address, save rule

![](/images/sg_6.png)

- Connect to public1A again
- Get private IP of private1b
- run `curl -s <private1bIP>`

![](/images/sg_7.png)

- the curl works, recall for SG, it is a staeful firewall, so the inbound rules set up are sufficient

### 3.2 Test various NACL configurations 

- go to vpc > network ACLs > MyVPC ACL > edit inbound rules > add rule numebr 99, type http, deny my own IP

![](/images/sg_8.png)

- now the site can't be reached

### 3.3 Delete public 1b

- run `aws ec2 terminate-instances --instance-ids i-079f6ccd82c65ce2c`

![](/images/terminate_instance.png)

## Exercise 4 VPC peering
- allows routing of traffic between VPCs using private IPv4 or IPv6 addresses, not via public internet
- CIDR blocks cannot overlap (challenging as VPCs may be in different accounts, created at different times etc.)
- the Private addresses of the instances will be used for the connection

### 4.1 Set up vpc peering

![](/images/peer_1.png)

- this is what we will be creating in the lab
- Create a new VPC, 4 subnets, route table and internet gateway as per the [custom-vpc-prod.md](/Code/Amazon%20VPC/custom-vpc-prod.md) file
- Go VPC > Peering connections > Create peering connection

![](/images/peer_2.png)
- A peering request has been sent
![](/images/peer_3.png)
![](/images/peer_4.png)
- Accept the request (imagine using two different accounts)

- Create security group for PROD (VPC2)
![](/images/peer_5.png)
- the inbound rules allow connections from VPC1 with CIDR block 10.0.0.0/16

- Now we launch an instance that uses this SG
- Choose the correct VPC, subnet(public1a) and SG

![](/images/peer_6.png)

- Create security group for MGMT (VPC1)
- the inbound rules allow connections from VPC2 with CIDR block 10.1.0.0/16
![](/images/peer_7.png)

- Ensure instances are set up for testing
- Make sure to configure route table for public1a (VPC1) to (VPC2), and route table for public1a (VPC2) to (VPC1)
- Make sure to switch public1a mgmt instance  to VPCPEER-MGMT security group
- Make sure to swtich public1a prod instance to  VPCPEER-PROD security group

![](/images/peer_8.png)

- Now we try to ping public1A(VPC2) private IP from public1A(VPC1)

![](/images/peer_9.png)
![](/images/peer_10.png)

- Success! VPC peering is set up for these two instances to interact with each other via private IP

## Exercise 5 Create VPC endpoint

### 5.1 create VPC endpoint, S3 bucket
- allows instances to connect to other services via private IP, using interface or gateway endpoints

![](/images/endpoint_1.png)
![](/images/endpoint_2.png)

- VPC > endpoint > create endpoints

![](/images/endpoint_3.png)

- search s3, gateway, select VPC, select Main route table
- leave policy as full access for now
- go to route table to verify the endpoint is created and added

![](/images/endpoint_4.png)

- enter IAM in search bar, select IAM
- create role > ec2 > type s3 into search bar > AmazonS3FullAccess > create role
- EC2 > Public 1A mgmt > Actions > Security > Modify IAM role > select the role u just created and save
- S3 > create bucket with unique name > create bucket
- Upload some files into the S3
- EC2 > public1A mgmt > connect via ec2 instance connect
- we should be able to access s3 via public1A mgmt
- after updating the endpoint policy to deny, we can see the same commands will return an access denied

![](/images/endpoint_5.png)

### 5.2 Bucket policy
- this will prevent access from everywhere except the VPC endpoint 
- S3 > go to our bucket > Permissions > bucket policy > edit > copy bucket arn > paste in [Bucket-Policy-VPCE.json](/Code/Amazon%20VPC/Bucket-Policy-VPCE.json)
- copy and paste endpoint ID also

![](/images/endpoint_6.png)

- from my own terminal, able to use aws cli to see the bucket, but cannot access the files due to the updated s3 bucket policy

![](/images/endpoint_7.png)

- Funnily enough, the bucket poilcy actually locks us out, even from aws console. So enter the command to delete the s3 from public1a instance

### End of VPC basics
