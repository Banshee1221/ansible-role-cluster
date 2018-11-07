# ansible-role-cluster

This role is used to set up the basic packages and tools required for machines that are part of the SANBI HPC cluster.

In it's currect state this role will set up:
1) Check that the appropriate directories exist on the filesystem
2) Install and configure the Singularity runtime
    - It enables `hostfs` in Singularity by default, exposing user directories.
3) Apply sysctl tweaks for better performance for HPC applications

## Requirements

- Shared directories already mounted (refer to [ansible-cephfs-client](https://github.com/SANBI-SA/ansible-role-cephfs_client) for SANBI).
- Setting the inventory variable `headnode=True` on the node that will act as the head or login node of the cluster.

## Variables

### Directories

| Variable    | Type | Description                                                 |
|-------------|------|-------------------------------------------------------------|
| `tools_dir` | str  | The absolute path of where tools are stored on the cluster. |

### Software

| Variable                | Type        | Description                                  |
|-------------------------|-------------|----------------------------------------------|
| `apt_repos_singulairty` | list (str)  | List of repositories to add for Singularity. |
| `apt_key_singularity`   | str         | Apt key to compliment above repositories.    |

### Inventory

The `headnode` variable must be defined and set to true on the machines that will act as the head/login node. The role will fail if this is not set. Following is an example of a host group:

```ini
[cluster]
login.sanbi.ac.za headnode=True
node01.sanbi.ac.za
node02.sanbi.ac.za
node03.sanbi.ac.za
node04.sanbi.ac.za
```

## Usage

1. Set the appropriate variables (listed above).
2. Create a playbook that includes this role. Following is an example.
   ```shell
   - hosts: cluster
     become: true
     # The pre_tasks section will disable gathering of
     # facts to first install python2 if it is not
     # already installed and then gather facts.
     gather_facts: false
     pre_tasks:
     - name: Install python2 for Ansible
       raw: bash -c "test -e /usr/bin/python || (apt -qqy update && apt install -qqy python-minimal)"
       register: output
       changed_when: output.stdout != ""
     - name: Gathering Facts
       setup:
     roles:
       # This is actually where this role
       # (ansible-role-cluster) is being included.
       - role: ansible-role-cluster
   ```