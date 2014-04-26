# Canvas LMS Cloud

Automatically configure, deploy, and host the Canvas LMS on everything from a horizontally scaling, fault-tolerant, load balanced private cloud to a simple local virtual machine instance.

---

## Features
* High availability, fault tolerant, and horizontal scaling design
* Nginx used as an SSL terminator, static cache, and a round-robin load balancer
* Redis leveraged for improved system performance
* SPDY protocol support
* Handling for multiple physical datacenter setups
* Handling for multiple application servers with round robin load balancing
* Handling for multiple database servers (PostgreSQL clustering)
* Handling for multiple reverse proxies, SSL terminators, static caches, load balancers
* Handling for dedicated application server(s) to run automated jobs
* No dedicated hardware firewalls or load balancers needed
* Security hardening at multiple levels

## Server Targeting
Horizontal scaling concepts are used heavily (e.g. numerous servers per role, ability to dynamic add servers per role, load balancing per role, removing single points of failure per role, etc). With this in mind, we specifically target servers with roughly the following specifications:

```
CPU: Intel Xeon E3-1245v2
DISK: Intel SSDs in RAID 1/10 or 10k SATA HDDs in RAID 1/10
RAM 32 GB ECC
NIC: 1 Gbps
IPv4: 1 public
```

These types of servers can be custom built cost-effectively (i.e. $1,200) and leased in large quantities cost-effectively (e.g. $65/month at [SoYouStart](http://www.soyoustart.com/us/offers/sys-e32-4.xml)).

## Cloud Sizes
__Small__
* __Who:__ Targeted at local installations and small organizations.
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache, and application role servers. Supports a single vertically scaling database role server which can leverage a failover sister.

__Medium:__
* __Who:__ Targeted at medium organizations or larger organizations with read-heavy workloads.
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache, and application role servers. Supports [Pgpool II with streaming replication](http://www.pgpool.net/).

__Large:__
* __Who:__ Targeted at large organizations or smaller organizations with write-heavy workloads.
* __Scalability:__ Supports multiple datacenter setups. Supports horizontal scaling of proxy, cache, and application role servers. Supports [Postgres-XC](https://wiki.postgresql.org/wiki/Postgres-XC).

## Firewall Design

__Co-Hosts:__
* All ports and protocols are open. Co-host IP addresses are automatically discovered via the [Ansible inventory](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/production). This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

__Administrators:__
* All ports and protocols are open. If your IP address is explicitly defined in [group_vars](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/group_vars/all) you are considered an administrator. This is enforced via [iptables rules for all hosts](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/iptables/rules.v4.j2).

__Public:__
* TCP port 22 is open on all servers. However, only SSH key authentication is allowed which is enforced via [sshd_config](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/ssh/sshd_config.j2).
* ICMP echo requests are allowed to all servers. However, there is a threshold of 5 per second which is enforced via [sysctl.conf](https://github.com/rockymadden/canvas-lms-cloud/blob/master/src/ansible/roles/common/templates/etc/sysctl.conf.j2).
* TCP ports 80 and 443 are open on proxy servers and __are the only ports intended for public consumption.__ Via Ansible inventory variables, it is possible to rate-limit connections to help protect against abuse and attacks.

## Requirements

Ansible installed locally and one or more Ubuntu 14.04 LTS servers with:
* OpenSSH server installed
* Network interface and hostname configured
* SSH key transferred

## Usage

Configure and deploy all production servers:
```
$ ansible-playbook -i production-small site.yml
```

---

Configure and deploy all development servers:
```
$ ansible-playbook -i development-small site.yml
```

---

Configure and deploy just production application servers:
```
$ ansible-playbook -i production-small application.yml
```

---

Configure and deploy just production cache servers:
```
$ ansible-playbook -i production-small cache.yml
```

---
Configure and deploy just production database servers:
```
$ ansible-playbook -i production-small database.yml
```

---
Configure and deploy just production proxy servers:
```
$ ansible-playbook -i production-small proxy.yml
```

---

Perform apt maintenance on all production servers:
```
$ ansible-playbook -i production-small apt.yml
```

---

Perform apt maintenance on all production servers without rebooting:
```
$ ansible-playbook -i production-small apt.yml --skip-tags=reboot
```

---

Perform apt maintenance on all production application servers:
```
$ ansible-playbook -i production-small apt.yml --limit=application
```

---

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