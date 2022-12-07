![Texte
   alternatif](https://www.fullstackpython.com/img/logos/ansible-wide.png)

## This repository contains the projects and exercises that I was able to perform on Ansible during my training at Eazytraining

## :file_folder: TP5 - playbook :file_folder:

Here we are going to deploy an Apapche container using an Ansible playbook.
First we create our inventory file (hosts.yml) which will contain the group prod with the ip of our client machine (here 10.0.1.4).
Then we will create the group_vars folder which will include the prod.yml file.
This one regroups the connection informations (ansible_user and ansible_password).
We create our playbook named deploy.yml to deploy Apache with Docker on the host machine. We use the httpd image and expose port 80.
You will notice that we need some prerequisites before running this task: install EPEL repo, download pip script, install python-pip and Install docker python.
Moreover we need to create the ansible.cfg file with the variable "become_ask_pass = true" which allows to ask for the root password to run our playbook.


## :file_folder: TP6 -  templating, loop and condition   :file_folder:

In this exercise we will modify our previous playbook in order to implement the notions of templating, loop and condition.

Our playbook will use loops to install git, wget and epel-release if and only if we are on a CentOS system : 
```
pre_tasks:
  - name: Install some package
  package: name='{{ item }}' state=present
  when: ansible_distribution == "CentOS"
  loop:
    - epel-release
    - wget
    - git
  ```
  
Our site should display "Bienvenue sur  {{ ansible_hostname }}". 
For that we create the templates folder and the index.html.j2 file :

![Texte alternatif](https://raw.githubusercontent.com/vince0169/images_readme/main/Image2_Ansible.png?token=GHSAT0AAAAAAB35AY2U34S4U4QACKJ2FMXKY4QV2TA)


In the playbook we add the template module that copies the index.html.j2 file on our remote machine (in /home/admin/index.html) :
```
tasks:
  - name: Copy website file template
    template:
      src: index.html.j2
      dest: /home/admin/index.html
```

Finally,  you will notice that we add a volume at the end of the playbook for the index.html file : 
```
volumes:
  - /home/admin/index.html:/usr/local/apache2/htdocs/index.html
  ```


## :file_folder: TP7 -  security :file_folder:
In the previous exercises we defined our ansible_password variable in clear text in the group_vars/prod.yml file.
This is not a good practice but we can encrypt it.

To do this, we create the files/secret/credentials.vault file and move our ansible_password variable (we call it here vault_ansible_password) inside it :
![Texte
   alternatif](https://raw.githubusercontent.com/vince0169/images_readme/main/TP7_Ansible_Image1.png?token=GHSAT0AAAAAAB35AY2VB77D3DBEELWFMIHKY4QU2UA)
   
We encrypt this file with the ansible-vault encrypt command, we must then choose an mdp :
![Texte
   alternatif](https://raw.githubusercontent.com/vince0169/images_readme/main/TP7_Ansible_Image2.png?token=GHSAT0AAAAAAB35AY2VJR3GKAUHVBROFHM4Y4QU32Q)
   
We can see that the file is well encrypted by making a cat on it : 
![Texte
   alternatif](https://raw.githubusercontent.com/vince0169/images_readme/main/TP7_Ansible_Image3.png?token=GHSAT0AAAAAAB35AY2VY7W3R65ZSGVW74MSY4QU4ZA)
   
Then we specify the path of the file with the variable vars_files in our playbook : 
```
var_files:
  - files/secret/credentials.vault
```
  
Finally we can execute ansible-playbook command by specifying --ask-vault-pass in the command. We have then the mdp BECOME and Vault requested : 
![Texte
   alternatif](https://raw.githubusercontent.com/vince0169/images_readme/main/TP7_Ansible_Image4.png?token=GHSAT0AAAAAAB35AY2VE7NDRRIO5SEQNCEIY4QVA3Q)



## :file_folder: TP8 - role :file_folder:

In this exercise we are going to create a wordpress.yml playbook in order to deploy wordpress using an ansible role (available here : https://github.com/diranetafen/ansible-role-containerized-wordpress.git)

First we create the roles/requirements.yml file in which we specify the source of our role : 
````
# Install a role for wordpress
- src: https://github.com/diranetafen/ansible-role-containerized-wordpress.git
````
Then we install our role : 
![Texte
   alternatif](https://raw.githubusercontent.com/vince0169/images_readme/main/TP8_Ansible_Image1.png?token=GHSAT0AAAAAAB35AY2UOEAFTWURGBKSI5LMY4QVIAA)
   
Finally we create our playbook with our role and execute the ansible-playbook command: 
```
---
- hosts: prod
become: true
vars:
  system_user: admin
vars_files:
  - files/secrets/credentials.vault
pre_tasks:
  - name: create www-data
    user: name=www-data state=present
roles:
  - { role: ansible-role-containerized-wordpress }
   ```



## :file_folder: Mini-projet :file_folder:


In this project we're going to create our own ansible role in order to deploy our deploy.yml playbook.
As a reminder, this playbook allows you to deploy an Apache container using docker on the host machine and to display "Bienvenue sur {{ ansible_hostname }}"

My role is located at the following address: https://github.com/vince0169/ansible-role-containerized-apache.git

In this repository you can find the defaults/main.yml file.
It contains default variables for the role (tasks/main.yml). These variables have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

The meta/main.yml file contains metadata for the role, including role dependencies and optional Galaxy metadata such as platforms supported.

In the tasks repository you can find the main.yml file which is the main list of tasks that the role executes.
Also you can find 3 others files (download_pip_script.yml, install_packages.yml and install_python_pip.yml) which are pretasks of the playbook.

We invoke  pretasks in tasks/mains.yml by this way : 
````
- include_tasks: "install_packages.yml"
- include_tasks: "download_pip_script.yml"
- include_tasks: "install_python_pip.yml"
````

The templates/main.yml file involves templates that the role deploys.

To finish, tests/test.yml file is an example to show us how to consume our role.
