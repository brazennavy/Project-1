# Project-1
Azure/cloud/ansible/docker
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

https://github.com/brazennavy/Project-1/blob/main/Network%20Diagram.png

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the .yml file may be used to install only certain pieces of it, such as Filebeat.

Install_elk.yml
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azdmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes

  GNU nano 4.8                                                                                                                                                      filebeat-playbook.yml
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

  - name: setup filebeat
    command: filebeat setup

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

  GNU nano 4.8                                                                                                                                                     metricbeat-playbook.yml
      - name: Drop in metricbeat-config
        copy:
          src: /etc/ansible/metricbeat-config.yml
          dest: /etc/ansible/metricbeat.yml

      - name: Increase Virtual Memory
        command: sysctl -w vm.max_map_count=262144

      - name: Dedicating Memory On Startup
        shell: echo "vm.max_map_count=262144" >> /etc/sysctl.conf

      - name: Enable and configure docker module
        command: metricbeat modules enable docker

      - name: MetricBeat Dashboard configuration
        command: metricbeat setup

      - name: Metricbeat Setup
        command: metricbeat -e

      - name: Metricbeat Start
        command: service metricbeat start

      - name: Configuring Metricbeat to Run on Startup
        systemd:
          name: metricbeat
          enabled: yes



This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly resilient, in addition to restricting access to the network.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_
load balancers protect internal machines from external exposure, keeping information about interior systems inaccessible from unauthorized users. The jump box is a useful tool to access/configure machines within the resource group
Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the configuration and system security.
- _TODO: What does Filebeat do? Watch for logins using system logs
- _TODO: What does Metricbeat record? Metricbeat, intuitively, records and sends Metric logs

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux            |
| Web1     | WebServe | 10.0.0.7   | Linux            |
| Web2     | Webserve | 10.0.0.8   | Linux            |
| ElkServe | ELK-Serve| 10.1.0.5   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Load Balanced machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: Add whitelisted IP addresses (Local IP)

Machines within the network can only be accessed by SSH.
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address? I allowed the ansible container on the jumb box to access the ELK server

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 24.XXX.XXX.XXX       |
| Web1     | NO                  | 10.0.0.4             |
| Web2     | NO                  | 10.0.0.4             |
| ELK      | NO                  | 10.0.0.4             |
### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_
Automation allows configuing as many machines as you want simultaneously; this saves a massive amount of time
The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ...the .yml install elk playbook initially installs Docker
- ... Installs pip3 to allow ansible to function correctly on the machine
- ... Sets pip as the docker module
- ... Configures the minimum memory REQUIRED to run ELK stacks
- ... Downloads an elk container, and then sets that container to run on Startup

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.
https://ucb.bootcampcontent.com/CLennon/webserver_with_elk/-/blob/main/DOCKER_PS.png

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_
10.0.0.7
10.0.0.8

We have installed the following Beats on these machines: filebeat
filebeat
These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._
filebeat allows the collection of system logs, which are useful to look at events on the network.
### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file to /etc/ansible.
- Update the configuration file to include ports for services (5601 9200) 
- Run the playbook, and navigate to your machine to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_ the playbook.yml file
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on? the "hosts" section of the .yml playbook file indicates which group from your ansible hosts configuration will be acted on by the playbook.
- _Which URL do you navigate to in order to check that the ELK server is running? the public IP address

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
