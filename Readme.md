# Jenkins on Docker – CI/CD Setup & Recovery Story

## Overview

This repository documents the **complete setup of Jenkins on Docker**, including:

- Installing Jenkins in a Docker container  
- Setting up persistent storage for jobs and configuration  
- Installing suggested plugins  
- Handling admin password issues  
- Lessons learned from data loss and recovery  

This is a beginner-friendly guide for developers and DevOps enthusiasts exploring **CI/CD with Jenkins and Docker**.

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Pull Jenkins Docker Image](#pull-jenkins-docker-image)  
3. [Run Jenkins Container](#run-jenkins-container)  
4. [Access Jenkins](#access-jenkins)  
5. [Handle Admin Password Issues](#handle-admin-password-issues)  
6. [Persistent Volume Setup](#persistent-volume-setup)  
7. [Data Loss Fiasco & Recovery](#data-loss-fiasco--recovery)  
8. [Post-Recovery Actions](#post-recovery-actions)  
9. [Daily Login Steps](#daily-login-steps)  
10. [Lessons Learned](#lessons-learned)  
11. [Screenshots](#screenshots)  
12. [References](#references)  

---

## Prerequisites

- Docker installed and running  
- Basic command-line knowledge  
- Internet connection for pulling Docker images and installing plugins  

---

## Pull Jenkins Docker Image

Pull the latest LTS Jenkins image:

```bash
docker pull jenkins/jenkins:lts-jdk21
