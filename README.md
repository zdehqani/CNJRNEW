# Conjur Automation

This ansible playbook is used to automate the installation and configuration of the Conjur cluster(master,standby and follower nodes).
And also it is used to perform the failover of the conjur cluster and restore the cluster from a backup file.

## Requirements

- Host FQDN's to execute the playbook against Linux on the remote hosts
- SSH open on port 22 on the remote Linux host
- PYthon 3.6 or Higher and Git installed on the workstation running the playbook
- The workstation running the playbook must have network connectivity to the remote host and DNS resolution.

## Environment setup

- Get the Conjur Cluster Playbooks

  ```bash
  git clone https://github.com/<org>/conjur_automation.git
  cd conjur_automation
  ```

- Install Python Dependencies

  ```bash
  python3 -m venv ./env
  source env/bin/activate
  pip install --upgrade pip
  python3 -m pip install --ignore-installed -r requirements.txt
  ```
## Inventory

Prior to running the playbooks, make sure the [hosts file](https://github.com/conjur_automation/blob/main/inventories/iam-lab/hosts) of the respective environment is `updated` with relevant hosts data as shown below.

```ini
[master]
#Add the hostname of the conjur master node
conjur1-poc.xyz.org

[standby]
#Add the hostname of the conjur standby node
conjur2-poc.xyz.org
conjur3-poc.xyz.org

[followers]
#Add the hostname of the conjur follower node
conjur5-poc.xyz.org

[synchronizer]
# Add here list of hosts of conjur vault synchronizers
#conjur4-poc.xyz.org

[conjurcluster:children]
master
standby

[linux:children]
conjurcluster
followers

[windows:children]
synchronizer
```

## Running the  playbook

###### Note: Replace ***** with actual password in place of the password field.

 Run the below playbook to **setup** the complete conjur cluster consisting of master, standby and follower nodes:
 
 ```bash
 ansible-playbook cluster_deploy.yml -i inventories/iam-lab/ --extra-vars "@vars/iam-lab.yml" 
 -e "ansible_user=<ansible_user> ansible_password=***** clusterAdminPass=*****
 hsm_partition_pin=<hsm_partition_pin> hsm_admin_password=***** certPassword=***** deployment_type=cluster"
 ```

Run the below playbook to **restore** the complete conjur cluster consisting of master, standby and follower nodes from a backup file:
 
 ```bash
 ansible-playbook restore.yml -i inventories/iam-lab/ --extra-vars "@vars/iam-lab.yml"
 -e "ansible_user=<ansible_user> ansible_password=***** clusterAdminPass=*****
 hsm_partition_pin=<hsm_partition_pin> hsm_admin_password=***** certPassword=***** backupfile=<backup_file_name> keyfile=<Key_file_name> master_node=<master_node> standby_node=<standby_node1:standby_node2> follower_node=<follower_node1:follower_node2> hsm_integartion=false"
 ```
 
 Run the below playbook to install only Conjur standby node(s):
 
 ```bash
 ansible-playbook standby_deploy.yml -i inventories/iam-lab/ --extra-vars "@vars/iam-lab.yml" 
 -e "ansible_user=<ansible_user> ansible_password=***** clusterAdminPass=*****
 hsm_partition_pin=<hsm_partition_pin> hsm_admin_password=*****
 certPassword=***** master_node=<master_node> standby_node=<standby_node1:standby_node2>"
 ```
 Run the below playbook to install only Conjur Follower node(s):
 
 ```bash
 ansible-playbook follower_deploy.yml -i inventories/iam-lab/ --extra-vars "@vars/iam-lab.yml" 
 -e "ansible_user=<ansible_user> ansible_password=***** clusterAdminPass=*****
 hsm_partition_pin=<hsm_partition_pin> hsm_admin_password=*****
 certPassword=***** master_node=<master_node> follower_node=<follower_node1:follower_node2>"
 ```
 Run the below playbook to perform manual **failover** of the Conjur cluster:
 
 ```bash 
 ansible-playbook autofailover.yml -i inventories/iam-lab/ -e "ansible_user=ansible_user ansible_password=***** master=<master_node>
 standby=<standby_node>"
 ```
 
