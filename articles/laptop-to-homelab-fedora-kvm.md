# How I Turned My HP Laptop into a Fedora Workstation and KVM Home Lab

A laptop can be more than a daily workstation. I turned my HP Envy x360 into a local test environment by installing Fedora Workstation and setting up KVM virtualization.

As an infrastructure and DevOps engineer, I wanted a practical space to experiment, build virtual machines, and test configurations. This article documents my setup and the lessons learned along the way.

## My Hardware and Operating System

| Component | Configuration |
| --- | --- |
| Laptop | HP Envy x360 2-in-1, 16-ac0xxx |
| Processor | Intel Core Ultra 7 155U |
| CPU resources | 12 cores and 14 logical processors |
| Memory | Approximately 32 GB RAM |
| Storage | Approximately 2 TB NVMe |
| Operating system | Fedora Linux 44 Workstation Edition |
| Virtualization support | Intel VT-x |
| Disk encryption | LUKS-encrypted root and home storage |
| Swap | 8 GiB zram |

This machine serves as both my workstation and the host for my virtual lab.

## Checking the Host

Before describing the virtualization setup, here are the commands I used to inspect the machine:

    hostnamectl
    free -h
    lsblk
    lscpu

These show the operating system, available memory, storage layout, and CPU capabilities. My CPU output reports Intel VT-x virtualization support.
