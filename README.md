# Azure-Cost-Optimization---HA-Web-Infrastructure
Azure Cost Optimization - HA Web Infrastructure

Overview

This project documents a cost analysis and remediation effort performed on an Azure-hosted high-availability web infrastructure. The audit identified and eliminated unnecessary billable resources that accumulated during active development, resulting in significant monthly cost savings.

Project Context

The ha-web-infrastructure-rg resource group hosted a high-availability web application built with:

- Two Linux virtual machines across availability zones
- An Azure Load Balancer
- An AKS (Azure Kubernetes Service) cluster
- A NAT Gateway
- Public IP addresses attached to VM NICs
- A Virtual Network with subnets

During active development, several resources continued running and billing after their immediate need passed. This audit targeted those resources for cleanup.

Problem Statement

Azure charges for the following resources even when idle or unused:

- Running virtual machines (compute + OS disk)
- AKS node pool VMs
- NAT Gateway (hourly + data processing fees)
- Public IP addresses (dynamic and static, whether attached or not)
- Load Balancer rules

Left unaddressed, these resources accumulate hundreds of dollars in monthly charges with zero business value during non-production periods.

Cost Analysis

| Resource | Type | Estimated Monthly Cost |
|---|---|---|
| AKS Cluster (1 node pool) | Compute | ~$140-180 |
| NAT Gateway | Networking | ~$32-45 |
| vm-web-01 | Standard B-series VM | ~$30-60 |
| vm-web-02 | Standard B-series VM | ~$30-60 |
| Public IP - vm-web-01 | Static Public IP | ~$3.65 |
| Public IP - vm-web-02 | Static Public IP | ~$3.65 |
| Load Balancer | Standard tier | ~$18-22 |
| Total (estimated) | | ~$257-374/month |

Remediation Steps

Step 1 - Audit Existing Public IPs

Listed all public IPs in the resource group to identify orphaned or unnecessary addresses:

``bash
az network public-ip list --resource-group ha-web-infrastructure-rg --output table
`

Step 2 - Stop the AKS Cluster

Stopped the AKS cluster to eliminate node pool compute charges during non-production periods:

`bash
az aks stop --resource-group ha-web-infrastructure-rg --name <aks-cluster-name>
`

Step 3 - Delete the NAT Gateway

Removed the NAT Gateway, which billed hourly regardless of traffic volume:

`bash
az network nat gateway delete --resource-group ha-web-infrastructure-rg --name <nat-gateway-name>
`

Step 4 - Detach Public IPs from VM NICs

Removed public IP associations from both VM network interface configurations:

VM Web 01:
`bash
az network nic ip-config update \
 --resource-group ha-web-infrastructure-rg \
 --nic-name vm-web-01<nic-suffix> \
 --name ipconfig1 \
 --remove publicIpAddress
`

VM Web 02:
`bash
az network nic ip-config update \
 --resource-group ha-web-infrastructure-rg \
 --nic-name vm-web-02596_z2 \
 --name ipconfig1 \
 --remove publicIpAddress
`

Step 5 - Delete Orphaned Public IP Resources

Deleted the standalone public IP resources after detaching them from NICs:

`bash
az network public-ip delete --resource-group ha-web-infrastructure-rg --name vmweb01ip<suffix>

az network public-ip delete --resource-group ha-web-infrastructure-rg --name vmweb02ip702
`

Step 6 - Verify Cleanup

Confirmed zero public IPs remain in the resource group:

`bash
az network public-ip list --resource-group ha-web-infrastructure-rg --output table
`

Results

- All orphaned public IP resources removed from the resource group
- AKS cluster stopped, eliminating node pool compute billing
- NAT Gateway deleted, eliminating hourly networking charges
- VM NICs cleaned up with no public exposure
- Infrastructure remains intact and ready for redeployment when needed

Screenshots

Before Cleanup
| Screenshot | File |
|---|---|
| Public IPs listed in resource group | screenshots/before/01-public-ips-before-cleanup.png |
| AKS cluster in Running state | screenshots/before/02-aks-cluster-running.png |
| NAT Gateway present in resource group | screenshots/before/03-nat-gateway-present.png |
| Cost Analysis dashboard | screenshots/before/04-cost-analysis-dashboard.png |

During Cleanup
| Screenshot | File |
|---|---|
| Public IP audit terminal output | screenshots/during/05-public-ip-audit-output.png |
| NIC detach command - VM Web 01 | screenshots/during/06-nic-detach-vm-web-01.png |
| NIC detach command - VM Web 02 | screenshots/during/07-nic-detach-vm-web-02.png |
| Public IP delete commands output | screenshots/during/08-public-ip-delete-commands.png |

After Cleanup
| Screenshot | File |
|---|---|
| Public IP list showing empty results | screenshots/after/09-public-ip-list-empty.png |
| AKS cluster in Stopped state | screenshots/after/10-aks-cluster-stopped.png |
| Resource group with cleanup confirmed | screenshots/after/11-resource-group-cleaned.png |

Key Takeaways

- Azure bills for public IPs even when detached from resources - always delete them explicitly after removing the association
- AKS node pools continue billing until the cluster reaches a stopped state - stopping the cluster fully is the correct action
- NAT Gateways carry a flat hourly charge regardless of usage volume - remove them during non-production periods
- Regular resource audits using az CLI list commands catch billing leaks before they compound

Tools Used

- Azure CLI (az)
- Azure Portal (verification)
- Resource Group: ha-web-infrastructure-rg`

Author

Maurrin Carter
Cloud Infrastructure Engineer
GitHub: [your-github-handle]

Related Projects

- HA Web Infrastructure (link-to-repo)
- Azure Monitoring & Alerting (coming Monday)
