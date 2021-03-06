# CyberSyndex
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

<img width="738" alt="CloudVirtFinalKrebs" src="https://user-images.githubusercontent.com/46389908/120214504-6a4e7700-c1e9-11eb-9c8f-c821847a0a25.png">

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select files such as filebeat-playbook.yml may be used to install only certain pieces of it, such as Filebeat.

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly stable, in addition to restricting access to the network.  Load balancers in particular protect against distributed denial-of-service (DDoS) attacks by shifting attack traffic from a corporate server to the public cloud provider.  Both software and hardware load balancers ensure high availability and reliability by sending requests only to servers that are online.

A jump box (or bastion host), which acts as a protective, auditable segregation layer between the target network and the user, provides the point of contact to the DVWA (containerized onto three web virtual machines).  

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the log content and system metrics.

Filebeat watches for new log content and creates harvesters to collect this data, which are then shipped to the designated output. 

Metricbeat is another lightweight shipper that periodically collects metrics from the perating system and services running on the server such as MongoDB, Nginx, etc. and ships these (like Filebeat) to either Elasticsearch or Logstash.

The configuration details of each machine may be found below.
| Name     | Function                                 | IP Address        | Operating System |
|----------|------------------------------------------|-------------------|------------------|
| Jump Box | Gateway                                  | 52.188.70.215     | Linux            |
| Web-1    | first DVWA                               | 10.0.0.5          | Linux            |
| Web-2    | second DVWA                              | 10.0.0.6          | Linux            |
| Web-3    | third DVWA                               | 10.0.0.7          | Linux            |
| ELK VM   | containerized app to access ELK stack    | 10.1.0.4          | Linux            |
| Kibana   | Visual app to analyze Elasticsearch data | 104.45.89.98:5601 | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 205.185.223.91

Machines within the network can only be accessed by the Jump Box.  Only the Jump Box can access the ELK VM (Jump Box IP address noted above)
- 52.188.70.215

A summary of the access policies in place can be found in the table below.

|   Name   | Publicly Accessible | Allowed IP Addresses |
|:--------:|:-------------------:|:--------------------:|
| Jump Box |         Yes         |     52.188.70.215    |
|  ELK VM  |          No         |       10.1.0.4       |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it is much more efficient to set up multiple machines this way.  This method helps prevent error as well if each were to be set up manually.

The playbook implements the following tasks:

- Install Docker and verify that it is running
- Installs the Python3 pip installation module to help with Docker installation
- Increases virtual memory using the sysctl -w command and increases VM swappiness
- Finally it downloads and starts the ELK container running on ports 5044, 5601, and 9200

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

<img width="1388" alt="docker-ps-output" src="https://user-images.githubusercontent.com/46389908/120214589-881bdc00-c1e9-11eb-9aaf-aa2979f78c87.png">

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1
- Web-2
- Web-3

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat: collects system logs, which are used to track activity generated by the user acting against the 3 web VMs using the D*mn Vulnerable Web Application tool (http://138.91.125.79/index.php)
- Metricbeat: collects health and efficiency data from the web servers and reports back to Kibana

In Kibana GUI:
To set up Filebeat:
index page —> "add logs" —> "system logs" —> “add data” (final button, to check if receiving data)
To set up Metricbeat:
index page —> "add metrics" —> "docker metrics" —> “add data” (final button, to check if receiving data)

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the  file to /etc/ansible/roles.
- Update the install-elk.yml file to include entry for "hosts" is "elk" or "elkservers", use "update_cache" instead of "force_apt_get" under apt module, flesh out "use more memory" (sysctl module)task as name: vm.swappiness, and list ports as 5601, 9200, and 5044 (on which ELK will run))
- Run the playbook, and navigate to http://<public ELK server IP>:5601/app/kibana to check that the installation worked as expected and that the ELK server is running.

Essentially, there are four yaml playbook files (pentest.yml, install-elk.yml, filebeat-playbook.yml, and metricbeat-playbook.yml all stored in /etc/ansible/roles) that respectively install Docker and dvwas on 3 web servers, install elk on elk VM, and install filebeat and metricbeat on web servers.  The latter two playbooks use drop-in configuration files that are stored in /etc/ansible/files.  There is a fifth posible playbook file that could be created called "main.yml" that can contain each of the roles above, broken down into respective hosts.  It is this file that would be edited to make Ansible run the playbook on a specific mahcine.  Moreover, to specify which machine to install the ELK server on, this too in the main.yml file (or if not using this main file, then in the )

Commands needed to perform key actions in the ELK server setup:

externalMachine$ curl -4 icanhazip.com
externalMachine ~/.ssh$ ssh-copy-id <jumpboxusername>@<jumpbox external IP>
(or in RedTeam Azure GUI go to network settings and add public key)
externalMachine$ ssh <jumpboxusername>@<jumpbox external IP>
JumpBoxProvisioner$ sudo apt install docker.io
JumpBoxProvisioner$ systemctl service docker
JumpBoxProvisioner$ sudo docker pull cyberxsecurity/ansible
(can run $ sudo su or $ sudo -i to change to root after entering jump box)
JumpBoxProvisioner$ sudo docker run -ti cyberxsecurity/ansible:latest bash
(create inbound RedTeam NSG rule allowing SSH (port 22) from jump box to virtual network)
JumpBoxProvisioner$ sudo docker images ls
JumpBoxProvisioner$ sudo docker run -it cyberxsecurity/ansible /bin/bash

(** note: for subsequent entry into ansible container use:
JumpBoxProvisioner$ sudo docker container list -a
JumpBoxProvisioner$ sudo docker start <container name> (will be something like "clever_davinci")
JumpBoxProvisioner$ sudo docker attach <container name>
)

root@ansible# ssh-keygen && ls ~/.ssh && cat .ssh/id_rsa.pub (then copy to all 3 web VMs authorized keys list)
root@ansible# ls /etc/ansible && vi hosts (add under [webservers] heading:)
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3
10.0.0.7 ansible_python_interpreter=/usr/bin/python3
root@ansible# vi /etc/ansible/ansible.cfg (uncomment "remote_user" and hadd username for web VMs in place of "root")
root@ansible# ansible -m ping all (to make sure public keys and python interperter are working and that web VMs recognize user name)

create and run playbook /etc/ansible/pentest.yml (see above)
root@ansible# ansible-playbook pentest.yml
root@ansible# ssh ansible@10.0.0.5 (test whether dvwa is working on each of three web VMs)
(make sure that RedTeam NSG is not blocking all connections but allows port 80 connection from external machine)
ansible@Web-1$ curl localhost/setup.php (should get html text) (to further test connection)
ansible@Web-1$ exit
(set up ELK VM and ELK NSG in Azure, allow NSGs to communicate with one another)
root@ansible# vi /etc/ansible/install-elk.yml (see above for alterations/additions)

if for some reason install-elk.yml does not work can manually install ELK with:
root@ansible# docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk:761

root@ansible# ansible-playbook filebeat-playbook.yml
root@ansible# ansible-playbook metricbeat-playbook.yml
