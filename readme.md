# Canvas LMS Cloud <sub><sup>v0.0.2</sup></sub>

Automatically configure, deploy, and host the Canvas LMS on everything from a horizontally scaling, fault-tolerant, load balanced private cloud to a simple local virtual machine instance.

## Features
* High availability, fault tolerant, and horizontal scaling design
* No dedicated hardware firewalls or load balancers needed
* Nginx used as an SSL terminator, static cache, round-robin load balancer, with SPDY protocol support
* Redis leveraged for improved system performance
* Cassandra leveraged for improved system performance
* Handling for multiple physical datacenter setups
* Handling for multiple application hosts with round robin load balancing
* Handling for database clusters (e.g. Pgpool II and Postgres-XC)
* Handling for storage clusters (e.g. GlusterFS) or Amazon S3
* Handling for multiple reverse proxies, SSL terminators, static caches, load balancers
* Handling for dedicated application hosts to run automated jobs
* Security hardening at multiple levels

## Cloud Sizes

### Localhost ![Development Status](http://img.shields.io/badge/status-available-brightgreen.svg)
---
* __Available roles:__ proxy, application, cache, database
* __Scalability:__ N/A
* __Inventory files:__ [localhost](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/localhost)


![Diagram](https://googledrive.com/host/0B3I6erHNlT16MXUwY3R2WHJpZEU/canvas-lms-cloud-localhost.png)

### Small ![Development Status](http://img.shields.io/badge/status-wip-yellow.svg)
---
* __Available roles:__ proxy, application, cache, database master, database slave
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports a master database host with a warm standby. Supports file storage via Amazon S3.
* __Inventory files:__ [production_small](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/production_small), [development_small](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development_small), [test_small](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/test_small)

![Diagram](https://googledrive.com/host/0B3I6erHNlT16MXUwY3R2WHJpZEU/canvas-lms-cloud-small.png)

### Medium ![Development Status](http://img.shields.io/badge/status-queued-lightgrey.svg)
---
* __Available roles:__ proxy, application, cache, database coordinator, database master, database slave, storage
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports [Pgpool II with streaming replication](http://www.pgpool.net/). Supports horizontal file storage scaling via [GlusterFS](http://www.gluster.org/).
* __Inventory files:__ [production_medium](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/production_medium), [development_medium](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development_medium), [test_medium](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/test_medium)

### Large ![Development Status](http://img.shields.io/badge/status-queued-lightgrey.svg)
---
* __Available roles:__ proxy, application, cache, database coordinator, database datanode, database global transaction manager, storage
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports [Postgres-XC](https://wiki.postgresql.org/wiki/Postgres-XC). Supports horizontal file storage scaling via [GlusterFS](http://www.gluster.org/).
* __Inventory files:__ [production_large](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/production_large), [development_large](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development_large), [test_large](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/test_large)

## Server Targeting
Horizontal scaling concepts are used heavily (e.g. numerous servers per role, ability to dynamic add servers per role, load balancing per role, removing single points of failure per role, etc). With this in mind, we specifically target servers with roughly the following specifications:

```
CPU: Intel Xeon E3-1245v3
DISK: Intel SSDs or 10k enterprise-level SATA HDDs in RAID 1/10
RAM 32 GB ECC
NIC: 1 Gbps (preferably dual)
IPv4: 1 public
```

These types of servers can be custom built cost-effectively (i.e. $1,200) and leased in large quantities cost-effectively (e.g. $65/month at [SoYouStart](http://www.soyoustart.com/us/offers/sys-e32-4.xml)).

## Firewall Design

All publicly available ports/protocols have IP based rate-limiting to help mitigate abuse and attacks (e.g. if you ping more than once a second, your requests are dropped). For details, check out the [iptables rules template](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

### Co-Hosts
---
* All ports and protocols are open. Co-host IP addresses are automatically discovered via the Ansible inventory. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

### Administrators
---
* All ports and protocols are open. If your IP address is explicitly defined in [group_vars](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/group_vars/all.example), you are considered an administrator. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

### Public
---
* ICMP echo requests are allowed to all hosts.
* TCP port 22 is open on all hosts. However, only SSH key authentication is allowed which is enforced via [sshd_config](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/ssh/sshd_config.j2).
* TCP ports 80 and 443 are open on proxy hosts and __are the only ports intended for public consumption.__

## Usage

All examples use a small cloud size. Be sure to adjust your commands based upon your own implementation size.

---

Copy your public SSH key to a new server (the first playbook run against any host):
```
$ ansible-playbook -i small_production util_ssh_copy_id.yml --user=username --ask-pass --ask-sudo-pass
```

---

Configure and deploy all production hosts:
```
$ ansible-playbook -i small_production small.yml
```

---

Configure and deploy all development hosts:
```
$ ansible-playbook -i small_development small.yml
```

---

Configure and deploy just production application hosts:
```
$ ansible-playbook -i small_production host_application.yml
```

---

Configure and deploy just production application hosts while only running ruby based tasks:
```
$ ansible-playbook -i small_production host_application.yml --tags ruby
```

---

Perform apt maintenance on all production hosts:
```
$ ansible-playbook -i small_production util_apt_upgrade.yml
```

---

Perform apt maintenance on all production hosts without rebooting:
```
$ ansible-playbook -i small_production util_apt_upgrade.yml --skip-tags=reboot
```

---

Perform apt maintenance on all production application hosts:
```
$ ansible-playbook -i small_production util_apt_upgrade.yml --limit=application
```

---

## Requirements

Ansible installed locally and one or more Ubuntu 14.04 LTS hosts with:
* OpenSSH server installed
* SSH key transferred

## License

```
The MIT License (MIT)

Copyright (c) 2014 Rocky Madden (http://rockymadden.com/)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```