# configure_bastion

Role that configure the bastion server from where OpenShift installer will be executed.

## Requirements

None

## Role Variables

### OpenShift general variables

- `openshift_version`: OpenShift Container Platform version to install

- `openshift_cluster_name`: OpenShift cluster name

- `openshift_base_domain`: OpenShift cluster base domain

- `openshift_worker_replicas_number`: Number of worker replicas (should be set to 0 for UPI install)

- `openshift_master_replicas_number`: Number of master replicas

These variables have default values (cf. below) that **must be replaced**.

```yaml
openshift_version: "4.2.7"
openshift_cluster_name: "ocp-cluster"

openshift_base_domain: "example.com"
openshift_worker_replicas_number: "0"
openshift_master_replicas_number: "3"
```

### OpenShift installer variables

These variables are initialized by default for the setting of OpenShift installer

- `openshift_binary_dir`: OpenShift binaries directory path

```yaml
openshift_binary_dir: "/usr/bin"
```

- `openshift_working_directory_name`: name of directory where the OpenShift installer creates/generates config files (set by default to  `openshift_cluster_name`)

```yaml
openshift_working_directory_name: "{{ openshift_cluster_name }}"
```

- `openshift_working_directory_path`: absolute path of directory where the OpenShift installer creates/generates config files (set by default to `~/< openshift_working_directory_name >`)

```yaml
openshift_working_directory_path: "{{ ansible_env.HOME }}/{{ openshift_working_directory_name }}"
```

- `openshift_binaries_url`: url of OpenShift binaries to download

```yaml
openshift_binaries_url: "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/{{ openshift_version }}"
```

- `openshift_install_config_yaml_name`: install-config filename

```yaml
openshift_install_config_yaml_name: "install-config.yaml"
```

The following variables concern the `openshift-install` and `oc` binaries:

```yaml
openshift_installer_binary_url: "{{ openshift_binaries_url }}/openshift-install-linux-{{ openshift_version }}.tar.gz"
openshift_installer_binary_name: "openshift-install"
openshift_installer_binary_mode: "0755"
_openshift_installer_cmd: "{{ openshift_binary_dir }}/{{ openshift_installer_binary_name }}"

openshift_oc_client_binary_url: "{{ openshift_binaries_url }}/openshift-client-linux-{{ openshift_version }}.tar.gz"
openshift_oc_client_binary_name: "oc"
openshift_oc_client_binary_mode: "0755"
_openshift_oc_client_cmd: "{{ openshift_binary_dir }}/{{ openshift_oc_client_binary_name }}"
```

### OpenShift manifests

These variables are initialized to configure the generated manifests.

```yaml
openshift_manifests_dir_name: "manifests"
openshift_cluster_scheduler_manifest:
  filename: "cluster-scheduler-02-config.yml"
  value: "false"
```

### OpenShift ignition config files

These variables are initialized to manage the generated ignition files.

```yaml
openshift_append_bootstrap_ign_name: "append-bootstrap.ign"
openshift_ignition_config_files_root_names:
  - "master"
  - "worker"
  - "append-bootstrap"
```

### SSH key

- `ssh_key_name`: SSH key name that will be generated to access to the nodes

```yaml
ssh_key_name: "cluster-ocp-key"
```

### HTTP server

These variables are initialized to install and configure http server.

```yaml
http_server_package_name: "httpd"
http_server_service_name: "httpd"
http_server_user: "apache"
http_server_group: "apache"
http_server_data_dir: "/var/www/html"
http_server_ignition_files_dir_name: "ignition"
```

## Dependencies

N/A

## Example Playbook

```yaml
- hosts: localhost
  roles:
     - role: configure_bastion
```
