## Cluster Setup for ONTAP 9.6 and later - REST API and Ansible

### Description
This is an Ansible role for accomplishing a basic cluster setup operation for a single or 2-node cluster. After gathering the needed information from a provided vars file, a JSON body is generated and the ONTAP REST APIs are called to start cluster creation. 

The goal is to use a SINGLE ROLE using a vars file to accomplish the needed tasks. Unlike many roles, the tasks/main.yml is used to invoke each task (include_tasks) for each step of the workflow, utilizing looping structures when required. This allows each task to be individually written, reducing complexity, and hopefully making it easier to read, modify, and maintain.

The 'default/main.yml' is well commented, covering each var with a description, usage, and examples. Use this file to build a system-specific vars file for input and modify to suit your needs. If a variable is not set then the relevant task is not invoked. The 'defaults/main.yml' has most of the vars unset so they can be simply omitted from the provided vars input file - Ansible will use the default setting.

### Disclaimer
The role and associated components are provided as-is and are intended to provide an example of utilizing the ONTAP REST API with Ansible. Fully test in a non-production enviornment before implementing. Feel free to utilize/modify any portion of code for your specific needs.

### Notes
* This role has only been tested against a Two Node Switchless Cluster (TNSC) which allows ONTAP to be 'aware' of the interconnects and assign link-locak (APIPA) addresses before cluster setup.  This has NOT been tested on a switched cluster so not sure if the same behavior will occur. Run the playbook with '-vvv' to see the output of the registered vars and returned JSON.
* A 2-node ONTAP Simulator configuration is not supported. There is no switchless connection (or shared anything), so ONTAP does not automatically generate cluster interfaces.

### Requirements
* ONTAP 9.6 or later
* Node setup has been completed on each node and the 'admin' password has been set
* Ansible 2.9.7 or later
* NetApp Library: netapp-lib (pip3 install --upgrade netapp-lib)
* This role copied to the desired location (ie. ~/roles or Ansible configured roles path) 

### Dev/Test Environment
* CentOS 8.1
* Ansible 2.9.7
* Python 3.6.8
* netapp-lib 
* FAS2552 running ONTAP v9.7P3 - Two Node Switchless Cluster
* ONTAP Simulator 9.7

### Usage
* Example: **ansible-playbook pb_clustersetup.yml -e "@vars_clustersetup.yml"**
* Implement password security in compliance with the environment and best practices (separate vars file, ansible-vault, etc)
* A sample playbook and vars files are provided as a reference.

### Workflow
1. Verify vars file settings before running tasks
2. Ping the cluster IP address - if reachable the task will stop (assumes cluster exists)
3. Ping each specified node - if NOT reachable the task will stop
4. If a 2-node cluster, get the cluster interface IP addresses from node 1 
5. Create the cluster - uses templates to build the URL and the JSON body (jinja2)
6. Monitor the job until success or failure

### Example Playbook
<pre>
- name: ONTAP Cluster Setup
  hosts: localhost
  gather_facts: no
  tasks:
  - include_role:
      name: ontapclustersetuprest
</pre>

### Example vars file
<pre>
clustername: fascluster
username: admin
password: "Netapp123!"
clusterip: 10.10.10.13
nodes: 
  - { nodename: "{{ clustername }}-01", nodeip: "10.10.10.14" }
  - { nodename: "{{ clustername }}-02", nodeip: "10.10.10.15" }
cidrnetmask: 24
gateway: 10.10.10.1
</pre>

To pass a vars file to the playbook:

   **ansible-playbook pb_clustersetup.yml -e "@vars_clustersetup.yml"**

The 'tasks/main.yml' calls the required tasks based on the vars file settings

### Comments
Development of this role is ongoing and new tasks are added regularly.

Here's a list of tasks currently being worked on:
* Use 'expect' to complete node setup and set 'admin' password - access through Service Processor(s) / Maintenace Menu / System Console
* Research serial console interface (would require a direct connection from the Ansible controller or a serial-Ethernet solution)
