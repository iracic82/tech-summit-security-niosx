# Tech Summit Security Lab — NIOS-X Infrastructure

Terraform and Python scripts for the **Tech Summit Security** Instruqt lab. Deploys a comprehensive security-focused environment in AWS with Infoblox NIOS Grid Master, NIOS-X servers, Windows Clients, and Ubuntu workstations.

## Architecture

```
VPC: 10.100.0.0/16 (eu-central-1)
├── Public Subnet:   10.100.0.0/24 (eu-central-1a)
└── Public Subnet B: 10.100.1.0/24 (eu-central-1b)

10.100.0.10   →  NIOS Grid Master (MGMT)  (m5.2xlarge, AMI-based)
10.100.0.11   →  NIOS Grid Master (LAN1)
10.100.0.110  →  Windows Client           (t3.medium, Windows Server 2022)
10.100.0.120  →  Windows Client 2         (t3.medium, Windows Server 2022)
10.100.0.130  →  Ubuntu Workstation       (t3.small, Ubuntu 22.04)
10.100.0.140  →  Ubuntu Syslog Server     (t3.small, Ubuntu 22.04, TCP/UDP 514)
10.100.0.200  →  NIOS-X Server #1         (m5.2xlarge, AMI-based)
10.100.1.200  →  NIOS-X Server #2         (m5.2xlarge, AMI-based)
```

## Components

| Component | Description |
|-----------|-------------|
| **NIOS Grid Master** | Traditional Infoblox appliance with dual NICs (MGMT + LAN1) |
| **NIOS-X Servers** | Cloud-native DNS services joined to Infoblox CSP |
| **Windows Clients** | Management workstations with RDP access |
| **Ubuntu Workstation** | CLI-based operations via SSH |
| **Ubuntu Syslog** | Centralized logging server with rsyslog (TCP/UDP 514) |

## Repository Structure

```
terraform/
├── providers.tf         # AWS provider
├── variables.tf         # Region, VPC CIDR, admin password, join tokens
├── main.tf              # VPC, subnets, IGW, SGs, GM, Windows Clients, Ubuntu VMs
├── niosx.tf             # 2x NIOS-X EC2 instances with cloud-init join tokens
├── outputs.tf           # Public IPs and SSH commands
└── scripts/
    ├── sandbox_api.py                 # CSP sandbox API client
    ├── create_sandbox.py              # Create Infoblox CSP sandbox
    ├── create_user.py                 # Create CSP user
    ├── deploy_api_key.py              # Generate and export API key
    ├── infoblox_create_join_token.py  # Generate NIOS-X join token
    ├── delete_sandbox.py              # Delete CSP sandbox
    ├── delete_user.py                 # Delete CSP user
    ├── setup_dns.py                   # Create DNS A records (Windows Clients, GM)
    ├── cleanup_dns_records.py         # Delete DNS records
    ├── create_dns_niosx.py            # Create DNS A records for NIOS-X servers
    ├── clean_dns_niosx.py             # Delete NIOS-X DNS records
    └── winrm-init.ps1.tpl             # Windows user_data (WinRM + RDP setup)
```

## DNS Records Created

| FQDN | Target |
|------|--------|
| `{participant_id}-client.iracictechguru.com` | Windows Client IP |
| `{participant_id}-client2.iracictechguru.com` | Windows Client 2 IP |
| `{participant_id}-infoblox.iracictechguru.com` | NIOS Grid Master IP |
| `{participant_id}-niosx1.iracictechguru.com` | NIOS-X Server #1 IP |
| `{participant_id}-niosx2.iracictechguru.com` | NIOS-X Server #2 IP |

## Required Variables

| Variable | Description |
|----------|-------------|
| `windows_admin_password` | Windows Administrator password (sensitive) |
| `infoblox_join_token` | CSP join token for NIOS-X servers (sensitive) |

## Required Environment Variables (for Python scripts)

| Variable | Description |
|----------|-------------|
| `Infoblox_Token` | CSP API token |
| `INFOBLOX_EMAIL` | CSP login email |
| `INFOBLOX_PASSWORD` | CSP login password |
| `INSTRUQT_PARTICIPANT_ID` | Instruqt participant ID |
| `INSTRUQT_EMAIL` | Participant email |
| `DEMO_AWS_ACCESS_KEY_ID` | AWS key for Route 53 DNS management |
| `DEMO_AWS_SECRET_ACCESS_KEY` | AWS secret for Route 53 DNS management |
| `DEMO_HOSTED_ZONE_ID` | Route 53 hosted zone ID |
| `DC1_IP` | Windows Client public IP |
| `CLIENT_2_IP` | Windows Client 2 public IP |
| `GM_IP` | NIOS Grid Master public IP |

## Usage

```bash
cd terraform/

# Run Python scripts first to create sandbox, user, API key, and join tokens
cd scripts/
python3 create_sandbox.py
python3 create_user.py
python3 deploy_api_key.py
python3 infoblox_create_join_token.py
source ~/.bashrc
cd ..

# Deploy infrastructure
terraform init
terraform apply -auto-approve

# Create DNS records
cd scripts/
python3 setup_dns.py
python3 create_dns_niosx.py
```

## Access

| Resource | URL/Command | Credentials |
|----------|-------------|-------------|
| NIOS Grid Master UI | `https://{participant_id}-infoblox.iracictechguru.com` | admin / Inf0blox2025! |
| Windows Client RDP | Via Guacamole | Administrator / Inf0blox2025! |
| Ubuntu SSH | `ssh -i instruqt-dc-key.pem ubuntu@{ip}` | Key-based |
| Infoblox Portal | https://csp.infoblox.com | CSP credentials |
