ansible-role-k3s
===============

The `ansible-role-k3s` ansible role deploy a **k3s cluster** to a list of nodes.

Role Variables
--------------

The `ansible-role-k3s` ansible role offers a few important variables one can set to modify the behavior.

| Variable                         | Description                                                                                        |
|:--------------------------------:|:---------------------------------------------------------------------------------------------------|
| `ansible_k3s_sysctl_deploy`      | Apply `sysctl` configuration                                                                       |
| `ansible_k3s_uninstall`          | Removes *k3s* from a node                                                                          |
| `ansible_k3s_daemon_mode`        | Defines whether a node is a `server` or an `agent` (choices `["server", "agent"]`)                 |
| `ansible_k3s_server_group`       | Defines the `server` ansible group to pull information from                                        |
| `ansible_k3s_server_env_file`    | A list of environment variables to be added to `k3s.service.env`                                   |
| `ansible_k3s_service_extra_args` | A list of extra arguments to be added to the `k3s.service` execution block for extra configuration |


Example Playbook
----------------

A small example snippet to showcase the deployment of *k3s*.

``` yaml
- hosts: k3s_server
  roles:
    - role: ansible-role-k3s
  vars:
    ansible_k3s_sysctl_deploy: yes
    ansible_k3s_server_group: "k3s-server"
    ansible_k3s_daemon_mode: server

- hosts: k3s_agent
  roles:
    - role: ansible-role-k3s
  vars:
    ansible_k3s_sysctl_deploy: yes
    ansible_k3s_server_group: "k3s-server"
    ansible_k3s_daemon_mode: server
```

Molecule
========

The [molecule.yml](molecule/default/molecule.yml) defines an *Ansible inventory* with one *server* and one *agent* node. Molecule is configured to use **vagrant** as a backend to create two servers, as previously mentioned.

You will need to install **vagrant** and its **molecule** dependencies.

``` text
$ molecule converge -- --diff
```

After **molecule** converges one can login to the master node.

``` text
$ molecule login -h k3s-server
```

Once logged into the **vagrant** node, one can check the cluster.

``` text
vagrant@k3s-server:~$ sudo kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k3s-server   Ready    master   49s   v1.18.8+k3s1
k3s-agent    Ready    <none>   19s   v1.18.8+k3s1
```

One can also connect from their own machine if they have `kubectl` configured.

``` text
$ molecule login -h k3s-server
vagrant@k3s-server:~$ sudo cat /etc/rancher/k3s/k3s.yaml
```

which will result in something like the following.

``` yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: some-random-long-long-string
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: some-password
    username: admin
```

Copy this to your `~/.kube/config` and run `kubectl get all --all-namespaces`.
