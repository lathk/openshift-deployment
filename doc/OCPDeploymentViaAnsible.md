# OCP 4 install on VMware

## Playbook Description
3 playbooks :
 - configure_bastion.yaml : Setup the Bastion node
 - create_openshift_nodes.yaml : to provisionng the infrastructure on vmware
 - installation_openshift.yaml : to install Openshift

### Configure variables
update the extra_vars in ansible/extra_vars/example.yaml

### Setup the Bastion node
configure_bastion.yaml


### Provisionning VMs for Openshift
create_openshift_nodes.yaml

### Installation de Openshift

-  installation_openshift.yaml
