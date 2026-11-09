# Enterprise Linux Infrastructure Lab

### My journey towards becoming a Linux System Administrator

## Why I Started This Project

I created this project because I want to take the next step in my career and develop my skills in Linux system administration.

I'm particularly interested in working with Linux servers, infrastructure, networking and storage in an enterprise environment.

I understand that becoming a good system administrator requires more than theoretical knowledge. It requires practical experience, troubleshooting skills and an understanding of how different systems work together.

That's why I decided to build my own Linux infrastructure lab at home.

Rather than simply reading about Linux administration, I want to install, configure, troubleshoot and maintain systems myself.

My goal is to understand not only **how to perform a task, but why it works**.

## Who Is This Project For?

I'm sharing this repository with recruiters, hiring managers and technical interviewers who are interested in understanding my technical abilities and my motivation to develop as a Linux System Administrator.

In particular, I'm using this project to support my application for a **Junior Linux System Administrator position in Linköping**.

I don't expect this lab to replace professional experience. Instead, I want it to demonstrate that I'm actively investing my own time in learning the technologies required for the role.

I believe that curiosity, initiative, problem-solving and willingness to learn are important qualities in a junior engineer.

This repository is my way of demonstrating those qualities through practical work.

## What Am I Building?

I'm building a small enterprise-style Linux infrastructure environment using:

- Red Hat Enterprise Linux 10.2
- Oracle VirtualBox
- Linux networking and SSH
- LVM and XFS storage management
- NFS network file sharing
- Linux permissions and security
- firewalld and SELinux
- systemd services and automatic mounting

My environment currently consists of two RHEL servers connected through a private virtual network.

The goal is to gradually expand the environment and learn how to manage different infrastructure services.

## Projects and Progress

### Project 01 – Linux Storage Management (LVM)

Practiced managing Linux storage using Physical Volumes, Volume Groups, Logical Volumes and XFS.

Created and extended logical volumes, configured persistent mounts and verified storage after restarting the system.

### Project 02 – NFS Network Storage

Built an NFSv4.2 environment using two separate RHEL servers.

Configured network interfaces, NFS exports, firewall rules, file ownership and systemd automount.

Tested file access between both servers and verified that the configuration survived a reboot.

### Project 03 – iSCSI SAN Storage (Planned)

The next objective is to learn block-level network storage using iSCSI.

I plan to configure an iSCSI target and initiator, connect remote storage to a Linux server and manage it as a block device.

## How I'm Learning

I work through each project step by step.

My approach is to:

1. Understand the purpose of the technology.
2. Configure it in a practical lab.
3. Verify that it works.
4. Investigate problems when something fails.
5. Document the commands, configuration and lessons learned.

I also use documentation and AI-assisted learning tools to help me understand concepts and troubleshoot. My goal is to develop my own understanding rather than simply copy commands.

## My Goal

My long-term goal is to become a skilled Linux System Administrator who can confidently operate, troubleshoot and maintain Linux infrastructure.

I know I still have a lot to learn, and that's part of why I started this project.

I'm not presenting myself as an expert. I'm documenting my progress as someone who is serious about developing the skills needed for the profession.

**Thank you for taking the time to look at my work.**
