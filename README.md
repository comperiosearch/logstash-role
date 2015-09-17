Ansible Role to Install and Configure Logstash
=========


Requirements
------------

On Debian OS family, **python-pycurl** and **python-apt** are required to deal with apt Ansible modules. The role already take care of these dependencies, you can disable this behavior with `--skip-tags=logstash_req`.

**Java** should be present on the nodes machines in order to run Logstash. This role does not install Java.

Logstash configuration can be set in playbook input, or in a folder with files.
The files will be run through the templating engine - meaning you can use variables.


Example Playbooks
----------------

```yaml
- hosts: LogstashNodes
  roles:
    - role: logstash-role

      logstash_version: "1.5"

      logstash_defaults: |
        LS_USER=root
        LS_HEAP_SIZE="256m"

     logstash_config_files: include/logstash

     logstash_inputs: |
       syslog { host => "{{ ansible_eth0.ipv4.address }}"
                port => "514"
                type => "syslog_input"
              }

       syslog { host => "{{ ansible_lo.ipv4.address }}"
                port => "515"
                type => "syslog_input_local"
              }

     logstash_filters: |
       geoip { source => "ip_address" 
             }
 
       multiline { pattern => "^No lfn2pfn"
                   what => "previous"
                 }

     logstash_outputs: |
       file { path => "/var/log/logstash/output.log"
            }

      tags: logstash
```

Role Variables
--------------

```yaml
logstash_python_utils:
 - { package: "python-pycurl" }
 - { package: "python-apt" }

logstash_version: "none"

logstash_apt_repo: "deb http://packages.elasticsearch.org/logstash/{{ logstash_version }}/debian stable main"
logstash_repo_key: "http://packages.elasticsearch.org/GPG-KEY-elasticsearch"
logstash_yum_repo_dest: "/etc/yum.repos.d/logstash.repo"

logstash_conf_dir: "/etc/logstash/conf.d/"

logstash_defaults: 
 - { directive: "LS_USER=logstash" }

defaults_RedHat: "/etc/sysconfig/logstash"
defaults_Debian: "/etc/default/logstash"
```




## Include role in a larger playbook
### Add this role as a git submodule
Assuming your playbook structure is such as:
```
- my-master-playbook
  |- vars
  |- roles
  |- my-master-playbook-main.yml
  \- my-master-inventory.ini
```

Checkout this project as a submodule under roles:

```
$  cd roles
$  git submodule add https://github.com/comperiosearch/logstash-role.git ./logstash
$  git submodule update --init
$  git commit ./submodule -m "Added submodule as ./subm"
```

### Include this playbook as a role in your master playbook
Example `my-master-playbook-main.yml`:

```
---

#########################
# Logstash install #
#########################

- hosts: all_nodes
  user: ubuntu
  sudo: yes

  roles:
    - logstash

  vars_files:
    - vars/my-vars.yml
```


License
-------

GNU General Public License Version 2

Author Information
------------------

Valentino Gagliardi - valentino.g@servermanaged.it
Christoffer Vig  - christoffer.vig@gmail.com
