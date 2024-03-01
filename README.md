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

        harbor_projects_to_remove:
          - library

        harbor_users:
          - name: "user-one"

        harbor_registries:
          - url: "https://hub.docker.com"
            name: "docker-hub"
            type: "docker-hub"

        harbor_projects:

          - name: "new-project"
            auto_scan: yes
            public: no
            retention_policy:
              schedule: Hourly
              rules:
                - repositories_mask_type: "matching"
                  repositories_mask: "**"
                  tags_mask_type: "matching"
                  tags_mask: "dev-*"
                  rule_type: "pushed_by_days"
                  rule_value: 30

          - name: "docker-hub"
            public: yes
            proxy_registry: "docker-hub"

        harbor_members:
          - project: "new-project"
            name: "user-one"
            role_id: 2
```

## [Role Variables](#role-variables)

You should be able to use any boolean variables (yes/no/true/false).

The default values for the variables are set in [`defaults/main.yml`](https://github.com/kolesaev/ansible-role-harbor/blob/master/defaults/main.yml).

```yaml
---
# Should we disable unattended-upgrades service
system_disable_unattended_upgrades: false

# Version you'd like to install
harbor_version: "2.10.0"

# Should we force install Harbor even if installed
harbor_force_install: false

# What type of installation would you like, either "online" or "offline".
harbor_installation_type: online

# Specify the IP address or the fully qualified domain name (FQDN) of the target host on which to deploy Harbor.
harbor_hostname: "{{ ansible_host }}"

# Set an initial password for the Harbor system administrator.
harbor_admin_password: "Harbor12345"

# Where to create Harbor directory
harbor_parent_dir: /home

# Where to store data
harbor_data_dir: "{{ harbor_parent_dir }}/harbor/data"

# Http port
harbor_http_port: 80

# Https port
harbor_https_port: 443

# Install with trivy or not
harbor_enable_trivy: false

# Fill harbor_external_url if you want to enable external proxy.
harbor_external_url: ""

# We are able to define an interface IP for open port when we enable external proxy
harbor_iface_ip: "" 
  ###########################################################
  #####                                                 #####
  ##### Interface IP can be set once using this role,   #####
  ##### it will not be updated after changing the value #####
  ##### but only until you delete the indicator file    #####
  ##### or use harbor_force_install variable            #####
  #####                                                 #####
  ###########################################################

#################
### API stuff ###
#################

# Are admins only able to create projects
harbor_project_creation_restriction: false

# Remove registry list
harbor_registries_to_remove: []
  # - registry-name

# Remove member list
harbor_members_to_remove: []
  # - project: project-name
  #   member_id: member-id
  ##################################################
  ##### You are able to get member IDs via API #####
  #####     But easier to remove them via UI   #####
  ##################################################

# Remove project list
harbor_projects_to_remove: []
  # - project-name

# Remove user list
harbor_users_to_remove: []
  # - user-name

# Create project list
harbor_projects: []
  # - name: "project-name"
  #   public: no
  #   auto_scan: false
  #   proxy_registry: registry-name
  #   # tag retention policy
  #   retention_policy:
  #     schedule: "0 0 * * * *" # accepts cron and value Hourly/Daily
  #     rules:
  #       - repositories_mask_type: "matching"
  #         repositories_mask: "**"
  #         tags_mask_type: "matching"
  #         tags_mask: "dev-*"
  #         rule_type: "pushed_by_days"
  #         rule_value: 30
  #         with_untagged: false
  ####################################################
  #####                                          #####
  ##### Mapping for retention rules              #####
  #####                                          #####
  ##### rule_types:                              #####
  #####   pushed_by_count: "latestPushedK"       #####
  #####   pulled_by_count: "latestPulledN"       #####
  #####   pushed_by_days:  "nDaysSinceLastPush"  #####
  #####   pulled_by_days:  "nDaysSinceLastPull"  #####
  #####   always:          "always"              #####
  ##### repositories_mask_types:                 #####
  #####   matching: "repoMatches"                #####
  #####   excluding: "repoExcludes"              #####
  ##### tags_mask_types:                         #####
  #####   matching: "matches"                    #####
  #####   excluding: "excludes"                  #####
  #####                                          #####
  ####################################################

# Create user list
harbor_users: []
  # - name: "user"
  #   email: "email@example.com"
  #   password: "Harbor12345" # The password can be set once using this role, it will not be updated after changing the value
  #   realname: "user"
  #   comment: "comment"

# Create project members
harbor_members: []
  # - project: "project-name"
  #   name: "user-name"
  #   role_id: 1
  #############################
  ##### Role IDs:         #####
  #####                   #####
  ##### Project Admin = 1 #####
  ##### Developer     = 2 #####
  ##### Guest         = 3 #####
  ##### Maintainer    = 4 #####
  ##### Limited Guest = 5 #####
  #############################

# Create mirror registries
harbor_registries: []
  # - url: "https://hub.docker.com"
  #   insecure: false
  #   name: "Docker Hub"
  #   type: "docker-hub"
  #   auth_type: basic
  #   access_key: name
  #   access_secret: password

# Configuring garbage collector
harbor_gc: {}
  # type: Custom # will be custom if empty
  # schedule: 0 0 1 * * *
  # delete_untagged: false
  # workers: 2
  #######################################
  #####                             #####
  ##### By default it will be with  #####
  #####                             #####
  ##### type: Custom                #####
  ##### schedule: 0 0 0 * * *       #####
  ##### worker: 1                   #####
  ##### delete_untagged: false      #####
  #####                             #####
  ##### To disable set type None    #####
  #####                             #####
  ##### Available types:            #####
  #####  - None                     #####
  #####  - Weekly                   #####
  #####  - Daily                    #####
  #####  - Hourly                   #####
  #####                             #####
  #######################################
```

## [Reinstall](#reinstall)

As this role will create an indicator file it will not be installed and configured again after this role was already used. To reinstall Harbor you are able to delete file *installed-indicator* in harbor directory or set *true* as a value for *harbor_force_install* variable

API actions will be performed every time this role is used, even if the indicator file exists

## [Features](#features)

- Installing Harbor

- Creating/Updating/Configuring via API:
  - Projects
  - Registries
  - Users
  - Members
  - Retention Policies
  - Garbage Collector

- API related features:
  - Each object will be updated each time it's in related creation list
  - It won't remove an object if it's also in related creation list. To recreate it, first remove it from related creation list, then add again.

## [Requirements](#requirements)

### [Remote OS Packages](#remote-os-packages)

Docker and docker-compose-plugin (you are able to install them manually or use any ansible playbook/task/role, e.g.  [`geerlingguy.docker`](https://github.com/geerlingguy/ansible-role-docker))

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

## [Not implemented](#not-implemented)

- For installation:
  - ability to use an external database
  - ability to use an external redis
  - ability to use an external syslog server
  - ability to use an external tracing
  - ability to open metrics endpoint
  - (and other things which are not configurable via default variables)

- For API:
  - CVE allow lists
  - job services
  - purges
  - replications
  - robots
  - user groups
  - webhooks
  - immutable tag rules
  - preheats
  - external auth providers
  - (and other things which are not configurable via default variables)

## [Used solutions](#used-solutions)

Many thanks to the following projects that helped me in the implementation

[`robertdebock.harbor`](https://galaxy.ansible.com/ui/standalone/roles/robertdebock/harbor/)

[`one_mind.harbor_ansible_role`](https://galaxy.ansible.com/ui/standalone/roles/one_mind/harbor_ansible_role/)
