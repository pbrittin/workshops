# Exercise 2.1: Using a combination of modules to delete configuration on the BIG-IP

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Playbook Output](#playbook-output)
- [Solution](#solution)
- [Verifying the Solution](#verifying-the-solution)

# Objective

Demonstrate use of the different modules to delete the configuration (Nodes/Pool/Virtual Server) on the BIG-IP.
# Guide

## Step 1:

Using VSCode create a new file called `bigip-delete-configuration.yml` by clicking the new file icon in the left pane.

![picture of create file icon](../1.1-get-facts/images/vscode-openfile_icon.png)


## Step 2:

Enter the following play definition into `bigip-delete-configuration.yml`:

{% raw %}
``` yaml
---
- name: BIG-IP TEARDOWN
  hosts: lb
  connection: local
  gather_facts: false
```
{% endraw %}
- The `---` at the top of the file indicates that this is a YAML file.
- The `hosts: f5`,  indicates the play is run only on the F5 BIG-IP device
- `connection: local` tells the Playbook to run locally (rather than SSHing to itself)
- `gather_facts: false` disables facts gathering.  We are not using any fact variables for this playbook.

## Step 3

Add a tasks section with a set_fact for setting the provider values

{% raw %}
``` yaml
  tasks:
    - name: Setup provider
      set_fact:
        provider:
          server: "{{private_ip}}"
          user: "{{ansible_user}}"
          password: "{{ansible_password}}"
          server_port: 8443
          validate_certs: false
```
{% endraw %}

## Step 4

Next, add the first `task` using the [bigip_virtual_server](https://docs.ansible.com/ansible/latest/modules/bigip_virtual_server_module.html).  This task will be identical to [Exercise 1.5 - Adding a virtual server](../1.5-add-virtual-server/README.md) with an additional **state** parameter.  The `state: absent` will remove the configuration from the F5 BIG-IP load balancer.

{% raw %}
``` yaml
    - name: DELETE VIRTUAL SERVER
      f5networks.f5_modules.bigip_virtual_server:
        provider: "{{provider}}"
        name: "vip"
        state: absent
```
{% endraw %}
- `state: absent` is a parameter that tells the module to delete the configuration

## Step 5

Next, add the second `task` using the [bigip_pool](https://docs.ansible.com/ansible/latest/modules/bigip_pool_module.html).  This task will be identical to [Exercise 1.3 - Adding a load balancing pool](../1.3-add-pool/README.md) with an additional **state** parameter set to `absent`.

{% raw %}
```yaml
    - name: DELETE POOL
      f5networks.f5_modules.bigip_pool:
        provider: "{{provider}}"
        name: "http_pool"
        state: absent
```
{% endraw %}

## Step 6

Finally, add the last `task` using the [bigip_node](https://docs.ansible.com/ansible/latest/modules/bigip_node_module.html).  This task will be identical to [Exercise 1.2 - Adding nodes to F5 BIG-IP](../1.2-add-node/README.md) with an additional **state** parameter set to `absent`.

{% raw %}
```yaml
    - name: DELETE NODES
      f5networks.f5_modules.bigip_node:
        provider: "{{provider}}"
        name: "{{hostvars[item].inventory_hostname}}"
        state: absent
      loop: "{{ groups['web'] }}"
```
{% endraw %}

Save File.

## Step 7
The playbook will delete the virtual server, then the pool and then the nodes configured in previous exercises.

Run the playbook - Go back to the Terminal on VS Code server and execute the following:

{% raw %}
```
[student1@ansible ~]$ ansible-navigator run bigip-delete-configuration.yml --mode stdout
```
{% endraw %}

# Playbook Output

{% raw %}
```
[student1@ansible]$ ansible-navigator run bigip-delete-configuration.yml --mode stdout

PLAY [BIG-IP TEARDOWN] ********************************************************

TASK [Setup provider] *********************************************************
ok: [f5]

TASK [DELETE VIRTUAL SERVER] **************************************************
changed: [f5]

TASK [DELETE POOL] ************************************************************
changed: [f5]

TASK [DELETE NODES] ***********************************************************
changed: [f5] => (item=node1)
changed: [f5] => (item=node2)

PLAY RECAP ********************************************************************
f5                         : ok=4    changed=3    unreachable=0    failed=0

```
{% endraw %}

# Solution

The finished Ansible Playbook is provided here for an Answer key. Click here: [bigip-delete-configuration.yml](https://github.com/network-automation/linklight/blob/master/exercises/ansible_f5/2.1-delete-configuration/bigip-delete-configuration.yml).

# Verifying the Solution

Login to the F5 with your web browser to see what was configured.  Grab the IP information for the F5 load balancer from the lab_inventory/hosts file, and type it in like so: https://X.X.X.X:8443/

Login information for the BIG-IP:
- username: admin
- password: **provided by instructor defaults to ansible**

Navigate the menu on the left and view that the configuration has been deleted
* Local Traffic Manager -> Virtual Server
* Local Traffic Manager -> Pool
* Local Traffic Manager -> Node

You have finished this exercise.  

[Click here to return to the lab guide](../README.md)
