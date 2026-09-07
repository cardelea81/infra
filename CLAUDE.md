# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

The `infra/` directory is a Git repository containing core infrastructure configuration files for the lab.dev.example.com environment. It manages network host mappings, repository configurations, and infrastructure reference files.

**Git Repository:** https://github.com/cardelea81/infra.git

**Parent Directory:** Part of the larger `~/lab2/infrastructure/` directory. See `/home/cardelea/lab2/infrastructure/CLAUDE.md` for the complete infrastructure automation suite.

## Directory Structure

```
infra/
├── hosts          # Primary hosts file (172.25.250.0/24 Satellite KVM network)
├── ocp            # OpenShift infrastructure hosts file (192.168.2.0/24)
├── satkvmnet      # Legacy Satellite KVM network hosts file
└── repo/          # YUM repository configuration files
    ├── rhel8.repo
    ├── rhel9.repo
    └── rhel10.repo
```

## Network Architecture

This repository manages two primary networks:

### Satellite KVM Network (172.25.250.0/24)
Defined in: `hosts` and `satkvmnet`

**Key Infrastructure Servers:**
- `172.25.250.7` - gitlab.lab.dev.example.com
- `172.25.250.8` - jenkinsmaster.lab.dev.example.com
- `172.25.250.10` - satellite.lab.dev.example.com
- `172.25.250.11` - capsule.lab.dev.example.com
- `172.25.250.17` - idm.lab.dev.example.com
- `172.25.250.30` - storage.lab.dev.example.com (FTP/NFS server, YUM repos)
- `172.25.250.31` - packstack.lab.dev.example.com

**Oracle RAC Nodes:**
- `172.25.250.67-68` - oracle-node01/02.lab.dev.example.com
- `172.25.250.69-71` - oracle-scan.lab.dev.example.com (SCAN IPs)
- `172.25.250.72-73` - oracle-node01/02-vip.lab.dev.example.com

### OpenShift Infrastructure Network (192.168.2.0/24)
Defined in: `ocp`

**Infrastructure Services:**
- `192.168.2.7` - gitlab.lab.dev.example.com
- `192.168.2.10` - satellite.lab.dev.example.com
- `192.168.2.17` - idm.lab.dev.example.com
- `192.168.2.18` - registry.lab.dev.example.com
- `192.168.2.30` - storage.lab.dev.example.com
- `192.168.2.31` - packstack.lab.dev.example.com
- `192.168.2.82` - jenkinsmaster.lab.dev.example.com

**OpenShift Cluster:**
- `192.168.2.60-62` - master-1/2/3.ocp.lab.example.com
- `192.168.2.63-65` - worker-1/2/3.ocp.lab.example.com
- `192.168.2.80` - ocp.lab.dev.example.com (cluster VIP)

## YUM Repository Configuration

The `repo/` directory contains repository files for local FTP-based YUM repositories hosted on `storage.lab.dev.example.com` (192.168.2.30).

### Using Repository Files

```bash
# Copy repository file to target system
sudo cp repo/rhel9.repo /etc/yum.repos.d/

# Verify repository accessibility
sudo yum repolist

# Install packages from local repository
sudo yum install <package-name>
```

### Repository Structure

**RHEL 9 (rhel9.repo):**
```
BaseOS:    ftp://192.168.2.30/pub/rhel9/BaseOS
AppStream: ftp://192.168.2.30/pub/rhel9/AppStream
```

**RHEL 8 (rhel8.repo):**
```
BaseOS:    ftp://192.168.2.30/pub/rhel8/BaseOS
AppStream: ftp://192.168.2.30/pub/rhel8/AppStream
```

**RHEL 10 (rhel10.repo):**
```
BaseOS:    ftp://192.168.2.30/pub/rhel10/BaseOS
AppStream: ftp://192.168.2.30/pub/rhel10/AppStream
```

### Network vs IP Differences

The two host files (`hosts` vs `ocp`) represent different network segments:
- **hosts/satkvmnet (172.25.250.0/24)**: Original Satellite KVM network, legacy infrastructure
- **ocp (192.168.2.0/24)**: Current production network for OpenShift and modern services

Some servers appear in both files with different IPs for multi-homed configurations.

## Git Operations

### Viewing Changes

```bash
cd /home/cardelea/lab2/infrastructure/infra

# Check current branch and status
git status
git branch

# View recent commits
git log --oneline -10

# View specific file history
git log --oneline hosts
git log --oneline ocp

# Compare networks
diff hosts ocp
```

### Making Changes

```bash
# Add new host entry
echo "192.168.2.XX  newhost.lab.dev.example.com" >> ocp

# Commit changes
git add ocp
git commit -m "Add newhost to infrastructure network"

# Push to remote
git push origin main
```

### Recent Changes

Based on git history:
- `e725703` - Update yum repo server IP address
- `0ca2d8f` - Update yum repo server IP address
- `93a719a` - Add ocp-network and yum repo
- `c6ea5b6` - Add old infra

## Common Use Cases

### Adding New Infrastructure Server

```bash
# 1. Add to appropriate network file
echo "192.168.2.XX  newserver.lab.dev.example.com" >> ocp

# 2. If needed on legacy network
echo "172.25.250.XX  newserver.lab.dev.example.com" >> hosts

# 3. Commit changes
git add hosts ocp
git commit -m "Add newserver infrastructure host"
git push
```

### Updating Repository Server IP

If the storage server IP changes (currently 192.168.2.30):

```bash
# Update all repository files
sed -i 's/192.168.2.30/NEW_IP/g' repo/*.repo

# Commit changes
git add repo/
git commit -m "Update YUM repository server IP to NEW_IP"
git push
```

### Deploying Repository Files to Systems

```bash
# Copy to single system
scp repo/rhel9.repo root@target.lab.dev.example.com:/etc/yum.repos.d/

# Using Ansible to deploy to multiple systems
ansible all -m copy -a "src=repo/rhel9.repo dest=/etc/yum.repos.d/rhel9.repo" -b
```

## Integration with Other Projects

### RHOSP 17 Standalone
The OpenStack standalone installation at `infrastructure/rhos/labrhos/rhosp17-standalone/` uses:
- Host: packstack.lab.dev.example.com (192.168.2.31)
- DNS: storage.lab.dev.example.com (192.168.2.30)
- Satellite: satellite.lab.dev.example.com (192.168.2.10)

### Oracle RAC 19c
The Oracle RAC automation at `oracle-projects/oraclerac19ocpv/` uses:
- Nodes: oracle-node01/02 (172.25.250.67-68)
- SCAN: oracle-scan (172.25.250.69-71)
- VIPs: oracle-node01/02-vip (172.25.250.72-73)

### Infrastructure Documentation
Server-specific documentation is maintained at:
- `infrastructure/infra-servers/` - Detailed runbooks for each infrastructure server

## File Format Notes

### hosts/ocp/satkvmnet Format
```
IP_ADDRESS  FQDN [aliases]
```

**Comments:** Lines starting with `#` are comments/section headers

**Sections in hosts file:**
- Satellite KVM Network (172.25.250.0/24)
- Satellite Network services
- CI/CD servers
- Oracle databases
- Linux cluster nodes
- OpenStack infrastructure
- Storage servers

### Repository File Format (repo/*.repo)
```ini
[RepositoryID]
name=Repository Name
baseurl=ftp://SERVER/path/to/repo
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

## Troubleshooting

### Repository Access Issues

```bash
# Test FTP connectivity
curl ftp://192.168.2.30/pub/rhel9/BaseOS/

# Test with wget
wget ftp://192.168.2.30/pub/rhel9/BaseOS/repodata/repomd.xml

# Check firewall on storage server
ssh root@192.168.2.30
firewall-cmd --list-services
# Should include ftp
```

### DNS Resolution Issues

```bash
# Verify host entries are in /etc/hosts on target system
grep "storage.lab.dev.example.com" /etc/hosts
grep "satellite.lab.dev.example.com" /etc/hosts

# Or use DNS server at 192.168.2.30
echo "nameserver 192.168.2.30" >> /etc/resolv.conf
```

### Network Connectivity

```bash
# Test connectivity to storage server
ping 192.168.2.30
ping storage.lab.dev.example.com

# Test FTP service
telnet 192.168.2.30 21

# Test from different network segment
ping 172.25.250.30  # If using legacy network
```
