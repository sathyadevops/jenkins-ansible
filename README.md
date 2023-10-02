# jenkins-ansible

                                                   Jenkins-Ansible Integration
                                                ----------------------------------------

Step-1:  Create Two Servers :
--------------------------------------------
                      1 - Jenkins and Ansible Master
                      1 - Ansible Node

Step-2: Jenkins Installation :
-------------------------------------------
           # sudo su -
           # apt-get update      
           # apt-get install openjdk-11-jdk  -y	
           # apt-get install tomcat9 -y
           # apt-get install git-core -y
           # apt-get install ant  -y
           # apt-get install maven  -y

to change tomcat port : 
----------------------------------
   #vi /etc/tomcat9/server.xml
   <Connector port="9090"

   # service tomcat9 restart


Google ---> jenkins download

Step-3: Download Jenkins :
----------------------------------------
           #  curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

  # echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
              
 Install Jenkins : 
--------------------------
           # apt-get update
           # apt-get install fontconfig openjdk-11-jre
           # apt-get install jenkins  -y

To Set paths:
=============
    Jenkins  --> select "Manage Jenkins" --> "Global tool configuration"

   # Java path  :  /usr/lib/jvm/java-11-openjdk-amd64
   # Git path :  /usr/bin/git
   # ANT Path:   /usr/share/ant/
   # Maven Path:  /usr/share/maven/

  Jenkins user Permissions:
 --------------------------------------
   # apt-get install acl -y
   # setfacl -m u:jenkins:rwx /opt
   # setfacl -m u:jenkins:rwx /var/lib/tomcat9/webapps
   # getfacl /opt 
   # getfacl /var/lib/tomcat9/webapps/ 

Step-4: Install ansible on Master:
------------------------------------------------
                 # apt-get update
                 # apt-get install software-properties-common -y
                 # apt-add-reposotiry ppa:ansibe/ansible
                 # apt-get update
                 # apt-get install ansible -y
                 # ansible -version

   # vi /etc/ansible/hosts
   [node]
      172.31.36.172
  
                   # ansible all -m ping
172.31.36.172 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}




Step-5: Configure SSH for Ansible-Master and Nodes
-------------------------------------------------------------------------------------------

 Step-a:  to change root password : 
------------------------------------------------
               node# passwd  root
       Enter new passwd :  root

 Step-b:  to configure ssh: 
------------------------------------
               node# vi /etc/ssh/sshd_config
        [press 'i' for insert mode]
	PermitRootLogin   yes
	PasswordAuthentication   yes
        [press 'Esc']
        [:wq  ---> write and quit]

 Step-c: to restart ssh service:
 -----------------------------------------
              node# service ssh restart         (for Ubuntu nodes)
                                  [OR]
              node# systemctl restart sshd   (for Redhat nodes)

 Step-d: to Connect with Nodes:
----------------------------------------------
              node# hostname  -i         (to get an  Private IP-Address)
             [OR]  # ifconfig      
             [OR]  # ip  a         

              node# curl ifconfig.me     (to get an  Public IP-Address)
               [OR]   curl ifconfig.in

              node# hostname  -f         (to get a Host Name / FQDN)
  
              mstr#  ssh  <node-IP>



                                Password less Authentication  (SSH-Keys)
                                -------------------------------------------------------------
 Step-x:  to generate a Key-pair
------------------------------------------
             mstr#  ls  -a          (to list hidden files)
             mstr#  cd .ssh   
             mstr#  ssh-keygen
             mstr#  ls
             id_rsa          (private key)
             id_rsa.pub   (public key)

 Step-y:  to send public key to nodes
----------------------------------------------------
          mstr#  ssh-copy-id   <node IP Address>

 Step-z: to Connect with Nodes:
----------------------------------------------
              mstr#  ssh  <node IP>




  
Step-6: Install and Configure Asible Plugin :
----------------------------------------------------------------
    Jenkins  ---> Manage Jenkins ---> "Plugins"  ---> Available --> 
             search for "Ansible Plugin" --> install without restart

    Jenkins  ---> Manage Jenkins  ---> "tools" --> select "Ansible"
          Name:   Ansible
          Path to ansible executables directory:  /usr/bin


Step-7: define playbooks
------------------------------------
        # mkdir playbooks
        # vi pb1.yml

---
- hosts: all
  gather_facts: true
  become: true
  tasks:
  - name: to install apache
    apt: name=apache2 update_cache=yes state=present
    changed_when: true

  - name: to deploy file
    copy:
      src: /opt/index.html
      dest: /var/www/html/index.html
    changed_when: true

  - name: to start service
    service: name=apache2 state=started
    changed_when: true


   # cd /opt
   # ls -l playbooks/
   # chown -R jenkins:jenkins playbooks/
   # cd /var/lib/jenkins
   # mkdir .ssh
   # cp ~/.ssh/* .ssh
   # chown -R jenkins:jenkins .ssh
  
Step-8: Create Jenkins Project :
------------------------------------------------
   Jenkins  ---> New Item --> "Free style project"

    Build steps --> select "Invoke Ansible Playbook"  --> 	
 
    Ansible installation: Ansible
    Playbook path: /root/playbooks/pb1.yml
    Inventory: File or host list  --> /etc/ansible/hosts
    Host subset:  web
    Credentials:  xxxxx


  

Started by user admin
Running as SYSTEM
Building on the built-in node in workspace /var/lib/jenkins/workspace/JENKINS-ANSIBLE-PROJECT
[JENKINS-ANSIBLE-PROJECT] $ /usr/bin/ansible-playbook /opt/pb1.yml -i /etc/ansible/hosts -l web -f 5 --private-key /tmp/ssh15539247242160740393.key -u root

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.31.7.40]

TASK [to install apache] *******************************************************
changed: [172.31.7.40]

TASK [to deploy file] **********************************************************
changed: [172.31.7.40]

TASK [to start service] ********************************************************
changed: [172.31.7.40]

PLAY RECAP *********************************************************************
172.31.7.40                : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Finished: SUCCESS
