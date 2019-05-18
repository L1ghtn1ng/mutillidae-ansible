# Mutillidae-Ansible
Easy way to install Mutillidae on Ubuntu 19.04 using Ansible,
for more information on this app please see the upstream repo https://github.com/webpwnized/mutillidae

# Running playbook
Setup an Ubuntu 19.04 server with a user called ansible and update the IP in the hosts file and do the following from the root of this folder on a host that has ansible installed

ansible-playbook main.yml -i hosts --ask-become-pass -v
