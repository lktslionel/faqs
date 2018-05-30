###### FAQ / ANSIBLE
# Debugging 

## 1. Active the ansible debugger for a playbook 

To activate the debugger on a playbook, add the following line to your playbook header : 

```yaml
---
  strategy: debug
```  

Yopur playbook header will look like : 

```yaml
- hosts: localhost
  gather_facts: no
  ...
  strategy: debug
  ...
  tasks:
    ...
```

When an error occurs on any tasks of your play, the `debugger` will be invoked and you will see a prompt like: 

```bash
Debugger invoked
(debug)
```

For more informations, go to the [Ansible 2.4 - Playbook Debugger doc](https://docs.ansible.com/ansible/2.4/playbooks_debugger.html)
