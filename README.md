<h1 align="center">DevOps Journey</h1>

<p align="center">
  A practical, structured collection of DevOps learning notes, commands, configuration examples, and hands-on labs.
</p>

<p align="center"><strong>10 Guides</strong> &nbsp;|&nbsp; <strong>Hands-on Learning</strong> &nbsp;|&nbsp; <strong>Beginner to Intermediate</strong></p>

---

## Overview

This repository brings together ten standalone guides in a suggested learning order. Each topic has its own `README.md`, so you can find commands, configuration examples, and troubleshooting notes without jumping between repositories.

**Recommended path:** Linux & Bash -> Git & GitHub -> Networking & Nginx -> Docker -> CI/CD tools -> AWS -> Terraform -> Ansible -> Kubernetes.

## Learning Roadmap

| # | Guide | What you will practice | Separate repository |
|---|---|---|---|
| 01 | [Linux & Bash Scripting](01-Linux-Bash/README.md) | Linux commands, permissions, logs, and shell scripts | [Original](https://github.com/Safayet19/Hello-Linux) |
| 02 | [Git & GitHub](02-Git-GitHub/README.md) | Commits, branches, pull requests, and recovery | [Original](https://github.com/Safayet19/Hello-Github) |
| 03 | [Networking, Nginx & Web Security](03-Networking-Nginx-Security/README.md) | DNS, ports, reverse proxy, HTTPS, and firewall | [Original](https://github.com/Safayet19/Networking-Nginx-Web-Security) |
| 04 | [Docker & Docker Compose](04-Docker-Compose/README.md) | Images, containers, Dockerfiles, Compose, and volumes | [Original](https://github.com/Safayet19/Hello-Docker) |
| 05 | [GitHub Actions](05-GitHub-Actions/README.md) | Workflow YAML, triggers, testing, and deployment | [Original](https://github.com/Safayet19/Github-Actions) |
| 06 | [Jenkins](06-Jenkins/README.md) | Pipelines, agents, builds, tests, and automation | [Original](https://github.com/Safayet19/Hello-Jenkins) |
| 07 | [Amazon Web Services (AWS)](07-AWS/README.md) | IAM, VPC, EC2, S3, monitoring, and guided labs | [Original](https://github.com/Safayet19/Hello-AWS) |
| 08 | [Terraform](08-Terraform/README.md) | HCL, providers, state, plans, and EC2 provisioning | [Original](https://github.com/Safayet19/Hello-Terraform) |
| 09 | [Ansible](09-Ansible/README.md) | Inventory, playbooks, modules, and server automation | [Original](https://github.com/Safayet19/Hello-Ansible) |
| 10 | [Kubernetes](10-Kubernetes/README.md) | Pods, deployments, services, YAML, and troubleshooting | [Original](https://github.com/Safayet19/Hello-Kubernetes) |

## How the Tools Fit Together

```text
Git & GitHub        -> Manage and version source code
GitHub Actions      -> Run automated builds and tests
Jenkins             -> Alternative CI/CD pipeline platform
Docker & Compose    -> Package and run applications in containers
AWS                 -> Provide cloud infrastructure
Terraform           -> Define infrastructure as code
Ansible             -> Configure servers and automate operations
Kubernetes          -> Deploy and manage containerized workloads
Linux + Networking  -> Foundation used throughout the workflow
```

GitHub Actions and Jenkins are alternatives for many CI/CD tasks. You do not need both in every project.

## Repository Structure

```text
DevOps-Journey/
  README.md
  01-Linux-Bash/
    README.md
  02-Git-GitHub/
    README.md
  03-Networking-Nginx-Security/
    README.md
  04-Docker-Compose/
    README.md
  05-GitHub-Actions/
    README.md
  06-Jenkins/
    README.md
  07-AWS/
    README.md
  08-Terraform/
    README.md
  09-Ansible/
    README.md
  10-Kubernetes/
    README.md
```

## How to Use This Repository

1. Open a guide from the roadmap table.
2. Read its prerequisites before running commands.
3. Copy and practice commands in an appropriate test environment.
4. Use the troubleshooting sections when a command fails.
5. Follow the included practical projects to combine your skills.

> **Safety:** Some examples can create AWS costs, change server settings, delete data, or require elevated access. Review each guide's prerequisites and cleanup instructions before running commands.

## Future Additions

Practical end-to-end projects can be added after they are built and tested. Advanced guides such as Helm, Argo CD, or monitoring can be introduced when needed; they are not prerequisites for these ten foundational topics.

---

Maintained as a personal DevOps learning reference. Each guide is available here and linked to its separate GitHub repository above.
