Switch from FGCI to OHPC slurm packages
---------------------------------------

NOTE 2021-09-30:
Changes has been made to slurm role after writing this doc.
It may not be relevant anymore. Now by default the slurm is coming from ohpc.

In general one needs to be careful with slurmdbd and run it in
the foreground during upgrade to monitor progress. See
http://slurm.schedmd.com/quickstart_admin.html#upgrade

- Service break is required. Make sure no jobs are running.

### On the node running slurmdbd, install node

- Stop slurmdbd: systemctl stop slurmdbd
- Remove all slurm and munge packages: yum remove '*slurm*' 'munge*'
- Install slurm server packages: yum install ohpc-slurm-server
- Start munge: systemctl restart munge
- Start slurmdbd in the foreground: /sbin/slurmdbd -D -v
- Wait until the DB upgrade is completed "slurmdbd version XX.XX.X started" (can take 45 mins)
- Stop the slurmdbd running in the foreground: Ctrl-C
- Start slurmdbd via systemd: systemctl start slurmdbd


### On the node with ansible

- In ansible group_vars/all/all.yml, set slurm_repo: "ohpc"
- Run: ansible-playbook install.yml --tags=fgci-install

### On the slurm controller node(s), install node

- systemctl stop slurmctld
- Remove slurm and munge packages if not done already: yum -y remove '*slurm*' 'munge*'
- Install slurm server packages if not done already: yum -y install ohpc-slurm-server
- If install node: systemctl restart slurmdbd; sleep 3; systemctl restart munge; sleep 3; systemctl restart slurmctld
- If only controller node: systemctl restart munge; sleep 3; systemctl restart slurmctld


### On compute nodes

- systemctl stop slurmd
- yum -y remove '*slurm*' 'munge*'
- yum -y install ohpc-slurm-client
- systemctl start munge; sleep 3; systemctl start slurmd
- Run to lock slurm version: ansible-playbook install.yml -t slurm
- Run to lock slurm version: ansible-playbook compute.yml -t slurm

### Grid node
- yum remove '*slurm*' 'munge*'
- Run ansible: ansible-playbook grid.yml -t slurm

### Login node
- yum -y remove '*slurm*' 'munge*'
- yum -y install ohpc-slurm-client
- Run ansible: ansible-playbook login.yml -t slurm
- Switching from FGCI to OHPC slurm packages is complete

Upgrading OHPC slurm packages
-----------------------------

To upgrade to a newer slurm OHPC version:

1. Do steps 0-1 from previous list above
2. Delete all the slurm and munge versionlock stuff from /etc/yum/pluginconf.d/versionlock.list
3. In group_vars set slurm_ohpc_versionlock to False
4. ansible-playbook install.yml --tags=fgci-install
5. yum update
6. Do steps 4-7 from the above list.
7. Upgrade all the nodes per steps 9-16 except run "yum update" instead of remove+install.
8. In group_vars set slurm_ohpc_versionlock to True and run this role to lock the version again.


From 15.08 to 16.05
-------------------

Mostly the steps are done outside ansible, but it helps a bit.

Slurm 16.05 details: http://slurm.schedmd.com/SLUG16/V16.05.pdf

Official upgrade documentation: http://slurm.schedmd.com/quickstart_admin.html#upgrade

This guide is not a replacement for the official instructions.

It assumes you are using the https://github.com/fgci-org/fgci-ansible playbooks where install.yml is to the service node and compute.yml to the compute nodes. It also assumes you are using the FGCI yum repo to fetch slurm packages.

 1. stop slurmdbd
 1. take backup with something like: /usr/local/sbin/dump-all-databases.sh -o /outdir -z
 1. set ansible variable fgci_slurmrepo_version to "fgcislurm1605" (this will point the yum.repos/fgislurm.repo to the 1605 repo in group_vars/all
 1. run slurm role until task "Add FGI slurm repo" on install node
 1. ansible-playbook install.yml -t slurm -step
 1. answer Y on the setup task and "Add FGI slurm repo" tasks only, rest N. ctrl+C quit after the "FGI slurm repo" task is done
 1. yum update
 1. schema upgrade, slurmdbd -D
 1. ctrl+C when it says started
 1. systemctl daemon-reload
 1. systemctl start slurmdbd
 1. systemctl restart slurmctld
 1. start slurmdbd 
 1. bash tools/pullReqs.sh (to rsync group_vars to the internal web server for ansible-pull)
 1. ansible-playbook compute.yml -t slurm
 1. run yum update to update slurm
 1. pdsh -g compute -l root yum -y update
 1. optionally systemctl daemon-reload
 1. systemctl restart slurmd
 1. run the slurm role on the submit hosts (grid and login)
 1. ansible-playbook site.yml -t slurm -l login,grid
 1. yum update on login and grid node
