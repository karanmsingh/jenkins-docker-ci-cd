# Jenkins on Docker – CI/CD Setup \& Recovery Story

## Overview

This repository documents the **complete setup of Jenkins on Docker**, including:

* Installing Jenkins in a Docker container
* Setting up persistent storage for jobs and configuration
* Installing suggested plugins
* Handling admin password issues
* Lessons learned from data loss and recovery

This is beginner-friendly for developers and DevOps enthusiasts exploring **CI/CD with Jenkins and Docker**.

\---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Pull Jenkins Docker Image](#pull-jenkins-docker-image)
3. [Run Jenkins Container](#run-jenkins-container)
4. [Access Jenkins](#access-jenkins)
5. [Handle Admin Password Issues](#handle-admin-password-issues)
6. [Persistent Volume Setup](#persistent-volume-setup)
7. [Data Loss Fiasco \& Recovery](#data-loss-fiasco--recovery)
8. [Post-Recovery Actions](#post-recovery-actions)
9. [Daily Login Steps](#daily-login-steps)
10. [Lessons Learned](#lessons-learned)
11. [References](#references)

\---

## Prerequisites

* Docker installed and running
* Basic command-line knowledge
* Internet connection for pulling Docker images and installing plugins

\---

## Pull Jenkins Docker Image

Pull the latest LTS Jenkins image:

```bash
docker pull jenkins/jenkins:lts-jdk21
```

\---

## Run Jenkins Container

### Option 1: Temporary container

```bash
docker run -d --name jenkins\_temp -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts-jdk21
```

* Runs Jenkins in **detached mode** (`-d`)
* Exposes **8080** (web UI) and **50000** (agent connections)

### Option 2: Persistent container (Recommended)

```bash
docker volume create jenkins\_home

docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins\_home:/var/jenkins\_home jenkins/jenkins:lts-jdk21
```

* Creates a **named volume** `jenkins\_home` to persist jobs, plugins, and configurations

\---

## Access Jenkins

1. Get the initial admin password:

```bash
docker exec jenkins cat /var/jenkins\_home/secrets/initialAdminPassword
```

2. Open browser: `http://localhost:8080`
3. Unlock Jenkins using the password
4. Install **suggested plugins**

\---

## Handle Admin Password Issues

If the password does not work:

1. Create a Groovy init script to reset the admin:

```bash
cat << 'EOF' > /var/jenkins\_home/init.groovy.d/basic-security.groovy
#!groovy
import jenkins.model.\*
import hudson.security.\*

def instance = Jenkins.getInstance()
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "admin123")
instance.setSecurityRealm(hudsonRealm)

def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
instance.setAuthorizationStrategy(strategy)
strategy.setAllowAnonymousRead(false)

instance.save()
EOF
```

2. Restart Jenkins:

```bash
docker restart jenkins
```

3. Log in using:
* **Username:** admin
* **Password:** admin123
4. Remove the script after login:

```bash
docker exec -it jenkins rm /var/jenkins\_home/init.groovy.d/basic-security.groovy
```

\---

## Persistent Volume Setup

* Named volume ensures **data persistence** across container restarts and removals:

```bash
docker volume create jenkins\_home
```

* Run Jenkins with:

```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins\_home:/var/jenkins\_home jenkins/jenkins:lts-jdk21
```

* Your jobs, plugins, and configs are **safe even if the container is deleted**

\---

## Data Loss Incident \& Recovery

* Multiple containers were created accidentally
* Old jobs disappeared after removing containers or pruning
* Verified Jenkins home:

```bash
docker exec -it jenkins ls /var/jenkins\_home
```

* Recovery: Mounting a persistent volume restored previous jobs automatically

> \*\*Lesson:\*\* Always use named volumes; containers are ephemeral by default.

\---

## Post-Recovery Actions

* Remove Groovy script to prevent future overwrites
* Secure Jenkins with a proper admin user
* Install plugins as required
* Set up pipelines and job configurations

\---

## Daily Login Steps

1. Ensure Docker is running
2. Start Jenkins if stopped:

```bash
docker start jenkins
```

3. Access Jenkins at `http://localhost:8080`
4. Log in with your admin credentials
5. Verify jobs and plugins

> \*\*Tip:\*\* No need to retrieve the initial admin password again  

\---

## Lessons Learned

* Initial Admin Password only works on first setup
* Docker containers are ephemeral — deleting a container deletes its data unless a volume is used
* Persistent volumes are essential for production-like Jenkins setups
* Groovy scripts in `init.groovy.d` allow safe admin recovery
* Naming containers and volumes reduces confusion
* Always back up your Docker volumes periodically

\---

## References

* [Official Jenkins Docker Documentation](https://www.jenkins.io/doc/book/installing/docker/)
* [Docker Volumes](https://docs.docker.com/engine/storage/volumes/)
* [Jenkins Init Groovy Scripts](https://www.jenkins.io/doc/pipeline/steps/groovy/)

