# Devops-project-4

- In this project i made a ci/cd pipeline using jenkins and deployed a web app to apache server using ansible.

- <b> Tools Used : </b>
- Terraform
- Git
- Jenkins
- Ansible
- Aws Ec2


- <b> Work Flow </b>

![devops-project-4 drawio (1)](https://user-images.githubusercontent.com/53990452/179393362-080f0d11-d314-4c58-9ae4-fa5108b89d16.png)


Steps Covered :
1) Setting up Infrastructure on AWS with terraform

- Creating 2 aws linux instance with security group.
- Code : main.tf
```
provider "aws" {
    region = "ap-south-1"
    profile = "Saumil1"
}

variable "amiid_aws-linux" {
  default = "ami-052cef05d01020f1d"
}


resource "aws_security_group" "terraform-prac" {
  name        = "terraform-prac"
  description = "All_traffic_tf"
  vpc_id      = "vpc-e970be82"

  ingress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_tls"
  }
}


resource "aws_instance" "Terraform-instance-1" {
  ami           = "${var.amiid_aws-linux}"
  instance_type = "t2.micro"
  key_name = "terraform-prac"
  vpc_security_group_ids =  ["${aws_security_group.terraform-prac.id}"]
  associate_public_ip_address = true
  tags = {
    Name = "Terraform-instance-1"
    
  }
}


resource "aws_instance" "Terraform-instance-2" {
  ami           = "${var.amiid_aws-linux}"
  instance_type = "t2.micro"
  key_name = "terraform-prac"
  vpc_security_group_ids =  ["${aws_security_group.terraform-prac.id}"]
  associate_public_ip_address = true
  tags = {
    Name = "Terraform-webserver-1"
    
  }
}


```

2) Setup Jenkins for ansible
-  We need to first download a plugin named "Ansible" and install it over our jenkins server, which can be done by going to jenkins sever -> manage jenkins -> manage plugins -> search and install the plugin

- To Configure it go to manage jenkins ->  global tool config -> scroll to ansible section -> Give a name and path where ansible is installed usually ("/usr/bin")

3) Setting up Github WebHooks :
- for the example we are using git repo : https://github.com/Saumil343/test1.git
- Go to settings of your respective repo
- Go to webhooks
- Add web Hook
- And in the payload section add: http://public_ip_of_jenkins_Server:8080/github-webhook/
- Content Type : application/json
- Select : Push Events
- And click create webhook
- EXAMPLE: 
- ![image](https://user-images.githubusercontent.com/53990452/178097731-8ffe3e39-ae38-40f2-adce-e5e429ad50a0.png)

4) Creating Continious Integration Job
- Go to Dashboard
- create new item and give it a name and select Free Style Project
- Go to general section and scroll to Source Code Management Section and select git and add your git repo link of the project to be deployed.
- EXAMPLE
![image](https://user-images.githubusercontent.com/53990452/177819185-2e169902-5f0d-43a5-887d-02f13759d2cc.png)
- Scroll and add build triggers and select GitHub hook trigger for GITScm polling.
- EXAMPLE
- ![image](https://user-images.githubusercontent.com/53990452/177819421-1e0c9080-b845-4091-94a4-a89b54e378b6.png)
- Scroll and add POST BUILD ACTION and give a name of Continious deployment job[yet to be created]
- Example
- ![image](https://user-images.githubusercontent.com/53990452/177819619-145f1db2-6565-4aff-a975-5b506e735787.png)

5) Creating a deployment Pipeline
-  For this example to run we need to have a git repo with the required files that are 1)ansible Playbook 2) Inverntory to run, in the git repo of our project to be deployed 
- Now go to jenkins dashboard 
- and create a new item -> give it a name and select pipeline
- scroll to bottom and create a pipeline 
- for the ease we can use the syntax generator like: 
- first step in the pipeline would be accessing the git repo :
- commands to which can be generated by the syntax generator like: 
![image](https://user-images.githubusercontent.com/53990452/178097873-9fe96b76-7374-4e1b-bfe4-b8d981dc7b8c.png)

- once this is done we can copy that to our pipeline code
- second stage in the pipeline would be to select and run the ansible playbook on our host machine
- Again go to syntax generator and select ansible invoke playbook to generate it's script 
-  Fill the details there, like the username,password of the machine, name of our playbook file (whcih is in git repo),Name/path of our inventory file(whcih is in our git repo) which can be seen as follows: 
-  ![image](https://user-images.githubusercontent.com/53990452/178098054-cd14999a-b12b-4da2-b348-85b833559297.png)
- add ssh credentials as per you system (give the credential of the system which is specified in ansible inverntory/hosts file):
- EXAMPLE
- ![image](https://user-images.githubusercontent.com/53990452/178098072-bf9b4636-ef48-4e17-bf9b-39e8b8fb5819.png)

- once all this is done generate the script and copy it : 
- EXAMPLE: 
![image](https://user-images.githubusercontent.com/53990452/178098099-eb6b63a5-d7ba-497c-b4f1-078f9b7e948d.png)

- Now get back to the pipeline we created and we will add steps we need it to follow :
- Complete Pipeline Code : (change the generated scripts in your case as per the req) 

 ```
 pipeline{
    agent any
    stages{
        stage("git checks"){
            steps{
                git branch: 'main', url: 'https://github.com/Saumil343/test1.git'
            }
        }
        stage("Ansible playbook"){
            steps{
                ansiblePlaybook credentialsId: '91fda46f-c50f-40ce-bab7-8a965994c32b', disableHostKeyChecking: true, installation: 'ansible1', inventory: 'hosts', playbook: 'second.yml'
            }
        }
    }
}
 ```
 - ANSIBLE FILE CODE: 
 ```
 ---
- hosts: gp1
  tasks:
    - name: Install Jenkins
      shell: sudo yum install jenkins -y

    - name: start jenkins
      shell: sudo systemctl start jenkins

    - name: Installing git
      shell: sudo yum install git -y

    - name: Installing Apache
      shell: sudo yum install httpd -y
    
    - name: start apache
      shell: sudo systemctl start httpd

    - name: Cleaning Webserver
      shell: rm -rf /var/www/html/*

    - name: cleaning files
      shell: rm -rf ~/test1

    - name: git pull
      shell: git clone https://github.com/Saumil343/test1.git

    - name: git copy
      shell: cp -r ~/test1 /var/www/html/
      
    - name: service restart
      service:
        name: httpd
        state: restarted

      
  
 ```
 6) Trigger a build
 -  now as soon as someting has changed in our git repo the change will be detected by our first pipeline which would trigger the ansible conitinious deployment pipeline which would run the playbooks
 
 
