## `ocp_virt`

This role manages OpenShift VirtualMachines (create, delete, start, stop, restart). For `create`, it clones from an existing DataVolume/DataSource, configures storage/networking/cloud-init, and waits for the VM to be running.

---

### ✅ Role Requirements

- Access to an OpenShift cluster with OpenShift Virtualization enabled.
- The `redhat.openshift_virtualization` Ansible collection must be installed.
- Cluster access credentials should be provided via `KUBECONFIG` or environment variables.

---

### 📦 Role Variables

| Variable                | Description                                                                 | Required | Default                                      |
|-------------------------|-----------------------------------------------------------------------------|----------|----------------------------------------------|
| `vm_action`             | Action to perform: `create`, `delete`, `start`, `stop`, `restart`          |          | `create`                                     |
| `vm_name`               | Name of the VirtualMachine                                                  | ✅       | —                                            |
| `vm_namespace`          | Namespace for the VirtualMachine                                            | ✅       | —                                            |
| `source_name`           | Name of the source DataVolume/DataSource to clone                           | ✅       | —                                            |
| `storage_class`         | Storage class for the cloned DataVolume                                     | ✅       | —                                            |
| `instance_type`         | Instance type for the VM                                                    | ✅       | —                                            |
| `cloud_init_user_data`  | Cloud-init user data for VM initialization                                  | ✅       | —                                            |
| `app_label`             | Label for the VM (defaults to `vm_name`)                                    |          | `vm_name`                                    |
| `preference`            | VM preference                                                               |          | `rhel.9`                                     |
| `source_kind`           | Kind of source to clone (`DataSource` or `DataVolume`)                      |          | `DataSource`                                 |
| `source_namespace`      | Namespace of the source image                                               |          | `openshift-virtualization-os-images`         |
| `wait_timeout`          | Timeout (seconds) to wait for action completion                              |          | `3600` (create), `600` (start/stop/restart/delete) |
| `network_name`          | Name of the network interface                                               |          | `default`                                    |
| `vm_secondary_networks` | Optional list of NAD-backed secondary interfaces                            |          | `null`                                       |

`vm_secondary_networks` item schema:
- `nad_name` (required): NetworkAttachmentDefinition name.
- `nad_namespace` (optional): NAD namespace. Defaults to `vm_namespace`.
- `interface_name` (optional): VM interface/network name. Defaults to `nad<index>`.

---

### **Example Usage**

```yaml
---
- name: Create a RHEL9 VM from a DataSource
  hosts: localhost
  gather_facts: false
  vars:
    vm_action: create
    vm_name: rhel9-vm
    vm_namespace: default
    source_name: rhel9-image
    storage_class: ocs-storagecluster-ceph-rbd
    instance_type: u1.medium
    cloud_init_user_data: |
      #cloud-config
      password: mypassword
      chpasswd: { expire: False }
    vm_secondary_networks:
      - nad_name: vlan-100
        nad_namespace: default
        interface_name: net1
  roles:
    - role: ocp_virt
```

`stop` action example:

```yaml
- name: Stop VM
  hosts: localhost
  gather_facts: false
  vars:
    vm_action: stop
    vm_name: rhel9-vm
    vm_namespace: default
  roles:
    - role: ocp_virt
```

`delete` action example:

```yaml
- name: Delete VM
  hosts: localhost
  gather_facts: false
  vars:
    vm_action: delete
    vm_name: rhel9-vm
    vm_namespace: default
  roles:
    - role: ocp_virt
```

---

### **Structure**
```
ocp_virt/
├── defaults/main.yml
├── vars/main.yml
├── tasks/
│   ├── main.yml
│   └── create_vm.yml
├── templates/
├── handlers/main.yml
├── files/
├── tests/
│   ├── inventory
│   └── test.yml
└──
```
