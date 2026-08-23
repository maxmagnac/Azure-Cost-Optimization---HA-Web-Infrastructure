# Azure Cost Optimization: HA Web Infrastructure

Project Overview

This project documents a cost optimization initiative for a high-availability web infrastructure hosted on Azure. The infrastructure runs inside the ha-web-infrastructure-rg resource group and includes virtual machines, a virtual network with public and private subnets, an AKS cluster, and supporting network resources.

The goal centers on identifying unnecessary costs within the resource group and eliminating them without disrupting the core infrastructure design.

Problem Statement

Azure Cost Management flagged rising charges tied to resources that ran continuously without active use. A NAT Gateway operated around the clock, generating hourly charges regardless of traffic volume. Multiple public IP addresses sat allocated to resources, some idle, each accruing a standard hourly rate. The AKS cluster ran at full capacity during periods when the workload didn't require it.

Together, these resources drove up the monthly bill well beyond what the actual usage justified.

Cost Analysis

| Resource | Type | Status Before | Estimated Monthly Cost |
|---|---|---|---|
| ha-web-nat-gateway | NAT Gateway | Running 24/7 | ~$32/month |
| Public IP addresses | Standard SKU | Multiple allocated | ~$3-4/month each |
| AKS cluster | Kubernetes cluster | Running continuously | Variable, node-pool dependent |
| Virtual machines | vm-web-01, vm-web-02, vm-db-01 | Running | Variable |

The Cost Analysis dashboard in Azure Portal confirmed these charges accumulating daily, with the NAT Gateway and public IPs representing steady, avoidable spend.

Remediation Steps

1. Audited public IP addresses - Ran az network public-ip list --resource-group ha-web-infrastructure-rg -o table to identify every public IP tied to the resource group, including ones attached to kubernetes, ha-load-balancer, and individual VMs.
2. Reviewed the NAT Gateway configuration - Ran az network nat gateway list --resource-group ha-web-infrastructure-rg -o table to confirm the gateway's status, location, and idle timeout settings before removal.
3. Verified subnet dependencies - Checked az network vnet subnet list output to confirm which subnets relied on the NAT Gateway for outbound access before deleting it.
4. Stopped the AKS cluster - Deallocated the cluster through the Azure Portal to eliminate compute charges during idle periods.
5. Deleted the NAT Gateway - Removed ha-web-nat-gateway after confirming no active dependencies required it.
6. Removed unused public IP addresses - Deleted the IPs no longer tied to active services.
7. Re-ran the public IP audit - Confirmed the resource group returned an empty result, verifying complete cleanup.

Results

The final audit confirmed the NAT Gateway and all flagged public IP addresses no longer exist in ha-web-infrastructure-rg. The resource group overview page shows only the VMs, virtual network, logs, and alert rules that support the active infrastructure. The AKS cluster sits in a stopped state, ready to restart when the workload calls for it.

This cleanup removes recurring charges tied to idle networking resources and continuous cluster uptime, translating to a measurable reduction in the monthly Azure bill.

Lessons Learned

Public IP addresses and NAT Gateways carry hourly charges independent of actual traffic, making regular audits essential for cost control. Stopping an AKS cluster during idle periods preserves the cluster configuration while eliminating compute costs. The Azure CLI provides fast, repeatable verification steps that complement the Portal's visual confirmation, giving stronger proof of both the problem and the fix.

Screenshots

Before Cleanup
- screenshots/before/01-public-ips-before-cleanup.png - All public IPs listed in ha-web-infrastructure-rg prior to removal.
- screenshots/before/03-nat-gateway-cli-output.png - CLI output confirming the NAT Gateway's active state.
- screenshots/before/04-cost-analysis-dashboard.png - Cost Analysis dashboard showing estimated charges.
- screenshots/before/vm-status-before-cleanup.png - Virtual machine status before any changes.
- screenshots/before/vnet-subnet-configuration.png - VNet and subnet configuration showing outbound access settings.

During Remediation
- screenshots/during/10-aks-cluster-stopped.png - AKS cluster shown in a Stopped state.

After Cleanup
- screenshots/after/09-public-ip-list-empty.png - CLI output confirming zero public IPs remain.
- screenshots/after/11-resource-group-cleaned.png - Resource group overview showing the NAT Gateway and public IPs removed.
`
