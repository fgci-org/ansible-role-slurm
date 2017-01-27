ansible-role-slurm
------------------
[![GitPitch](https://gitpitch.com/assets/badge.svg)](https://gitpitch.com/CSC-IT-Center-for-Science/ansible-role-slurm/master) 
[![Build Status](https://travis-ci.org/CSC-IT-Center-for-Science/ansible-role-slurm.svg?branch=master)](https://travis-ci.org/CSC-IT-Center-for-Science/ansible-role-slurm)

# Creates a SLURM cluster

Tested with SLURM versions:
 - 14.11.0
 - 14.11.3
 - 15.08.x
 - 16.05.x

Tested with these Linux distributions:
 - CentOS 6
  - Only 14.11.x
 - CentOS 7
  - 15.08.x
  - 16.05.x

## Dependencies

 - https://github.com/jabl/ansible-role-pam
  - configure PAM - limit access to the compute nodes
 - https://github.com/CSC-IT-Center-for-Science/ansible-role-nhc
  - Configure Node Health Checker

## How-To

### Initial playbook configuration

*Playbook variables:*

All variables should be defined in defaults/main.yml

All nodes run munge. Nodes which are part of the slurm\_compute host
group will additionally run slurmd. Nodes which are part of the
slurm\_service host group will additionally runs slurmctld and
slurmdbd (unless {{ slurm_accounting_storage_host }} is not the same as {{ slurm_service_node }}). Nodes which are in neither of these two host groups are 
assumed to be submit hosts.

You also need to add a mysql\_slurm_password: "PASSWORD" string
somewhere. This will be used to set a password for the slurm mysql
user. See http://docs.ansible.com/ansible/playbooks_vault.html

To add your own nodes and queues define the slurm_nodelist and slurm_partitionlist lists.

It is possible to run the slurmdbd on a different host than the slurmctld by changing the slurm_accounting_storage_host variable.

SLURM 16.05 can be gotten from the FGCI yum repo by setting:
<pre>
fgci_slurmrepo_version: "fgcislurm1605"
</pre>

### Implementation

A playbook that uses this role: https://github.com/CSC-IT-Center-for-Science/fgci-ansible

Or you can check out the [tests/test.yml](tests/test.yml) in this repo.

Example Playbook
----------------

<pre>
    - hosts: compute
      strategy: free
      roles:
         - { role: ansible-role-pam }
         - { role: ansible-role-nhc }
         - { role: ansible-role-slurm }

    - hosts: install
      roles:
         - { role: ansible-role-slurm }
</pre>

### Known Issues

 - This role used to be able to build slurm rpms, distribute them and install them. The last tag/release that had this feature was v1.5.0

# Authors / Contributors:

 - Marco Passerini (original author)
 - https://github.com/martbhell
 - https://github.com/tiggi
 - https://github.com/A1ve5
 - https://github.com/jabl
