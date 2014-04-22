# Canvas LMS Cloud
Operate a self-hosted private [Canvas LMS](https://github.com/instructure/canvas-lms) cloud installation, powered by [Ansible](https://github.com/ansible/ansible).

## Features
* High availability, fault tolerant, and horizontal scaling design
* Nginx used as an SSL terminator, static cache, and a round-robin load balancer
* Redis leveraged for improved system performance
* Handling for multiple physical datacenter setups
* Handling for multiple application servers with round robin load balancing
* Handling for multiple database servers (PostgreSQL clustering)
* Handling for multiple reverse proxies, SSL terminators, static caches, load balancers
* Handling for dedicated application server(s) to run automated jobs
* Security hardening at multiple levels

## Requirements

One or more Ubuntu 14.04 LTS servers with:
* OpenSSH server installed
* Basic network interface and hostname configured
* SSH keys transferred
 