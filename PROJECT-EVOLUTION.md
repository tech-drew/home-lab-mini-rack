# Version 1: Enterprise Homelab Prototype

## Overview

Version 1 was designed as an affordable, low-power prototype with the goal of creating an enterprise-inspired infrastructure environment within the practical constraints of a home lab.

Rather than purchasing enterprise hardware upfront, the objective was to build a functional infrastructure using inexpensive hardware, gain operational experience, and identify the requirements needed to design a more capable platform.

The focus of Version 1 was to understand how enterprise infrastructure is planned, deployed, secured, and operated—not simply to build the largest cluster possible.

## Objectives

The primary objectives for Version 1 were to:

* Learn Proxmox virtualization and cluster management
* Build a highly available virtualization environment
* Design an enterprise-inspired network with segmentation
* Implement centralized logging, monitoring, and alerting
* Configure shared storage and backup strategies
* Automate routine administration where appropriate
* Evaluate the capabilities and limitations of the hardware
* Define the requirements for the next iteration of the lab

## What Version 1 Achieved

The four-node cluster successfully provided an environment for learning enterprise infrastructure concepts, including:

* Proxmox cluster administration
* Hypervisor management
* High availability
* Network segmentation
* ZFS storage
* iSCSI
* SMB file services
* Remote VPN access
* Centralized logging
* NTP
* Monitoring and alerting
* Backup and recovery

The environment fulfilled its intended purpose by providing practical experience operating and maintaining a self-hosted infrastructure while remaining inexpensive to build and operate.

## Lessons Learned

Operating Version 1 clarified both the strengths of the design and the limitations imposed by the available hardware.

The cluster provided sufficient resources to explore virtualization, clustering, storage, networking, and enterprise infrastructure operations. However, expanding the environment into a platform engineering lab introduced new requirements.

Running Kubernetes on top of the virtualization platform would require significantly more compute capacity than the prototype hardware could reasonably provide. While it would have been technically possible to overcommit virtual resources, doing so would have reduced the realism and reliability of the environment.

Because the long-term goal is to emulate enterprise infrastructure within the practical limits of a home lab, Version 2 was designed to provide dedicated resources for both the virtualization layer and a highly available Kubernetes environment.

## Requirements Identified for Version 2

Experience gained from Version 1 established the design goals for the next iteration:

* Increase compute capacity to support additional workloads
* Build a six-node Proxmox cluster as the infrastructure foundation
* Deploy a highly available Kubernetes cluster with three control-plane nodes and three worker nodes
* Automate infrastructure provisioning using Terraform
* Automate configuration management using Ansible
* Reduce manual configuration through Infrastructure as Code
* Improve repeatability and consistency across the environment
* Create the infrastructure foundation for an Internal Developer Platform

## Evolution

Version 1 demonstrated how an enterprise-inspired infrastructure could be designed and operated using affordable hardware.

Version 2 builds upon those lessons by shifting the focus from infrastructure experimentation to infrastructure automation and platform engineering. The result is a repeatable, automated environment that serves as the foundation for an Internal Developer Platform, where developers can deploy and manage applications through standardized, self-service workflows.
