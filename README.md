Flannel
=========

Ansible role to install flannel to Ubuntu 14.04!

This role copy the flannel release into remote host,
unpack it, and then config it with backend etcd, and start it.

That's it, simple and native.

Requirements
------------

None

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

``` conf
# defaults file for flannel

ansible_temp_dir: /tmp/.ansible/files

# the local dir that contain the release files of flannel.
flannel_release_dir: ./releases/flannel
flannel_version: 0.6.1

# flannel name
flannel_cluster_name: "flannel_default"

# group name that has role etcd installed.
# flannel role will use its first etcd server to load flannel config
# to etcd.
flannel_etcd_group: etcd


#
flannel_network: 172.16.0.0/12
flannel_subnet_len: 24
flannel_backend: vxlan

# a comma-delimited list of etcd endpoints
# "{% for node in groups['etcd'] %}http://{{ node }}:2379{% if not loop.last %},{% endif %}{% endfor %}"
flannel_etcd_endpoints: "{% for node in groups[flannel_etcd_group] %}http://{{ hostvars[node]['ansible_default_ipv4'].address }}:2379{% if not loop.last %},{% endif %}{% endfor %}"

# Any additional options that you want to pass, example: "--iface=eth1".
# By default, we just add a good guess for the network interface on Vbox.  Otherwise, Flannel will probably make the right guess.
flannel_opts: ''
```

**notes**

If `docker_config_file` was passed, flannel will config docker to use
flannel network, and restart docker.

Dependencies
----------------

`etcd.ansible`

flannel depends on etcd, It's why this role need etcd.

Example Playbook
----------------

``` yaml
- hosts: docker
  roles:
    - role: docker
- hosts: servers
  roles:
    - role: etcd
      etcd_release_dir: ./releases/etcd
      etcd_version: 3.0.7
      etcd_client_port: 2379
      etcd_interface: eth1 # this is needed, as we are in vagrant-virtualbox
    - role: flannel
      flannel_release_dir: ./releases/flannel
      flannel_cluster_name: "sensetime"
      flannel_version: 0.6.1
      flannel_etcd_group: etcd
      flannel_etcd_endpoints: "{% for node in groups[flannel_etcd_group] %}http://{{ hostvars[node].ansible_eth1.ipv4.address }}:2379{% if not loop.last %},{% endif %}{% endfor %}"
      flannel_opts: "-iface=eth1"
      docker_config_file: /etc/default/docker
```

License
-------

MIT
