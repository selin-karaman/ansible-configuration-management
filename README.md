# Ansible Configuration Management

A hands-on Ansible project demonstrating centralized configuration management across multiple Linux nodes in a local Docker environment.

The project uses Ansible roles, inventories, variables, Jinja2 templates, handlers, and idempotent playbooks to configure and manage three Ubuntu nodes consistently.

## Features

- Multi-node configuration management with Ansible
- SSH-based node management
- Role-based Ansible structure
- Shared variables with `group_vars`
- Package installation and user management
- Directory provisioning
- Nginx installation and service management
- Dynamic configuration with Jinja2 templates
- Handlers for conditional service restarts
- Idempotent playbook execution
- Local lab environment with Docker Compose

## Architecture

```text
                    Ansible Control Machine
                           |
              -----------------------------
              |             |             |
              v             v             v
           node1          node2          node3
           Ubuntu         Ubuntu         Ubuntu
              |             |             |
              v             v             v
            Nginx         Nginx         Nginx
```

The local machine acts as the Ansible control node, while three Docker-based Ubuntu containers act as managed nodes.

Ansible connects to each node over SSH and applies the same desired configuration.

## Project Structure

```text
ansible-configuration-management/
│
├── inventory/
│   └── hosts.ini
│
├── group_vars/
│   └── all.yml
│
├── roles/
│   ├── common/
│   │   ├── files/
│   │   └── tasks/
│   │       └── main.yml
│   │
│   └── nginx/
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       └── templates/
│           └── nginx.conf.j2
│
├── playbook.yml
├── ansible.cfg
├── docker-compose.yml
├── .gitignore
└── README.md
```

## How It Works

The Docker Compose environment creates three Ubuntu containers that can be managed through SSH.

The Ansible inventory defines these containers as managed nodes:

```text
node1 → localhost:2221
node2 → localhost:2222
node3 → localhost:2223
```

The main playbook applies two roles to all nodes.

### Common Role

The `common` role performs baseline Linux configuration:

- updates the package cache
- installs common packages such as `curl`, `vim`, `unzip`, and `htop`
- creates a `devops` user
- creates and configures `/opt/app`

### Nginx Role

The `nginx` role:

- installs Nginx
- deploys a custom web page using a Jinja2 template
- ensures the Nginx service is running
- restarts Nginx only when the deployed template changes

The template uses `inventory_hostname`, allowing the same template to generate node-specific output such as:

```text
Hello from node1
Hello from node2
Hello from node3
```

## Requirements

- Docker
- Docker Compose
- Ansible

## Usage

Start the managed nodes:

```bash
docker compose up -d
```

Verify that the containers are running:

```bash
docker compose ps
```

Test Ansible connectivity:

```bash
ansible all -m ping
```

Expected result:

```text
node1 | SUCCESS
node2 | SUCCESS
node3 | SUCCESS
```

Run the configuration playbook:

```bash
ansible-playbook playbook.yml
```

## Idempotency

Running the playbook for the first time applies the required configuration and reports changed resources.

Running the same playbook again should result in:

```text
changed=0
```

for all managed nodes if no configuration has changed.

This demonstrates Ansible's idempotent configuration management model.

## Verification

Each node exposes its Nginx service through a different local port:

```text
node1 → localhost:8081
node2 → localhost:8082
node3 → localhost:8083
```

Verify the deployed pages:

```bash
curl http://localhost:8081
curl http://localhost:8082
curl http://localhost:8083
```

Each node should return a page containing its own hostname.

Expected output:

```text
<h1>Hello from node1</h1>
<h1>Hello from node2</h1>
<h1>Hello from node3</h1>
```

## Key Takeaways

This project helped me strengthen my practical understanding of:

- managing multiple Linux nodes from a central Ansible control machine
- organizing Ansible automation with inventories, roles, tasks, and variables
- using Jinja2 templates for dynamic configuration
- triggering handlers only when configuration changes
- managing Linux users, packages, directories, and services declaratively
- verifying idempotent configuration management behavior
- combining Docker and Ansible to build a repeatable local infrastructure lab
