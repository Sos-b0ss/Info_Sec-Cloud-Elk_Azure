# Info_Sec-Cloud-Elk_Azure
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

[Link to ELK stack architecture sketch](https://github.com/Sos-b0ss/Info_Sec-Cloud-Elk_Azure/blob/main/Elk_Stack.drawio.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Ansible file may be used to install only certain pieces of it, such as Filebeat.

  - filebeat-playbook.yml
  - install-elk.yml
  - metricbeat-playbook.yml
  - pentest.yml

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly redundant, in addition to restricting bottlenecking to network access.
- Load balancers provide network security because in the case of an attack or incident the publicly available web servers will be able to stay up. 
- The advantage of using a jumpbox to access the backend of the server is that if in the event of infiltration the intruder would need to have not only the SSH key on their device, 
  the correct IP to connect from, but would also need to know the IP address of all the web virtual machines which are also provisioned in a way to need a specific SSH key to access.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the packets and system traffic.
- Uses FileBeat to generate a heatmap for which files are being accessed on the servers at what times.
- Metricbeat is used to monitor the network traffic and make sure it is shipped where it needs to be to prevent possible egress.

The configuration details of each machine may be found below.

| Name                  | Function           | IP Address | Operating System     |
|-----------------------|--------------------|------------|----------------------|
| Jump-Box-Provisioner2 | Gateway            | 10.1.0.4   | Linux (ubuntu 18.04) |
| Elk-1                 | DVWA web server VM | 10.0.0.4   | Linux (ubuntu 18.04) |
| Web-1                 | ELK stack host VM  | 10.1.0.7   | Linux (ubuntu 18.04) |
| Web-2                 | DVWA web server VM | 10.1.0.8   | Linux (ubuntu 18.04) |
| ...                   | ...                | ...        | ...                  |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Gateway machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- Public IP address of admin machine [Dynamic/Static]

Machines within the network can only be accessed by the Gateway Machine.
- Ansible docker container | 10.1.0.4

A summary of the access policies in place can be found in the table below:

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 10.0.0.1 10.0.0.2    |
|  ELK Box | No                  | 97.115.69.132        |
| DVWA Box | No                  | 97.115.113.235       |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- Reusability and to maintain a consistent secure provisioning structure

The playbook implements the following tasks:
- Install Docker
- Download image
- Install necessary Python DB
- Increase Virtual memory
- Use more memory
- Launch docker Container

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

[Link to docker ps screenshot](https://github.com/Sos-b0ss/Info_Sec-Cloud-Elk_Azure/blob/main/Docker_Commands.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1/Web-2 | 40.122.185.195

We have installed the following Beats on these machines:
- Metric Beat
- File Beat

These Beats allow us to collect the following information from each machine:
- These beats will allow us to collect web-app server access and provide the amount of requests received per hour to prevent and anticipate an attempted DOS attack. Logstash will compile all the logs for the webapp activity.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the Ansible playbook file to your provisioner docker container.
- Update the 'filebeat-config.yml' file to include the hosts private IP for your ELK VM.
- Run the playbook, and navigate to the virtual machine that the script was ran on to check that the installation worked as expected.
- The .yml file is the playbook? You copy it to the /etc/ansible/
- You need to update the hosts file to make sure the playbook runs on the correct machine? You will specify which server the ELK server will be installed on based off the IP address of your ELK server in the VNET.
- After correctly configuring your Security group for the ELK VM, and allow your IP, you will navigate to http://[your.ELK.VM.IP]:5601/app/kibana

Once you get connected to your Ansible provisioner virtual machine using SSH to get your container started and attached use:

>\$ sudo docker container list -a
>
>\$ sudo docker container start <name_of_Ansible_Provisioner_container>
>
>\$ sudo docker container attach <name_of_Ansible_Provisioner_container>

Inside your provisioners docker container you want to get your file copied to to your device, you will use Curl:
>\# cd /etc$ sudo dock/ansible
>
>\# mkdir files
>
>\# cd files
>
>\# Curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > filebeat-config.yml

Now that the file is copied over to your ansible container and you get your config file edited and the proper IPs filled, you will need to run the ansible playbook:
>\# ansible-playbook filebeat-config.yml

When this playbook runs and you need to check whether your VM was properly imaged, you will SSH into it using the Private IP that is in the VNET.
>\# ssh <username>@<ELK_stack_private_IP>

This will allow you to make sure you have proper connectivity to the Virtual Machine still, and you can try to ping public domains to verify it is not publicly available. 
(because you want to make it so that only your public IP is white-listed on your Azure Security group.)

Now you will open your browser on a device connected to that white-listed public IP and navigate to the web-address:
- http://[your.VM.IP]:5601/app/kibana

From this page you will be able to use test whether your webapp is properly configured, ported and accessable from your ELK VM.

Common errors during Imaging:
- If the Azure Security group is not configured correctly, your Ansible docker container on the Provisioner_VM and the ELK_VM are unable to communicate.
- The hosts file MUST be updated with the proper IP addresses and [Server_Names] that are being used in the ansible playbook so that your playbooks dont throw errors during the commands.
- You need to make certain your VMs are running and that the IPs did not change in case they are still dynamic and the VMs got restarted.
- Ensure you update your SSH, id_rsa.pub was updated properly in your VMs Azure password settings and that you are using the right Username before the public ip.
