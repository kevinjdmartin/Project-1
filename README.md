# Project-1
13-Elk-Stack-Project
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.
--
![Diagram Project Week 13](https://user-images.githubusercontent.com/98208819/167271559-6b6a27af-afef-4334-909d-c5ce1a16f755.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

[pentest.yml.txt](https://github.com/kevinjdmartin/Project-1/files/8645935/pentest.yml.txt)
--
[elk-playbook.yml.txt](https://github.com/kevinjdmartin/Project-1/files/8645946/elk-playbook.yml.txt)
--
[filebeat-playbook.yml.txt](https://github.com/kevinjdmartin/Project-1/files/8645947/filebeat-playbook.yml.txt)
--
[metricbeat-playbook.yml.txt](https://github.com/kevinjdmartin/Project-1/files/8645948/metricbeat-playbook.yml.txt)
--


Playbook File : pentest.yml
 ---
       name: Config Web VM with Docker
       hosts: webservers
       become: true
       tasks:
       - name: docker.io
       apt:
       force_apt_get: yes
       update_cache: yes
       name: docker.io
       state: present

       - name: Install pip3
       apt:
       force_apt_get: yes
       name: python3-pip
       state: present

       - name: Install Docker python module
       pip:
       name: docker
       state: present

       - name: download and launch a docker web container
       docker_container:
       name: dvwa
       image: cyberxsecurity/dvwa
       state: started
       published_ports: 80:80

       - name: Enable docker service
       systemd:
       name: docker
       enabled: yes
      
Playbook File : Elk-playbook.yml      
 ---
       - name: Configure Elk VM with Docker
       hosts: elk
       remote_user: azureuser
       become: true
       tasks:
       # Use apt module
       - name: Install docker.io
       apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use shell module
    - name: Increase virtual memory on restart
      shell: echo "vm.max_map_count=262144" >> /etc/sysctl.conf
      
Playbook File : Filebeat-playbook.yml
---
     - name: installing and launching filebeat
     hosts: webservers
     become: yes
     tasks:

      - name: download filebeat deb
     command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    - name: enable and configure system module
    command: filebeat modules enable system
    - name: Setup filebeat
    command: filebeat setup

    - name: Start filebeat service
    command: service filebeat start

    - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

Playbook File : Metricbeat-playbook.yml     
---
           - name: Install metric beat
           hosts: webservers
           become: true
           tasks:

           - name: download metricbeat
           command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

           - name: install metricbeat
           command: dpkg -i metricbeat-7.4.0-amd64.deb

           - name: drop in metricbeat config
           copy:
           src: /etc/ansible/metricbeat-config.yml
           dest: /etc/metricbeat/metricbeat.yml

           - name: enable and configure docker module for metric beat
           command: metricbeat modules enable system

           - name: Setup metric beat
           command: metricbeat setup

           - name: Start metric beat
           command: service metricbeat start

           - name: enable service metricbeat on boot
           systemd:
           name: metricbeat
           enabled: yes




This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly capable, in addition to restricting access to the network.
- What aspect of security do load balancers protect?
   >Mitigates DoS Attacks
 
- What is the advantage of a jump box?
   >Install, update, monitor VM Servers/Webservers and increase web security

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the network traffic and system files.
- What does Filebeat watch for?
  > Collects data to the file system
- What does Metricbeat record?
  > Collects data metrics mesures how the system is improving or slowing. analyzing how healthy system is

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.1   | Linux (ubuntu 20.04) |
| Web-1    | Web Server | 10.1.0.8 | Linux (ubuntu 20.04) |
| Web-2    | Web Server | 10.1.0.9 | Linux (ubuntu 20.04) |
| Web-3    | Web Server | 10.1.0.10 | Linux (ubuntu 20.04)|
| Elk-1    | Monitoring | 10.0.0.4 | Linux (ubuntu 20.04) |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- Add whitelisted IP addresses: 172.114.97.49


Machines within the network can only be accessed by Jump Box.
- Which machine did you allow to access your ELK VM? What was its IP address?
  IP: 10.0.0.1 allow jumpbox private IP go through port 22

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes (SSHPORT22      | 172.114.97.49        |
| Web-1    | Yes (HTTPPORT80)    | 172.114.97.49        |
| Web-2    | Yes (HTTPPORT80)    | 172.114.97.49        |
| Web-3    | Yes (HTTPPORT80)    | 172.114.97.49        |
| Elk-1    | Yes (HTTPPORT5601)  | 172.114.97.49        |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- > Ansible open-source tool
- > System installation and update can be streamlined
- > processes become more replicale
- > Simple set up no coding skills are necessary to use

The playbook implements the following tasks:
- > Use Ansible Play-book to install the following:
- > Docker.io & Python3-pip
- > Docker Module
- > Sysctl -w vm.max_map_count=262144
- > increase virtuak memory on restart
- > Ansible Playbook Code:
  
      name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      Use apt module
      name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      Use pip module (It will default to pip3)
      name: Install Docker module
      pip:
        name: docker
        state: present

      Use command module
      name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      Use shell module
      name: Increase virtual memory on restart
      shell: echo "vm.max_map_count=262144" >> /etc/sysctl.conf
  
  

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![Ansible Container](https://user-images.githubusercontent.com/98208819/167273677-c3cf6730-40c6-4915-a3ba-7956b654e9db.png)
---

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- > Web-1: 10.1.0.8
- > Web-2: 10.1.0.9
- > Web-3: 10.1.0.10

We have installed the following Beats on these machines:
 - > Filebeat
 - > Matricbeat

These Beats allow us to collect the following information from each machine:
- > Filebeat collects data log events, is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the     log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- > Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash. Metricbeat helps you     monitor your servers by collecting metrics from the system and services running on the server, such as: Apache. HAProxy.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- > Copy the configuration file to ansible container.
- > Update the configuration file to include Elk server's IP address 10.0.0.4 in the file.
- Run the playbook, and navigate to http://Elk-server-ip:5601/app/kibana#/home to check that the installation worked as expected.

Answer the following questions to fill in the blanks:
- Which file is the playbook?
 > Filebeat-playbook.yml & Metricbeat-playbook.yml
-  Where do you copy it?
 > /etc/ansible/[Playbook.yml]

- Which file do you update to make Ansible run the playbook on a specific machine?
 > Filebeat-config.yml & Metricbeat-config.yml
-  How do I specify which machine to install the ELK server on versus which to install Filebeat on?
             
              output.elasticsearch:
              hosts: ["10.0.0.4:9200"]
              username: "elastic"
              password: "changeme"
    
              ...
    
             setup.kibana:
             host: "10.0.0.4:5601"
             
- Which URL do you navigate to in order to check that the ELK server is running?
 > http://20.216.19.254:5601/app/kibana#/home

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

  To run and download playbook we need to install and update ansible containers
  Start by installing docker.io
  
           sudo apt update
           sudo apt install docker.oi
  Verify if the Docker service is running
  
           sudo systemctl status docker
  When docker is installed, pull the container we using to complete the task.
  
           sudo docker pull cybersecurity/ansible
  Now we going to check if we have ansible container.
  
           sudo docker ps -a
  We going to run the ansible container by this command
  
          sudo docker start [ansible container name]
          sudo dockat attach [ansible container name]
  Root host: now we going to create yml  and config yml to rush the playbook in /etc/ansible/[PLAYBOOK.yml] and /etc/ansible/[Confic.yml]
  To run play book   : ansible-playbook /etc/ansible/[Sameple.yml]
 
 ![Ansible ](https://user-images.githubusercontent.com/98208819/167275492-27285a9c-ed71-4591-9367-283322014f35.png)
  --- 
  
