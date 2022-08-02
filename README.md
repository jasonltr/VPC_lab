# VPC_lab  
#### This readme will document my learning process of virtual private cloud (VPC) on AWS  

I will be following this [video](https://www.youtube.com/watch?v=g2JOHLHh4rI)  

<!-- ![alttext](/images/filename) -->

Using an existing subscription to acloudguru (ACG), I create a sandbox AWS environment.  
  
This free account allows me to follow the video closely and create the resources.  

![](/images/acg_aws.png)

![](/images/aws_credentials.png)

###  Exercise 1. Creating a VPC with subnets  



- Login to AWS console  
- Go to VPC dashboard  
- Create VPC  
- Refer to custom-vpc.md to copy and paste the values into the forms  
- Created VPC, `Name: MyVPC IPv4 CIDR Block: 10.0.0.0/16`  

![](/images/vpc_cidr.png)

- Now to create Subnets, recall that subnets must have IPs within the original `IPv4 CIDR Block: 10.0.0.0/16`  

