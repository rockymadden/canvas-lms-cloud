# Canvas LMS Cloud <sub><sup>v0.2.6</sup></sub>

Automagically configure, deploy, and host the Canvas LMS on everything from a simple local virtual machine instance to a horizontally scaling, fault-tolerant, load balanced private cloud.

## Features
* High availability, fault tolerant, and horizontal scaling design
* No dedicated hardware firewalls or load balancers needed
* Nginx used as an SSL terminator, static cache, round-robin load balancer, with SPDY protocol support
* Redis leveraged for improved system performance
* Cassandra leveraged for improved system performance
* Handling for multiple physical datacenter setups
* Handling for multiple application hosts with round robin load balancing
* Handling for relational database clusters (e.g. Pgpool II and Postgres-XC)
* Handling for storage clusters (e.g. GlusterFS) or Amazon S3
* Handling for multiple reverse proxies, SSL terminators, static caches, load balancers
* Handling for dedicated application hosts to run automated jobs
* Security hardening at multiple levels
* Playbooks to handle for routine tasks and maintenance (e.g. copying SSH keys, performing LMS upgrades, performing host upgrades, toggling maintenance mode)

## Cloud Sizes

### Localhost ![Development Status](http://img.shields.io/badge/status-available-brightgreen.svg)
---
* __Available hosts:__ proxy, application, cache, rdb
* __Inventory files:__ [localhost_production](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/localhost_production), [localhost_development](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/localhost_development), [localhost_test](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/localhost_test)

![Diagram](http://share.rockymadden.com/ADvO+)

### Small ![Development Status](http://img.shields.io/badge/status-available-brightgreen.svg)
---
* __Available hosts:__ proxy, application, cache, rdb master, rdb slave
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports a master rdb host with a warm standby. Supports file storage via Amazon S3.
* __Inventory files:__ [small_production](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/small_production), [small_development](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/small_development), [small_test](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/small_test)

![Diagram](http://share.rockymadden.com/TyYO+)

### Medium ![Development Status](http://img.shields.io/badge/status-queued-lightgrey.svg)
---
* __Available hosts:__ proxy, application, cache, rdb coordinator, rdb master, rdb slave, storage
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports [Pgpool II with streaming replication](http://www.pgpool.net/). Supports horizontal file storage scaling via [GlusterFS](http://www.gluster.org/).
* __Inventory files:__ [medium_production](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/medium_production), [medium_development](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/medium_development), [medium_test](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/medium_test)

![Diagram](http://share.rockymadden.com/Y0uW+)

### Large ![Development Status](http://img.shields.io/badge/status-queued-lightgrey.svg)
---
* __Available hosts:__ proxy, application, cache, rdb coordinator, rdb datanode, rdb global transaction manager, storage
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache (via sharding), and application hosts. Supports [Postgres-XC](https://wiki.postgresql.org/wiki/Postgres-XC). Supports horizontal file storage scaling via [GlusterFS](http://www.gluster.org/).
* __Inventory files:__ [large_production](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/large_production), [large_development](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/large_development), [large_test](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/large_test)

## Playbook Usage

All examples use a small cloud size. Be sure to adjust your commands based upon your own implementation size.

---

Copy your public SSH key to a new server (typically the first playbook run against any host):
```bash
$ ansible-playbook -i small_production util_copy_ssh_id.yml \
> --user=username --limit=hostname \
> --ask-pass --ask-sudo-pass
```

---

Configure and deploy all production hosts:
```bash
$ ansible-playbook -i small_production small.yml
```

---

Configure and deploy all test hosts:
```bash
$ ansible-playbook -i small_test small.yml
```

---

Configure and deploy all development hosts:
```bash
$ ansible-playbook -i small_development small.yml
```

---

Configure and deploy just production application hosts:
```bash
$ ansible-playbook -i small_production host_application.yml
```

---

Configure and deploy just production application hosts while only running ruby based tasks:
```bash
$ ansible-playbook -i small_production host_application.yml --tags=ruby
```

---

Make Canvas unavailable to perform maintenance:
```bash
$ ansible-playbook -i small_production util_make_canvas_unavailable.yml
```

---

Make Canvas available after performing maintenance:
```bash
$ ansible-playbook -i small_production util_make_canvas_available.yml
```

---

Perform apt maintenance on all production hosts:
```bash
$ ansible-playbook -i small_production util_perform_apt_upgrade.yml
```

---

Perform apt maintenance on all production hosts without rebooting:
```bash
$ ansible-playbook -i small_production util_perform_apt_upgrade.yml --skip-tags=reboot
```

---

Perform apt maintenance on all production application hosts:
```bash
$ ansible-playbook -i small_production util_perform_apt_upgrade.yml --limit=application
```

---

## Firewall Design

All publicly available ports/protocols have IP based rate-limiting to help mitigate abuse and attacks (e.g. if you ping more than once per second, your requests would be dropped). For details, check out the [iptables rules template](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

---

__Co-Hosts:__

All ports and protocols are open. Co-host IP addresses are automatically discovered via the Ansible inventory. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

---

__Administrators:__

All ports and protocols are open. If your IP address is explicitly defined in [group_vars](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/group_vars/all.example), you are considered an administrator. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

---

__Public:__

ICMP echo requests are allowed to all hosts. TCP port 22 is open on all hosts. However, only SSH key authentication is allowed which is enforced via [sshd_config](https://github.com/rockymadden/canvas-lms-cloud/blob/master/ansible/roles/common/templates/etc/ssh/sshd_config.j2). TCP ports 80 and 443 are open on proxy hosts and are the only ports intended for public consumption.

---

## Server Targeting
Horizontal scaling concepts are used heavily (e.g. numerous servers per role, ability to dynamic add servers per role, load balancing per role, removing single points of failure per role, etc). With this in mind, we specifically target servers with roughly the following specifications:

```
CPU: Intel Xeon E3/E5/E7 (e.g. E3-1230v3, E5-1620v2)
DISK: Intel SSDs or 10k enterprise-level SATA HDDs in RAID 1/10
RAM 32 GB ECC
NIC: 1 Gbps (preferably dual)
IPv4: 1 public
```

These types of servers can be custom built cost-effectively (i.e. $1,200) and leased in large quantities cost-effectively (e.g. ~$120/month at [OVH](https://www.ovh.com/us/dedicated-servers/infra/2014-EG-32.xml)).

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
