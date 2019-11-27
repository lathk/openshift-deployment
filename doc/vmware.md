# OCP 4 install on VMware

## vSphere preparation

- download [RHCOS OVA](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.2/4.2.0/rhcos-4.2.0-x86_64-vmware.ova)

- create new folder with the name of the OCP cluster

- import OVA in vSphere

- edit OVA template settings

  - on `VM Options`, expand `Advanced` and set `Latency Sensivity` to *HIGH*

  - next to `Configuration Parameters`, click on `Edit Configuration...` and add following parameters:
    - `guestinfo.ignition.config.data`, set the value to `changeme`
    - `guestinfo.ignition.config.data.encoding`, set the value to `base64`
    - `disk.EnableUUID`, set this value to `TRUE`

- clone the OVA template for bootstrap, master and worker nodes


## Bastion VM configuration

- create a VMware VM

- install OS

- configure proxy (to get access on Internet)

- install tools

```
yum install -y tmux jq
```

- get OpenShift installer binary

```shell
OCP_VERSION=4.2.7
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux-${OCP_VERSION}.tar.gz
tar zxvf openshift-install-linux-${OCP_VERSION}.tar.gz -C /usr/bin
rm -f openshift-install-linux-${OCP_VERSION}.tar.gz /usr/bin/README.md
chmod +x /usr/bin/openshift-install
```

- get oc client

```shell
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux-${OCP_VERSION}.tar.gz
tar zxvf openshift-client-linux-${OCP_VERSION}.tar.gz -C /usr/bin
rm -f openshift-client-linux-${OCP_VERSION}.tar.gz /usr/bin/README.md
chmod +x /usr/bin/oc
```

- add oc completion to bashrc

```shell
oc completion bash >/etc/bash_completion.d/openshift
```

- configure a HTTP server

The HTTP server will be used to provide bootstrap ignition configuration file

```shell
yum install -y nginx
systemctl start nginx
mkdir /usr/share/nginx/html/ignition/
```

## Installation configuration files

- create [cloud.openshift.com](https://cloud.openshift.com/) pull secret

- create ssh key

```shell
ssh-keygen -f ~/.ssh/cluster-ocp-key -N ''
```

- create working installer directory

```shell
mkdir ocp-vsphere
```

- create `install-config.yaml` file

```yaml
apiVersion: v1
## The base domain of the cluster. All DNS records will be sub-domains of this base and will also include the cluster name.
baseDomain: example.com
## The proxy configuration (if it is necessary)
proxy:
  httpProxy: http://<username>:<pswd>@<ip>:<port>
  httpsProxy: http://<username>:<pswd>@<ip>:<port>
  noProxy: example.com
compute:
- name: worker
  replicas: 0
controlPlane:
  name: master
  replicas: 3
metadata:
  ## The name for the cluster
  name: test
platform:
  vsphere:
    ## The hostname or IP address of the vCenter
    vcenter: your.vcenter.server
    ## The name of the user for accessing the vCenter
    username: your_vsphere_username
    ## The password associated with the user
    password: your_vsphere_password
    ## The datacenter in the vCenter
    datacenter: your_datacenter
    ## The default datastore to use.
    defaultDatastore: your_datastore
## The pull secret that provides components in the cluster access to images for OpenShift components.
pullSecret: ''
## The default SSH key that will be programmed for `core` user.
sshKey: ''
```

**NB**: the number of worker replicas is set to 0 because UPI does not perform creating and managing workers

- generate kubernetes manifests

```shell
openshift-install --dir ocp-vsphere create manifests
```

- modify manifests (if necessary)

If the manifest `manifests/cluster-scheduler-02-config.yml` exists:

 ```
 sed -i 's/mastersSchedulable: true/mastersSchedulable: false/g' manifests/cluster-scheduler-02-config.yml
 ```

- generate ignition files

```shell
openshift-install --dir ocp-vsphere create ignition-configs
```


## RHCOS VM configuration with ignition files

### Ignition files

- create a append-bootstrap.ign file

```json
{
  "ignition": {
    "config": {
      "append": [
        {
          "source": "http://<http_server_ip>/ignition/bootstrap.ign",
          "verification": {}
        }
      ]
    },
    "timeouts": {},
    "version": "2.1.0"
  },
  "networkd": {},
  "passwd": {},
  "storage": {},
  "systemd": {}
}
```

- encode ignition files to base64

```shell
for i in append-bootstrap master worker
do
base64 -w0 < $i.ign > $i.64
done
```

- upload the bootstrap ignition file

```shell
scp bootstrap.ign root@<http_server_ip>:/usr/share/nginx/html/ignition/
```

### Create VMs

On vSphere GUI,

- right-click on the bootstrap template
  - click on `New VM From this Template...`
  - `Select a name and folder`
  - `Select storage`
  - on `Select clone options`, check `Customize this virtual machine’s hardware`
    - on `Customize hardware`, modify the parameter `guestinfo.ignition.config.data` with the content of ignition file encoded on base64 (for example, `cat append-bootstrap.64`)

- do it for the all of nodes of the cluster

- boot all VMs


## Creating the cluster

On Bastion VM,

### Monitor the bootsrap process

```shell
openshift-install --dir ocp-vsphere wait-for bootstrap-complete --log-level debug
```

**NB**: The message `DEBUG Bootstrap status: complete`

### Login to the cluster

- export `KUBECONFIG`

```
export KUBECONFIG=<working_installer_dir>/auth/kubeconfig
```

### Approving CSRs

- confirm that the cluster recognizes the machines

```shell
oc get nodes
```

- review the status of approval CSRs

```shell
oc get csr
```

- If the CSRs were not approved, after all of the pending CSRs for the machines you added are in *Pending* status, approve the CSRs for your cluster machines

```
oc get csr -ojson | jq -r '.items[] | select(.status == {} ) | .metadata.name' | xargs oc adm certificate approve
```
