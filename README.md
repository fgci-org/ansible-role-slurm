ansible-role-slurm
------------------

[![Build Status](https://travis-ci.org/CSC-IT-Center-for-Science/ansible-role-slurm.svg?branch=master)](https://travis-ci.org/CSC-IT-Center-for-Science/ansible-role-slurm)

# Creates a slurm cluster

Tested with slurm versions:
 - 14.11.0
 - 14.11.3
 - 15.08.x

Tested with these linux distributions:
 - CentOS 6
  - Only 14.11.x
 - CentOS 7
  - Only 15.08.x

## Dependencies

 - https://github.com/jabl/ansible-role-pam
  - Used to configure PAM - limit access to the compute nodes


## How-To

### Initial playbook configuration

*Playbook variables:*

All variables should be defined in defaults/main.yml

 - if {{ slurm_type }} in group_vars/group/group.yml is set to:
  - "compute" the group runs slurmd
  - "submit" the group run munge
  - "service" the group runs slurmdbd, slurmctld

You also need to add a mysql_slurm_password: "PASSWORD" string somewhere. This will be used to set a password for the slurm mysql user. See http://docs.ansible.com/ansible/playbooks_vault.html

### Implementation

A playbook that uses this role: https://github.com/CSC-IT-Center-for-Science/fgci-ansible

# Authors / Contributors:

 - Marco Passerini (original author)
 - https://github.com/martbhell
 - https://github.com/tiggi
 - https://github.com/A1ve5
 - https://github.com/jabl
