Implementing a Team Communication Solution using Mattermost and AWS. 

This is a scalable solution that can be hosted on a on-premises data center or on a public cloud,with its servers, storage etc. completely managed and controlled by your IT team in accordance with a company’s governance and security requirements.

Step 1: VPC and Subnet Creation

a. Creation of VPC

1) Navigate to VPC using the Services button at the top of the screen
2) Select "Your VPCs" on the left side of the screen
3) Click on "Create VPC"
4) Enter the following fields :

Name: Project 1 VPC (give any suitable name)
IPv4 CIDR Block : 10.0.0.0/16
The rest of the options can be ignored
5) Select "Create VPC"
6) Select the VPC and click on Actions->Edit DNS hostnames
7) Enable DNS hostnames and click on Save

b. Creation of public subnet


1) Navigate to VPC->Subnets
2) Click on "Create Subnet"
3) Enter the following fields
Name tag : Public Subnet
VPC : Select the Project 1 VPC
IPv4 CIDR block : 10.0.1.0/24
The other options can be ignored
4) Click on Create
5) Once the subnet has been created, select the subnet and click on Actions->Modify Auto-assign IP settings
6) Enable the option "Auto assign IPv4" and select Save

c. Creation of private subnet

1) Navigate to VPC->Subnets
2) Click on "Create Subnet"
3) Enter the following fields
Name tag : Private Subnet
VPC : Select the Project 1 VPC
IPv4 CIDR block : 10.0.2.0/24
The other options can be ignored
4) Click on Create


Step 2: Internet Gateway and VPC

Creation and Configuration of Internet Gateway

1) Navigate to VPCs->Internet Gateway
2) Click on "Create Internet Gateway"
3) Enter the name tag "Project 1 Internet Gateway" and click on "Create Internet Gateway"
4) After the gateway is created, select it and click on Actions->Attach to VPC
5) Select the Project 1 VPC and click on "Attach Internet Gateway"


Creation and Configuration of Internet Gateway

1) Navigate to VPCs->Internet Gateway
2) Click on "Create Internet Gateway"
3) Enter the name tag "Project 1 Internet Gateway" and click on "Create Internet Gateway"
4) After the gateway is created, select it and click on Actions->Attach to VPC
5) Select the Project 1 VPC and click on "Attach Internet Gateway"

Creation of public route table

1) Navigate to VPC -> Route Tables and click on Create Route table
2) Enter the name tag "Public Route Table", select the Project 1 VPC from the dropdown and click on Create
3) Once the route table is created, select it and select the Routes tab below the list of route tables
4) Click in Edit Routes and add the following route (Don't edit the existing one)
- Destination : 0.0.0.0/0
- Target : Select Internet Gateway and the select the Project 1 Internet Gateway
Click on Save Routes
5) Select the Subnet Associations tab and click on Edit Subnet Associations
6) Select the Public Subnet from the list and click on Save

Creation of NAT gateway

1) Navigate to VPC using the Services button at the top of the screen
2) Select NAT Gateway at the left side of the screen
3) Click on Create NAT Gateway
- Deploy it in the public subnet
- Connectivity type : Public
- Allocate an elastic IP by clicking on “Allocate Elastic IP”
4) Click on “Create NAT Gateway” to create the gateway

Creation of private route tables

1) Navigate to VPC -> Route Tables and click on Create Route table
2) Enter the name tag "Private Route Table", select the Project 1 VPC from the dropdown and click on Create
3) Once the route table is created, select it and select the Routes tab below the list of route tables
4) Click in Edit Routes and add the following route (Don't edit the existing one)
- Destination : 0.0.0.0/0
- Target: Select NAT Gateway and select the NAT Gateway created in the previous step
Click on Save Routes
5) Select the Subnet Associations tab and click on Edit Subnet Associations
6) Select the private Subnet from the list and click on Save

Step 3 : Creation of  database and application servers

1) Navigate to EC2 using the Services button at the top of the screen
2) Select Instances at the left side of the screen
3) Click on Launch Instance
- Select the AMI Amazon 2 Linux
- Select the instance type t2.micro
- Select Network as "Project 1 VPC" and subnet as "Public Subnet"
- For the security group, open the ports 80,443, 22 and 8065 for source set to "Anywhere"
4) Launch the instance after creating a new pem file and downloading it

Creation of database server

1) Navigate to EC2 using the Services button at the top of the screen
2) Select Instances at the left side of the screen
3) Click on Launch Instance
- Select the AMI Amazon 2 Linux
- Select the instance type t2.micro
- Select Network as "Project 1 VPC" and subnet as "Private Subnet"
- For the security group, open the ports 80, 443,22 and 3306 for source set to "Anywhere"
4) Launch the instance by selecting the same pem file created in the previous step

Step 4: Application and Database Installation and Testing

1) Copy the database pem file into the application server using the below command
scp -i <application server pem file> <database server pem file > ec2-user@<application server public IP>:/home/ec2-user
2) Log into the application server using SSH/Putty
3) From the application server, log into the database server using the pem file copied in step 1and the private IP address of the database server with the following command
ssh -i <database server pem file> ec2-user@<private IP of database server>
4) Enter the following commands to install and configure MySQL on the database server
sudo yum update
wget http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
sudo yum localinstall mysql57-community-release-el7-9.noarch.rpm -y
sudo yum install mysql-community-server -y
sudo systemctl start mysqld.service

Run the below command to retrieve a temporary password for MySQL
sudo grep 'temporary password' /var/log/mysqld.log | rev | cut -d" " -f1 | rev | tr -d "."

Log in to MySQL with the below command and enter the above password when prompted
mysql -u root -p

Enter the below command after you login to MySQL 
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password42!';

Type ‘exit’ into the MySQL prompt and press Enter to exit out of the MySQL environment.
Enter the below commands to complete the setup. Ignore any warning messages you receive.
wget https://d6opu47qoi4ee.cloudfront.net/install_mysql_linux.sh
chmod 777 install_mysql_linux.sh
sudo ./install_mysql_linux.sh
5) Type exit to exit the database server and go back to the application server

Installation and configuration of Mattermost

) Enter the following commands after logging into the application server via SSH to install and configure Mattermost

wget https://d6opu47qoi4ee.cloudfront.net/install_mattermost_linux.sh


sudo yum install dos2unix -y
sudo dos2unix install_mattermost_linux.sh

chmod 700 install_mattermost_linux.sh
sudo ./install_mattermost_linux.sh <private IP of MySQL server>
Example : sudo ./install_mattermost_linux 173.65.34.7
sudo chown -R mattermost:mattermost /opt/mattermost
sudo chmod -R g+w /opt/mattermost
cd /opt/mattermost
sudo -u mattermost ./bin/mattermost

2) Check whether the server has been successfully deployed by navigating to the following URL in your web browser. The web page might take a couple of minutes to load. 
<public IP of the application server>:8065
