# Version 2: Automated Infrastructure Foundation

## Overview

Version 2 builds upon the lessons learned from Version 1 by shifting the focus from infrastructure experimentation to infrastructure automation, platform engineering, and repeatable operations.

Where Version 1 demonstrated enterprise infrastructure concepts using affordable hardware, Version 2 is designed to create an automated, enterprise-inspired Home Lab that serves as the foundation for an Internal Developer Platform (IDP).

The objective is no longer simply to deploy infrastructure manually, but to provision, configure, and manage the environment using Infrastructure as Code (IaC) and modern platform engineering practices.

> **Project Status:** Version 2 is currently under active development. Progress is primarily limited by available funding, as the target architecture requires equipment that is not yet owned.

---

# High-Level Architecture

The following diagrams illustrate the current architectural vision for Version 2. They are intended to communicate the overall design and relationships between infrastructure components.

As development continues, the architecture, hardware selection, networking, and deployment strategy may evolve based on lessons learned, funding, and changing project requirements. These diagrams should be considered **high-level conceptual designs**, not final implementation diagrams.

## Infrastructure Architecture

<img width="1024" height="559" alt="HomeLabV2" src="https://github.com/user-attachments/assets/41ecc83e-cf1b-41b2-b1bc-4c2f2d0eecd0" />

---

## Example Rack Layout

The following rack diagram illustrates the intended physical organization of the Home Lab. It represents a planning reference rather than a finalized rack layout.

<img width="974" height="1024" alt="HomeLabV2Rack" src="https://github.com/user-attachments/assets/9326d382-82f5-43f1-935b-49cb2f34ded0" />

---

# Objectives

The primary objectives for Version 2 are to:

- Build a scalable enterprise-like infrastructure platform
- Deploy a six-node Proxmox cluster as the virtualization foundation
- Provision infrastructure using Terraform
- Configure systems using Ansible
- Eliminate manual configuration wherever possible
- Build a highly available Kubernetes platform
- Develop an Internal Developer Platform (IDP)
- Standardize application deployment through GitOps workflows
- Improve observability, security, and operational consistency
- Create a repeatable environment that mirrors modern enterprise infrastructure

---

# Planned Architecture

Version 2 expands the Home Lab into multiple infrastructure layers.

## Infrastructure Layer

- Six-node Proxmox cluster
- High availability
- Shared storage
- Enterprise-like networking
- Infrastructure monitoring
- Centralized logging
- Automated backups

## Automation Layer

Infrastructure provisioning and lifecycle management will be fully automated using:

- Terraform
- Ansible
- Git-based Infrastructure as Code

Manual configuration will be minimized to improve consistency, repeatability, and disaster recovery.

## Platform Layer

A dedicated Kubernetes environment will provide the foundation for platform engineering.

Planned capabilities include:

- Highly available Kubernetes control plane
- Dedicated worker nodes
- GitOps deployments
- Secrets management
- Ingress management
- Certificate automation
- Platform monitoring
- Application observability

---

# Design Goals

Version 2 is designed around several key principles:

- Automation first
- Infrastructure as Code
- Repeatability
- High availability
- Security by default
- Enterprise-like operational practices
- Platform engineering over manual administration

---

# Expected Outcomes

Upon completion, Version 2 will provide:

- A fully automated enterprise-like Home Lab
- Repeatable infrastructure deployments
- Automated configuration management
- A production-style Kubernetes platform
- A foundation for an Internal Developer Platform
- A realistic environment for learning modern infrastructure engineering practices

---

# Relationship to Version 1

Version 1 established the infrastructure foundation and identified the limitations of running increasingly complex workloads on low-power hardware.

Version 2 addresses those limitations by introducing additional compute capacity, enterprise-like automation, Infrastructure as Code, and a dedicated Kubernetes platform. Rather than focusing on learning individual technologies, Version 2 focuses on integrating those technologies into a cohesive, automated platform that reflects modern enterprise operations.

Together, the two versions document the evolution of the Home Lab from a learning-focused infrastructure prototype to an automated platform engineering environment.
