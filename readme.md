# Canvas LMS Cloud

Automatically configure, deploy, and host the Canvas LMS on everything from a horizontally scaling, fault-tolerant, load balanced private cloud to a simple local virtual machine instance.

---

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
* Handling for email hosts (e.g. sendmail)
* Handling for multiple reverse proxies, SSL terminators, static caches, load balancers
* Handling for dedicated application hosts to run automated jobs
* Security hardening at multiple levels

## Cloud Sizes
__Localhost:__ ![Development Status](http://img.shields.io/badge/status-wip-yellow.svg)
* __Available roles:__ proxy, application, cache, database
* __Scalability:__ N/A
* __Inventory files:__ [localhost](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/localhost)

__Small:__ ![Development Status](http://img.shields.io/badge/status-wip-yellow.svg)
* __Available roles:__ proxy, application, cache, database master, database slave
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports a master database host with a hot standby. Supports file storage via Amazon S3.
* __Inventory files:__ [production-small](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development-small), [development-small](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development-small), [test-small](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/test-small)

__Medium:__ ![Development Status](http://img.shields.io/badge/status-queued-lightgrey.svg)
* __Available roles:__ proxy, application, cache, database coordinator, database master, database slave, email, storage
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports [Pgpool II with streaming replication](http://www.pgpool.net/). Supports horizontal file storage scaling via [GlusterFS](http://www.gluster.org/).
* __Inventory files:__ [production-medium](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development-medium), [development-medium](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development-medium), [test-medium](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/test-medium)

__Large:__ ![Development Status](http://img.shields.io/badge/status-queued-lightgrey.svg)
* __Available roles:__ proxy, application, cache, database coordinator, database datanode, database global transaction manager, email, storage
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports [Postgres-XC](https://wiki.postgresql.org/wiki/Postgres-XC). Supports horizontal file storage scaling via [GlusterFS](http://www.gluster.org/).
* __Inventory files:__ [production-large](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development-large), [development-large](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/development-large), [test-large](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/test-large)

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

__Co-Hosts:__
* All ports and protocols are open. Co-host IP addresses are automatically discovered via the Ansible inventory. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

__Administrators:__
* All ports and protocols are open. If your IP address is explicitly defined in [group_vars](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/group_vars/all) you are considered an administrator. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

__Public:__
* TCP port 22 is open on all hosts. However, only SSH key authentication is allowed which is enforced via [sshd_config](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/ssh/sshd_config.j2).
* ICMP echo requests are allowed to all hosts. However, there is a threshold of 5 per second which is enforced via [sysctl.conf](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/sysctl.conf.j2).
* TCP ports 80 and 443 are open on proxy hosts and __are the only ports intended for public consumption.__ Via Ansible inventory variables, it is possible to rate-limit connections to help protect against abuse and attacks.

## Usage

Configure and deploy all production hosts:
```
$ ansible-playbook -i localhost site.yml
$ ansible-playbook -i production-small site.yml
$ ansible-playbook -i production-medium site.yml
$ ansible-playbook -i production-large site.yml
```

---

Configure and deploy all development hosts:
```
$ ansible-playbook -i localhost site.yml
$ ansible-playbook -i development-small site.yml
$ ansible-playbook -i development-medium site.yml
$ ansible-playbook -i development-large site.yml
```

---

Configure and deploy just production application hosts:
```
$ ansible-playbook -i localhost application.yml
$ ansible-playbook -i production-small application.yml
$ ansible-playbook -i production-medium application.yml
$ ansible-playbook -i production-large application.yml
```

---

Configure and deploy just production application hosts while only running ruby based tasks:
```
$ ansible-playbook -i localhost application.yml --tags ruby
$ ansible-playbook -i production-small application.yml --tags ruby
$ ansible-playbook -i production-medium application.yml --tags ruby
$ ansible-playbook -i production-large application.yml --tags ruby
```

---

Configure and deploy just production cache hosts:
```
$ ansible-playbook -i localhost cache.yml
$ ansible-playbook -i production-small cache.yml
$ ansible-playbook -i production-medium cache.yml
$ ansible-playbook -i production-large cache.yml
```

---
Configure and deploy just production database hosts:
```
$ ansible-playbook -i localhost database.yml
$ ansible-playbook -i production-small database.yml
$ ansible-playbook -i production-medium database.yml
$ ansible-playbook -i production-large database.yml
```

---
Configure and deploy just production proxy hosts:
```
$ ansible-playbook -i localhost proxy.yml
$ ansible-playbook -i production-small proxy.yml
$ ansible-playbook -i production-medium proxy.yml
$ ansible-playbook -i production-large proxy.yml
```

---

Perform apt maintenance on all production hosts:
```
$ ansible-playbook -i localhost apt.yml
$ ansible-playbook -i production-small apt.yml
$ ansible-playbook -i production-medium apt.yml
$ ansible-playbook -i production-large apt.yml
```

---

Perform apt maintenance on all production hosts without rebooting:
```
$ ansible-playbook -i localhost apt.yml --skip-tags=reboot
$ ansible-playbook -i production-small apt.yml --skip-tags=reboot
$ ansible-playbook -i production-medium apt.yml --skip-tags=reboot
$ ansible-playbook -i production-large apt.yml --skip-tags=reboot
```

---

Perform apt maintenance on all production application hosts:
```
$ ansible-playbook -i localhost apt.yml --limit=application
$ ansible-playbook -i production-small apt.yml --limit=application
$ ansible-playbook -i production-medium apt.yml --limit=application
$ ansible-playbook -i production-large apt.yml --limit=application
```

---

## Requirements

Ansible installed locally and one or more Ubuntu 14.04 LTS hosts with:
* OpenSSH server installed
* Network interface and hostname configured
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