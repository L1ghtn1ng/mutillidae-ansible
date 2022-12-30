# Mutillidae-Ansible
Easy way to install Mutillidae on Ubuntu 22.04 using Ansible,
for more information on this app please see the upstream repo https://github.com/webpwnized/mutillidae. Before running this playbook you will need to run collection_install.sh first but you do need ansible installed before that.

# Running playbook
Setup an Ubuntu 22.04 server with a user called ansible and update the IP in the hosts.yml file and do the following from the root of this folder on a host that has ansible installed

ansible-playbook main.yml -i hosts.yml main.yml -k -K
