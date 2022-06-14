# OCP 4 install on VMware from ansible playbook




## Playbook Description
3 playbooks :
 - configure_bastion.yaml : Setup the Bastion node
 - create_openshift_nodes.yaml : to provisionng the infrastructure on vmware
 - installation_openshift.yaml : to install Openshift

### ssh configuration
before run the playbook, create the ssh key 

```shell
ssh-keygen
ssh-copy-id localhost
```
need to install ansible also

```shell
subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
yum install -y ansible
```

- install `pyvmomi` library

```shell
pip3 install pyvmomi
```

- install `git` 

```shell
yum install git -y
```

### Setup the Bastion node
Clone the repo && change the repository : 
```shell
git clone https://github.com/NicolasO/openshift-hybrid-install.git
cd openshift-hybrid-install/ansible && cp extra_vars/example.yaml extra_vars/ocp_install.yaml
```

update the extra_vars in ansible/extra_vars/example.yaml

- replace with the right values into this copied `extra_vars/ocp_install.yaml` file

```shell
vim extra_vars/ocp_install.yaml
```

configure_bastion.yaml

- and execute `configure_bastion` playbook by also specified the extra_vars file

```shell
ansible-playbook playbooks/configure_bastion.yaml -e @extra_vars/ocp_install.yaml -e 'ansible_python_interpreter=/usr/bin/python3'
```
or provision up to the VMs: 

```shell
ansible-playbook playbooks/install_openshift.yaml -e @extra_vars/cic.yaml
```


### Provisionning VMs for Openshift
create_openshift_nodes.yaml

### Installation de Openshift

-  installation_openshift.yaml
