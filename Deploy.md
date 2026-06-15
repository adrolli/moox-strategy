# moox Deploy

As a potential future building block for `moox Cloud` — or before that as the next stepping stone for `moox DevOps` — we could leverage existing FOSS tooling to build a Laravel/PHP-focused operations and deployment platform.

## **Core building blocks**

- [Deployer](https://deployer.org/)
  PHP Composer package for deployment orchestration without UI.
  Provides Envoyer-like features:

  - zero downtime deployments
  - release management
  - rollback support
  - SSH orchestration
  - multi-server deployments

- [Coolify](https://coolify.io/)
  Very popular FOSS project, but container/Docker based and much broader in scope.
  Not aligned with the intended SSH-first, host-native and Laravel/VPS-oriented approach.

- [OpenTofu](https://opentofu.org/) -  Infrastructure provisioning and orchestration.
  Potential responsibilities:

  - server provisioning
  - networking
  - firewalls
  - floating IPs
  - SSH key registration
  - cloud provider orchestration

- [cloud-init](https://cloud-init.io/)
  Initial server bootstrap after VM creation.
  See https://community.hetzner.com/tutorials/basic-cloud-config/de
  Potential responsibilities:

  - user creation
  - SSH keys
  - package bootstrap
  - first boot initialization

- [Ansible](https://docs.ansible.com/) or plain Bash provisioning scripts over SSH
  Responsibilities may include installation and configuration of:

  - PHP

  - Nginx

  - MariaDB/MySQL

  - Redis

  - Supervisor

  - Certbot

  - Node/NPM

  - queue workers

  - scheduler

## **Important operational considerations**

### **Security & updates**

Ubuntu security updates must be explicitly enabled:

- unattended-upgrades
- reboot handling
- update visibility
- notifications and dashboard integration

Important distinction:

- security updates may be automated
- major upgrades (PHP, DB, Node, etc.) should remain controlled/manual

### **Infrastructure & operations**

Mind features like:

- monitoring
- alerts
- backups & restore workflows
- load balancers
- CDN / WAF and DDoS protection
- SSL lifecycle management
- health checks
- logs

### **Remote operations**

Potential features:

- remote command execution
- web terminal
- SSH session management
- daemon/process management
- queue worker control

Requires careful consideration regarding:

- permissions
- audit logs
- security
- abuse prevention

## **Conceptual architecture**

```text
Provider (optional)
    ↓
Server
    - Provider -> Instance
    - Existing (eg. shared hosting)
    ↓
Platform layer
    - SSH keys
    - Domains (dev / custom)
    - SSL certs
    ↓
Deployment
    - releases
    - rollback
    - health
```
