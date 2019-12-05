# provision_vmware_node

Role that provision VMware nodes.

## Requirements

- PyVmomi library for using VMware modules

- [Red Hat CoreOS OVA](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.2/latest/rhcos-4.2.0-x86_64-vmware.ova) available at vCenter

- Ignition configuration files still generated

## Role Variables

### Required extra variable

- `openshift_cluster_name`: OpenShift cluster name

### Default vars

- `openshift_working_directory_name`: OpenShift installer working directory

- `ignition_files_path`: Absolute path of ignition files directory

```yaml
ignition_files_path: "{{ ansible_env.HOME }}/{{ openshift_working_directory_name }}"
```

### RH CoreOS

- `openshift_rhcos_template_name`: Red Hat CoreOS template name imported into VMware vCenter

### List of nodes

All the nodes are ordered into different lists by type of OpenShift node. An element of these lists is the hostname of the node.

- `bootstrap_nodes_list`: list of bootstrap node (should contain one element)
- `master_nodes_list`: list of master nodes
- `worker_nodes_list`: list of worker nodes

### Node vm configuration

Each type of node has its VM configuration.

- `bootstrap_node`

```yaml
bootstrap_node:
  name: "ocp-bootstrap_node"
  ram: "16384"
  cpus: "4"
  disk:
    size: "120"
    type: "thin"
  ignition_file_path: "{{ ignition_files_path }}/append-bootstrap.64"
```

- `master_node`

```yaml
master_node:
  ram: "16384"
  cpus: "4"
  disk:
    size: "120"
    type: "thin"
  ignition_file_path: "{{ ignition_files_path }}/master.64"
```

- `worker_node`

```yaml
worker_node:
  ram: "16384"
  cpus: "4"
  disk:
    size: "120"
    type: "thin"
  ignition_file_path: "{{ ignition_files_path }}/worker.64"
```

## Dependencies

N/A

## Example Playbook

```yaml
- hosts: localhost
  roles:
    - provision_vmware_node
```

## License

BSD
