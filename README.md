# aws-ansible
Quick and dirty Ansible playbook and files for deploying to AWS EC2 Instance

# The Task:
+ Using the AWS console, build a tiny instance in the sandbox
+ Using that hosts IP, deploy Nginx using Ansible

# Notes

Create AWS account  
Create instance using Amazon’s guide using an AMI
Save PEM file  
Since I'm on Windows, seperately provision VM (Centos 7) and install Ansible  
Add 2 x security rules (1 for my Windows PC, 1 for Centos VM)  
Transfer PEM file to VM  
Create inventory config (inventory.cfg)  
Create nginx_install.yml (remembering to become AWS user).  Note - original YAML file used ***apt-get***  
Run `ansible-playbook -i inventory.cfg nginx_install.yml -b`  
Result:  

`PLAY [all] *********************************************************************`
`ASK [Gathering Facts] *********************************************************`
`fatal: [$servername]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host $servername port 22: Connection timed out\r\n", "unreachable": true}`
       `to retry, use: --limit @/usr/bin/nginx_install.retry`
 `PLAY RECAP *********************************************************************`
`$servername             : ok=0    changed=0    unreachable=1    failed=0` 


After some googling, I think I need to "Use the chmod command (in bold below) to make sure your private key file isn’t publicly viewable. Please see commonly asked questions section below if you have issues and are using windows."  
**chmod 400 /path_to_key/my_key.pem**  
Now that's done, try Ansible Playbook again.  Now we fail with the Python module:


`TASK [Gathering Facts] *********************************************************`
`fatal: [$servername]: FAILED! => {"changed": false, "module_stderr": "Shared connection to $servername closed.\r\n", "module_stdout": "/bin/sh: /usr/bin/python3: No such file or directory\r\n", "msg": "MODULE FAILURE", "rc": 127}`
      ` to retry, use: --limit @/usr/bin/nginx_install.retry`
 `PLAY RECAP *********************************************************************`
`$servername            : ok=0    changed=0    unreachable=0    failed=1`  


OK, been a bit naughty here – after this failure, I installed Python 3 by manually going to AWS box and installing it  
Still failing after this  
After some Googling – realise, no ‘apt-get’ on Amazon Linux!
Added this line to the YAML file:  


`shell: sudo amazon-linux-extras install nginx1.12` 


(Which is also a bit naughty)
SUCCESS:

`PLAY [all] ***********************************************************************************************`
 
`TASK [Gathering Facts] ***********************************************************************************`
`ok: [$servername]`
 
`TASK [ensure nginx is at the latest version] *************************************************************`
 `[WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo`
 
`changed: [$servername]`
 
`TASK [start nginx] ***************************************************************************************`
`changed: [$servername]`
 
`PLAY RECAP ***********************************************************************************************`
`$servername            : ok=3    changed=2    unreachable=0    failed=0   `


However, when trying to access $servername:80, could not  
Had to add a new security rule to allow http traffic on port 80  
***SUCCESS!!!***


# Next Steps  
Refactor YAML file to use yum
