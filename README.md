# [Ansible Role Harbor](#harbor)

## [Example Playbook](#example-playbook)

```yaml
---
- name: Harbor installation
  hosts: all
  become: yes
  gather_facts: yes

  roles:

    # install docker and docker-compose
    - role: geerlingguy.docker
    
    # install pip and pip packages
    - role: geerlingguy.pip
      vars:
        pip_install_packages:
          - name: docker 
          # be noticed that we use community.docker.docker_compose_v2
          # which doesn't use Docker SDK for Python

    # install Harbor
    - role: kolesaev.harbor
      vars:
        harbor_parent_dir: /data
        harbor_data_dir: /data/harbor_data
        harbor_enable_trivy: true
        harbor_external_url: "https://{{ ansible_hostname }}"
        harbor_admin_password: "{{ lookup('ansible.builtin.env', 'HARBOR_ADMIN_PWD') | default('Harbor12345') }}"
        harbor_http_port: 5680
        harbor_https_port: 5643
        harbor_iface_ip: 127.0.0.1
```

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/kolesaev/ansible-role-harbor/blob/master/defaults/main.yml):

```yaml
---
# defaults file for Harbor

# What version would you like to install?
harbor_version: "2.10.0"

# What type of installation would you like, either "online" or "offline".
harbor_installation_type: online

# Specify the IP address or the fully qualified domain name (FQDN) of the target host on which to deploy Harbor.
harbor_hostname: "{{ ansible_hostname }}"

# Set an initial password for the Harbor system administrator.
harbor_admin_password: "Harbor12345"

# Where to create Harbor directory
harbor_parent_dir: /home

# Where to store data
harbor_data_dir: "{{ harbor_parent_dir }}/harbor/data"

# Use https
# TODO: deprecation notice
harbor_use_https: true

# Http port
harbor_http_port: 80

# Https port
harbor_https_port: 443

# Install with trivy or not
harbor_enable_trivy: false

# Fill harbor_external_url if you want to enable external proxy.
# Use either harbor_hostname or harbor_external_url
harbor_external_url: ""

# Define an interface for open port if we enable external proxy
harbor_iface_ip: ""
```

## [Requirements](#requirements)

### [Remote OS Packages](#remote-os-packages)

Python-pip (you are able to install it manually or use any ansible playbook/task/role, e.g.  [`geerlingguy.pip`](https://github.com/geerlingguy/ansible-role-pip))

Docker and docker-compose (you are able to install them manually or use any ansible playbook/task/role, e.g.  [`geerlingguy.docker`](https://github.com/geerlingguy/ansible-role-docker))

### [Local Ansible Collections](#local-ansible-collections)

[`community.general`](https://docs.ansible.com/ansible/latest/collections/community/general/index.html)
```
ansible-galaxy collection install community.general
```

[`community.crypto`](https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html)
```
ansible-galaxy collection install community.crypto
```

[`community.docker`](https://docs.ansible.com/ansible/latest/collections/community/docker/index.html)
```
ansible-galaxy collection install community.docker
```
