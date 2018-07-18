# Ansible-Dynamic-Inventory-EC2
Dynamic inventory for Ansible hosts when working with AWS Cloud

# Purpose

The /etc/ansible/hosts is a static repo of the hosts managed by Ansible but is not good for cloud deployments where the inventory hosts changes regularly so dynamic inventory is used

Resource
 https://aws.amazon.com/blogs/apn/getting-started-with-ansible-and-dynamic-amazon-ec2-inventory-management/

# Pre-requisites 

-	Ansible Master
-	Ansible Slave/Client in my case it is an AWS EC2 Instance 

# Steps
-	Update the environment variable set it temporary and then make permanent in .profile set environment  variable 
-	$ export AWS_ACCESS_KEY_ID='YOUR_AWS_API_KEY' 
- $ export AWS_SECRET_ACCESS_KEY='YOUR_AWS_API_SECRET_KEY'
    make it permanent in .profile
-	Download the boto and python script

ansibleadmin@lpgpsap1014:/etc/ansible> wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py --2018-07-17 12:20:57--https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.16.133 Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.16.133|:443... connected. HTTP request sent, awaiting response... 200 OK Length: 72975 (71K) [text/plain] Saving to: ‘ec2.py’
100%[============================================================================================================================================>] 72,975 --.-K/s in 0.03s
2018-07-17 12:20:57 (2.78 MB/s) - ‘ec2.py’ saved [72975/72975]
ansibleadmin@lpgpsap1014:/etc/ansible>

wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini --2018-07-17 12:21:22--https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.16.133 Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.16.133|:443... connected. HTTP request sent, awaiting response... 200 OK Length: 9529 (9.3K) [text/plain] Saving to: ‘ec2.ini’
100%[============================================================================================================================================>] 9,529 --.-K/s in 0s
2018-07-17 12:21:22 (134 MB/s) - ‘ec2.ini’ saved [9529/9529]
- Setting up the Inventory
-rw-r--r-- 1 ansibleadmin ansible 72975 Jul 17 12:20 ec2.py -rw-r--r-- 1 ansibleadmin ansible 9529 Jul 17 12:21 ec2.ini drwxr-xr-x 4 ansibleadmin ansible 4096 Jul 17 12:21 . ansibleadmin@lpgpsap1014:/etc/ansible> chmod +x ec2.py ec2.ini

- Setting Environment Variables

  ansibleadmin@lpgpsap1014:/etc/ansible> export     ANSIBLE_HOSTS=/etc/ansible/ec2.py ansibleadmin@lpgpsap1014:/etc/ansible> export EC2_INI_PATH=/etc/ansible/ec2.ini

- Move the .pem file to Ansible Master server using SCP to the ansible master .ssh folder
-rwxr-xr-x 1 ansibleadmin ansible 1696 Jul 17 12:42 XXXXXX.pem drwx------ 2 ansibleadmin ansible 4096 Jul 17 12:42 . ansibleadmin@lpgpsap1014:~/.ssh> chmod 400 XXXXX.pem
$ ssh-agent bash $ ansibleadmin@lpgpsap1014:~/.ssh> ssh-add ~/.ssh/XXXXXX.pem Identity added: /home/ansibleadmin/.ssh/XXXXX.pem (/home/ansibleadmin/.ssh/XXXXX.pem

•	Make sure port 22 is open between the servers make sure ssh is working as ssh ec2-user@XXXX

•	For the ERROR: "Forbidden", while: getting ElastiCache clusters
    Set in the ec2.ini file the following below value 
    To exclude ElastiCache instances from the inventory, uncomment and set to False.
    elasticache = False This will fix the above issue


•	For the ERROR: "Failed to connect to the host via ssh: Permission denied (publickey).\r\n"
    set remote_user=ec2-user in the ansible.cfg file 
    ansibleadmin@lpgpsap1014:/etc/ansible> vi ansible.cfg 
    
•	 Ping the tag of the newly created Instance ot the Ansible Slave in my case the tag was Name:Ansible Created instance 
     it changes to format tag_Name_Ansible_Created_Instance in the command below

ansibleadmin@lpgpsap1014:/etc/ansible> ansible -m ping tag_Name_Ansible_Created_Instance
[DEPRECATION WARNING]: ANSIBLE_HOSTS option, The variable is misleading as it can be a list of hosts and/or paths to inventory sources , use ANSIBLE_INVENTORY instead. This feature will be removed in version 2.8. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg. 34.245.73.144 | SUCCESS => { "changed": false, "ping": "pong"


•	Helpful command 
 
 "ansibleadmin@lpgpsap1014:/etc/ansible> ansible-config dump | grep HOST "
 
 [DEPRECATION WARNING]: ANSIBLE_HOSTS option, The variable is misleading as it can be a list of hosts and/or paths to inventory sources , use ANSIBLE_INVENTORY instead. This feature will be removed in version 2.8. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg. DEFAULT_HOST_LIST(env: ANSIBLE_HOSTS) = [u'/etc/ansible/ec2.py'] DISPLAY_SKIPPED_HOSTS(default) = True HOST_KEY_CHECKING(/etc/ansible/ansible.cfg) = False LOCALHOST_WARNING(default) = True PARAMIKO_HOST_KEY_AUTO_ADD(default) = False

# Final Check

ansibleadmin@lpgpsap1014:/etc/ansible> /etc/ansible/ec2.py --list 

{ "_meta": { "hostvars": { "34.245.73.144": { "ansible_host": "34.245.73.144", "ec2__in_monitoring_element": false, "ec2_account_id": "462621403766", "ec2_ami_launch_index": "0", "ec2_architecture": "x86_64", "ec2_block_devices": { "sda1": "vol-0b4018c3dc3ea7c92" }, "ec2_client_token": "", "52.30.76.143" ], "tag_Name_Ansible_Created_Instance": [ "34.245.73.144" ], "tag_Name_Ansible_Server_Master_Host": [ "52.30.76.143" ], "type_t2_micro": [ "34.245.73.144", "52.30.76.143" ], "vpc_id_vpc_32569d55": [

# Result 

All newly added EC2 Instances through Ansible in AWS will be displayed by ec2.py –list 
If not visible try the command ec2.py --refresh-cache
