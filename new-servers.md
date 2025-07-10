
# Onboarding New Intranet Servers to Use the Intranet Repository Server

**Date:** July 10, 2025
**Author:** SYED ASIF
**Purpose:**
This document provides step-by-step instructions and an automated script to configure new Linux servers in your intranet to use your internal repository mirror for Rocky Linux, Oracle Linux, Debian, or Ubuntu.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Automated Setup Script](#automated-setup-script)
4. [Manual Configuration Steps](#manual-configuration-steps)
    - [Rocky/Oracle Linux](#rockyoracle-linux)
    - [Debian/Ubuntu](#debianubuntu)
5. [Usage Instructions](#usage-instructions)
6. [Verification](#verification)
7. [Troubleshooting](#troubleshooting)
8. [Appendix: Script Source](#appendix-script-source)

## Overview

To ensure fast, reliable, and secure package management, all new servers in the intranet should use the internal repository server instead of public mirrors. This document provides an automated script and manual steps for configuring supported Linux distributions to use the intranet repo.

## Prerequisites

- **Network:** The new server can reach the intranet repo server via HTTP.
- **Permissions:** You have root or sudo privileges.
- **Information Needed:** The intranet repo server’s hostname or IP address.


## Automated Setup Script

Below is the recommended shell script to automate repo configuration.
**Before use:**

- Replace `INTRANET_REPO_SERVER` with your actual repo server’s hostname or IP.

```bash
#!/bin/bash

# Set your intranet repo server address here
INTRANET_REPO_SERVER="intranet-repo-server"  # <-- CHANGE THIS

# Detect OS
if [ -f /etc/os-release ]; then
    . /etc/os-release
    OS_ID=$ID
else
    echo "Cannot detect OS."
    exit 1
fi

echo "Detected OS: $OS_ID"

case "$OS_ID" in
    rocky|centos|rhel|ol|oracle)
        # For Rocky Linux, Oracle Linux, CentOS, RHEL
        cat > /etc/yum.repos.d/intranet-local.repo <<EOF
[local-intranet]
name=Intranet Local Repo
baseurl=http://$INTRANET_REPO_SERVER/repos/rocky/
enabled=1
gpgcheck=0
EOF
        echo "Configured YUM/DNF to use intranet Rocky repo."
        ;;

    debian|ubuntu)
        # Backup existing sources.list
        cp /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%F-%H%M%S)
        # Add intranet repo (adjust 'stable main' as needed)
        echo "deb http://$INTRANET_REPO_SERVER/repos/debian/ stable main" > /etc/apt/sources.list.d/intranet-local.list
        echo "Configured APT to use intranet Debian repo."
        ;;

    *)
        echo "Unsupported OS: $OS_ID"
        exit 2
        ;;
esac

# Optionally update package cache
if command -v dnf >/dev/null 2>&1; then
    dnf clean all
    dnf makecache
elif command -v yum >/dev/null 2>&1; then
    yum clean all
    yum makecache
elif command -v apt-get >/dev/null 2>&1; then
    apt-get update
fi

echo "Repository configuration complete."
```


## Manual Configuration Steps

If you prefer to configure manually, follow the steps below for your OS.

### Rocky/Oracle Linux

1. **Create `/etc/yum.repos.d/intranet-local.repo`:**

```ini
[local-intranet]
name=Intranet Local Repo
baseurl=http://<INTRANET_REPO_SERVER>/repos/rocky/
enabled=1
gpgcheck=0
```

2. **Clean and refresh cache:**

```bash
sudo dnf clean all && sudo dnf makecache
# or
sudo yum clean all && sudo yum makecache
```


### Debian/Ubuntu

1. **Backup existing sources list:**

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%F-%H%M%S)
```

2. **Create `/etc/apt/sources.list.d/intranet-local.list`:**

```
deb http://<INTRANET_REPO_SERVER>/repos/debian/ stable main
```

3. **Update APT cache:**

```bash
sudo apt-get update
```


## Usage Instructions

1. **Download the script:**
    - Save the script above as `setup-intranet-repo.sh`.
2. **Edit the script:**
    - Set the correct value for `INTRANET_REPO_SERVER`.
3. **Run the script as root:**

```bash
sudo bash setup-intranet-repo.sh
```


## Verification

After running the script:

- **Rocky/Oracle:**

```bash
sudo dnf repolist
# or
sudo yum repolist
```

You should see `local-intranet` in the list.
- **Debian/Ubuntu:**

```bash
sudo apt-get update
sudo apt-cache policy
```

You should see your intranet repo listed.


## Troubleshooting

- **Cannot reach repo server:**
    - Check network connectivity and firewall rules.
    - Try: `curl http://<INTRANET_REPO_SERVER>/repos/rocky/` or `/repos/debian/`
- **Packages not found:**
    - Verify the correct repo paths and that the repo server is up-to-date.
- **OS not detected:**
    - Ensure `/etc/os-release` exists and is readable.


## Appendix: Script Source

```bash
#!/bin/bash

# Set your intranet repo server address here
INTRANET_REPO_SERVER="intranet-repo-server"  # <-- CHANGE THIS

# Detect OS
if [ -f /etc/os-release ]; then
    . /etc/os-release
    OS_ID=$ID
else
    echo "Cannot detect OS."
    exit 1
fi

echo "Detected OS: $OS_ID"

case "$OS_ID" in
    rocky|centos|rhel|ol|oracle)
        # For Rocky Linux, Oracle Linux, CentOS, RHEL
        cat > /etc/yum.repos.d/intranet-local.repo <<EOF
[local-intranet]
name=Intranet Local Repo
baseurl=http://$INTRANET_REPO_SERVER/repos/rocky/
enabled=1
gpgcheck=0
EOF
        echo "Configured YUM/DNF to use intranet Rocky repo."
        ;;

    debian|ubuntu)
        # Backup existing sources.list
        cp /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%F-%H%M%S)
        # Add intranet repo (adjust 'stable main' as needed)
        echo "deb http://$INTRANET_REPO_SERVER/repos/debian/ stable main" > /etc/apt/sources.list.d/intranet-local.list
        echo "Configured APT to use intranet Debian repo."
        ;;

    *)
        echo "Unsupported OS: $OS_ID"
        exit 2
        ;;
esac

# Optionally update package cache
if command -v dnf >/dev/null 2>&1; then
    dnf clean all
    dnf makecache
elif command -v yum >/dev/null 2>&1; then
    yum clean all
    yum makecache
elif command -v apt-get >/dev/null 2>&1; then
    apt-get update
fi

echo "Repository configuration complete."
```
