# Project 11 - Ansible Configuration Management

## Overview

This project demonstrates centralized configuration management and automation using Ansible.

The environment consists of an Ansible controller managing multiple Linux servers through SSH.

## Infrastructure

The managed infrastructure contains:

- Ansible Controller
- Ubuntu Load Balancer
- RHEL Web Server 1
- RHEL Web Server 2
- RHEL NFS Server
- RHEL Database Server

## Ansible Architecture

The Ansible controller connects to the managed nodes using SSH and executes configuration tasks through Ansible playbooks.

Inventory groups:

- webservers
- nfs
- db
- lb

## Configuration Management

The project uses an Ansible inventory to define managed hosts and their connection parameters.

The common playbook performs configuration tasks on the RHEL servers and configures the Ubuntu load balancer.

## Playbook

The primary playbook is:

`playbooks/common.yml`

The playbook includes:

- Gathering system facts
- Installing Wireshark on RHEL servers
- Installing Wireshark on the Ubuntu load balancer
- Automated configuration through Ansible

## Connectivity Verification

Ansible connectivity was verified using:

```bash
ansible all -i inventory/dev.yml -m ping

## CI/CD Validation

GitHub webhook integration with Jenkins successfully configured and tested.