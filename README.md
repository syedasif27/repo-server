# Two-Tier Linux Repository Mirroring Architecture

**Date:** July 10, 2025  
**Author:** SYED ASIF SJ  
**Purpose:**  
This document outlines the step-by-step process for setting up a secure, two-tier Linux repository mirroring system. The goal is to provide fast, reliable, and secure package access for Rocky Linux, Oracle Linux, and Debian Linux servers in an intranet environment.

---

## Architecture Diagram

![Architecture Diagram](image.jpg)

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Setting Up Internet Zone Repo Servers](#step-1-setting-up-internet-zone-repo-servers)
    - Rocky Linux Mirror
    - Oracle Linux Mirror
    - Debian Linux Mirror
4. [Step 2: Scheduling Repository Sync in Internet Zone](#step-2-scheduling-repository-sync-in-internet-zone)
5. [Step 3: Syncing Repos to Intranet Rocky Linux Server](#step-3-syncing-repos-to-intranet-rocky-linux-server)
6. [Step 4: Serving Repos via Apache in Intranet Zone](#step-4-serving-repos-via-apache-in-intranet-zone)
7. [Step 5: Configuring Client Servers](#step-5-configuring-client-servers)
8. [Maintenance & Monitoring](#maintenance--monitoring)

---

## Architecture Overview

- **Public Internet:** Source for official Rocky, Oracle, and Debian repositories.
- **Internet Zone:**  
  - Three dedicated servers (Rocky, Oracle, Debian) each mirror their respective repositories from the public internet.
  - Each server uses the appropriate tool (`rsync`, `reposync`, `apt-mirror`).
- **Intranet Zone:**  
  - One Rocky Linux server acts as the local repo server.
  - Internal servers fetch packages from this local mirror via Apache.
  - Only SSH is allowed through the firewall for repo synchronization.

---

## Prerequisites

- SSH key-based authentication between all relevant servers.
- Sufficient disk space for all mirrored repositories.
- Apache HTTP server installed on the intranet Rocky Linux server.
- Outbound internet access for the internet zone servers.
- Firewall configured to allow only SSH from internet zone to intranet zone.

---

## Step 1: Setting Up Internet Zone Repo Servers

### **A. Rocky Linux Mirror**

1. **Install rsync:**
    ```
    sudo dnf install rsync -y
    ```
2. **Create a directory for the repo:**
    ```
    mkdir -p /srv/repo/rocky
    ```
3. **Sync from official Rocky mirror:**
    ```
    rsync -avz --delete rsync://mirror.rockylinux.org/rocky/ /srv/repo/rocky/
    ```

### **B. Oracle Linux Mirror**

1. **Install reposync:**
    ```
    sudo dnf install yum-utils -y
    ```
2. **Create a directory for the repo:**
    ```
    mkdir -p /srv/repo/oracle
    ```
3. **Sync from Oracle repo:**
    ```
    reposync --repoid=ol8_baseos_latest --download_path=/srv/repo/oracle/
    ```

### **C. Debian Linux Mirror**

1. **Install apt-mirror:**
    ```
    sudo apt-get install apt-mirror -y
    ```
2. **Edit `/etc/apt/mirror.list` to specify desired repos:**
    ```
    deb http://deb.debian.org/debian stable main contrib non-free
    ```
3. **Create a directory for the repo:**
    ```
    mkdir -p /srv/repo/debian
    ```
4. **Run apt-mirror:**
    ```
    apt-mirror
    ```

---

## Step 2: Scheduling Repository Sync in Internet Zone

Automate repo syncs with cron jobs. Example for Rocky Linux (edit with `crontab -e`):

```

0 2 * * * rsync -avz --delete rsync://mirror.rockylinux.org/rocky/ /srv/repo/rocky/
0 3 * * * reposync --repoid=ol8_baseos_latest --download_path=/srv/repo/oracle/
0 4 * * * apt-mirror

```

---

## Step 3: Syncing Repos to Intranet Rocky Linux Server

On the Intranet Rocky Linux server, create directories for each repo:

```

mkdir -p /var/www/html/repos/rocky
mkdir -p /var/www/html/repos/oracle
mkdir -p /var/www/html/repos/debian

```

Set up rsync over SSH from each internet server:

```


# Rocky Linux

rsync -avz -e ssh user@internet-rocky:/srv/repo/rocky/ /var/www/html/repos/rocky/

# Oracle Linux

rsync -avz -e ssh user@internet-oracle:/srv/repo/oracle/ /var/www/html/repos/oracle/

# Debian Linux

rsync -avz -e ssh user@internet-debian:/srv/repo/debian/ /var/www/html/repos/debian/

```

Automate with cron on the intranet Rocky Linux server:

```

30 2 * * * rsync -avz -e ssh user@internet-rocky:/srv/repo/rocky/ /var/www/html/repos/rocky/
30 3 * * * rsync -avz -e ssh user@internet-oracle:/srv/repo/oracle/ /var/www/html/repos/oracle/
30 4 * * * rsync -avz -e ssh user@internet-debian:/srv/repo/debian/ /var/www/html/repos/debian/

```

---

## Step 4: Serving Repos via Apache in Intranet Zone

1. **Install Apache:**
    ```
    sudo dnf install httpd -y
    sudo systemctl enable --now httpd
    ```
2. **Set permissions:**
    ```
    sudo chown -R apache:apache /var/www/html/repos
    sudo chmod -R 755 /var/www/html/repos
    ```
3. **Verify access:**  
   Open `http://<intranet-repo-server>/repos/` in a browser or use `curl` to check.

---

## Step 5: Configuring Client Servers

### **For Rocky/Oracle Linux (YUM/DNF):**

Create a `.repo` file (e.g., `/etc/yum.repos.d/local.repo`):

```

[local-rocky]
name=Local Rocky Repo
baseurl=http://<intranet-repo-server>/repos/rocky/
enabled=1
gpgcheck=0

```

### **For Debian Linux (APT):**

Add to `/etc/apt/sources.list`:

```

deb http://<intranet-repo-server>/repos/debian/ stable main

```

---

## Maintenance & Monitoring

- **Disk Usage:** Monitor `/srv/repo` and `/var/www/html/repos` for space.
- **Sync Logs:** Check cron logs for sync errors.
- **Apache Logs:** Monitor `/var/log/httpd/` for access and errors.
- **Test:** Regularly verify that clients can install/update packages from the local mirror.

---

**End of Documentation**
```

Let me know if you need this as a downloadable file or want any further customization!

<div style="text-align: center">⁂</div>

[^1]: image.jpg
