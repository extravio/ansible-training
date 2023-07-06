# Ansible Training

## Inventories

Grouping tipically by
- function (database, web server,etc.)
- os
- location
- lifecycle (dev, prod, etc.)

```bash
# Check host belong to inventory
> ansible-navigator inventory -m stdout --host washington1.example.com
# List all hosts
> ansible-navigator inventory -m stdout --list
# List all hosts in a group
> ansible-navigator inventory -m stdout --graph canada
```

## Playbooks

Multiple playbooks: don't do it. If you really need to, use "import-playbook"

```yml
# wrapper.xml
---
- import_playbook: file1.yml
- import_playbook: file2.yml
```

### Loops

```yml
- name: Firewall enabled and running
  ansible.builtin.systemd:
    name: "{{ item }}"
    enabled: true
    state: started
  loop: 
  - httpd
  - firewalld
```
### Delegate

We test the connection from workstation, but "hosts" are the target machines in the inventory.

(As opposed to host being the machine from where we would test the connection)

```yml
- name: Test intranet web server
  hosts: web # localhost
  become: false
  gather_facts: true
  
  tasks:
  - name: Connect to intranet web server
    ansible.builtin.uri:
      url: http://{{ ansible_facts.fqdn }}
      return_content: true
      status_code: 200
    delegate_to: workstation
```

## Variables

Create folder / files with variable definition:
- host_vars/<host_name>/<file_name>[.yml]
- group_vars/<group_name>/<file_name>[.yml]

Variables can be used in variable files
```yaml
web_pkg: httpd

packages:
- "{{ web_pkg }}"
- firewalld
```

## Vault

```bash
ansible-navigator --run -m stdout demo.yml vault-id @prompt
# OR using password file
ansible-navigator --run -m stdout demo.yml vault-id @pass.vault 
```

## Facts

Disable fact gathering unless you're explicitly using facts. Modules *should* not be using facts (check docs).

```yaml
# If fact gathering not needed:
gather_facts: false
# OR gather a subset
gather_subset: ...
```

Custom Facts: no real use case.

## Loop

Can use objects in loop

```yaml
- name: Postfix and Dovecot are running
  ansible.builtin.service:
    name: "{{ item.name }}"
    state: "{{ item.state }}"
  loop: 
  - name: posfix
    state: started
  - name: dovecot
    state: started
```

## Handler

Notify by handler name or handler listener.

Execute handler if the task status is **changed** (not if ok or failed).

## Roles

> ansible-galaxy init <role>

Use Jinja to make variables mandatory:

```yml
system_owner: {{ user@host.example.com | mandatory }}
```