tenhishadow.manage_bind
=========
[![Build Status](https://travis-ci.com/tenhishadow/manage_bind.svg?branch=master)](https://travis-ci.com/tenhishadow/manage_bind)

It is an automation for configuring BIND on managed servers.
Bind will check config and zones befor restart.
During the execution ansible will:
  - install and configure firewalld ( if default var "firewalld" is not overrided )
  - install bind in chroot
  - configure named and zones with checking syntax with named-checkconf and named-checkzone
  - restart bind daemon in case of success

Requirements
------------
-

Example Playbook
----------------
```yaml
---

- hosts: external_dns
  become: "yes"
  become_user: "root"
  gather_facts: "true"
  any_errors_fatal: "true"
  serial: "1" 			# do it one by one to prevent fail on all BIND-servers
  vars:
    - configs: configs
  vars_files:
    - configs/ansible_vars.yml
  roles:
  - tenhishadow.manage_bind

...
```

Example files for playbook
----------------
```bash
.
├── configs
│   ├── ansible_vars.yml	# variables for playbook incude zones and params
│   ├── example.com.zone	# zone file with the records
│   ├── example1.com.zone	# zone file with the records
│   ├── named.j2		# central config which will be /etc/named.conf 
└── playbook.yml
```

Example content for ansible_vars.yml:
----------------
```yaml
---

zones:
  example.com:
    file: "example.com.zone"
    zone_serial: '2018052901'
    zone_refresh: '3600'
    zone_retry: '7200'
    zone_expire: '3600000'
    zone_minimum: '3600'
  example1.com:
    file: "example1.com.zone"
    zone_serial: '2018081301'
    zone_refresh: '3600'
    zone_retry: '7200'
    zone_expire: '3600000'
    zone_minimum: '3600'

...
```

Example content for named.j2
----------------
```shell
# {{ ansible_managed }}
options {
 listen-on port 53              { any; };
 listen-on-v6 port 53           { any; };
 directory                      "/var/named";
 dump-file                      "/var/named/data/cache_dump.db";
 statistics-file                "/var/named/data/named_stats.txt";
 memstatistics-file             "/var/named/data/named_mem_stats.txt";
 pid-file                       "/run/named/named.pid";
 session-keyfile                "/run/named/session.key";
 allow-query                    { any; };
 recursion                      no;
 version                        "AIX 5.1.0";
 auth-nxdomain                  no;
 dnssec-enable                  yes;
 dnssec-validation              auto;
 bindkeys-file                  "/etc/named.iscdlv.key";
 managed-keys-directory         "/var/named/dynamic";
 transfers-out                  100;
};

logging {
 channel  default_debug {
 file     "data/named.run";
 severity dynamic;
 };
};

# START ZONES
{% for key, value in zones.iteritems() %}
zone "{{ key }}" {
 type           master;
 file           "/var/named/data/{{ value.file }}";
 allow-transfer { none; };
};
{% endfor %}
# END OF ZONES
```
License
-------
GPL v 3.0

Author Information
------------------
* **[Stanislav Cherkasov](mailto:adm@tenhi.ru)** - [github](https://github.com/tenhishadow)
