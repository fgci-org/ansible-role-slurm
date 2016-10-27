From 15.08 to 16.05
-------------------

Mostly the steps are done outside ansible, but it helps a bit.

Slurm 16.05 details: http://slurm.schedmd.com/SLUG16/V16.05.pdf

Official upgrade documentation: http://slurm.schedmd.com/quickstart_admin.html#upgrade

This guide is not a replacement for the official instructions.

It assumes you are using the https://github.com/CSC-IT-Center-for-Science/fgci-ansible playbooks where install.yml is to the service node and compute.yml to the compute nodes. It also assumes you are using the FGCI yum repo to fetch slurm packages.

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
