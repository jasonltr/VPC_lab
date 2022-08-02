# Launch instance in Public 1A
aws ec2 run-instances --image-id ami-090fa75af13c156b4 --instance-type t2.micro --security-group-ids sg-0357d8e60abd96b29 --subnet-id subnet-0539005e4f26f40f3 --key-name cloud-labs-nv --user-data file://user-data-subnet-id.txt


# Launch instance in Public 1B
aws ec2 run-instances --image-id ami-090fa75af13c156b4 --instance-type t2.micro --security-group-ids sg-0357d8e60abd96b29 --subnet-id subnet-0a1741c1dbf606ae4 --key-name cloud-labs-nv --user-data file://user-data-subnet-id.txt


# Launch instance in Private 1B
aws ec2 run-instances --image-id ami-090fa75af13c156b4 --instance-type t2.micro --security-group-ids sg-0357d8e60abd96b29 --subnet-id subnet-075fdbc46e192642c --key-name cloud-labs-nv --user-data file://user-data-subnet-id.txt



# Terminate instances

aws ec2 terminate-instances --instance-ids <value> <value>