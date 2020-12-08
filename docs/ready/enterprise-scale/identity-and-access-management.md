---
title: Identity and access management
description: Examine design considerations and recommendations related to identity and access management in an enterprise environment.
author: BrianBlanchard
ms.author: brblanch
ms.date: 06/15/2020
ms.topic: conceptual
ms.service: cloud-adoption-framework
ms.subservice: ready
ms.custom: think-tank
---

# Identity and access management

Identity provides the basis of a large percentage of security assurance. It enables access based on identity authentication and authorization controls in cloud services to protect data and resources and to decide which requests should be permitted.

Identity and access management (IAM) is boundary security in the public cloud. It must be treated as the foundation of any secure and fully compliant public cloud architecture. Azure offers a comprehensive set of services, tools, and reference architectures to enable organizations to make highly secure, operationally efficient environments as outlined here.

This section examines design considerations and recommendations related to IAM in an enterprise environment.

## Why we need identity and access management

The technological landscape in the enterprise is becoming complex and heterogenous. To manage compliance and security for this environment, IAM enables the right individuals to access the right resources at the right time for the right reasons.

### Plan for identity and access management

Enterprise organizations typically follow a least-privileged approach to operational access. This model should be expanded to consider Azure through Azure Active Directory (Azure AD) role-based access control (RBAC) and custom role definitions. It's critical to plan how to govern control- and data-plane access to resources in Azure. Any design for IAM and RBAC must meet regulatory, security, and operational requirements before it can be accepted.

Identity and access management is a multistep process that involves careful planning for identity integration and other security considerations, such as blocking legacy authentication and planning for modern passwords. Staging planning also involves selection of business-to-business or business-to-consumer identity and access management. While these requirements vary, there are common design considerations and recommendations to consider for an enterprise landing zone.

![Diagram that shows identity and access management.](./media/iam.png)

_Figure 1: Identity and access management._

**Design considerations:**

- There are limits around the number of custom roles and role assignments that must be considered when you lay down a framework around IAM and governance. For more information, see [Azure RBAC service limits](/azure/azure-resource-manager/management/azure-subscription-service-limits#role-based-access-control-limits).
- There's a limit of 2,000 custom RBAC role assignments per subscription.
- There's a limit of 500 custom RBAC role assignments per management group.
- Centralized versus federated resource ownership:
  - Shared resources or any aspect of the environment that implements or enforces a security boundary, such as the network, must be managed centrally. This requirement is part of many regulatory frameworks. It's standard practice for any organization that grants or denies access to confidential or critical business resources.
  - Managing application resources that don't violate security boundaries or other aspects required to maintain security and compliance can be delegated to application teams. Allowing users to provision resources within a securely managed environment allows organizations to take advantage of the agile nature of the cloud while preventing the violation of any critical security or governance boundary.

<!-- docutune:ignore Azure-AD-only Azure-AD-managed -->

**Design recommendations:**

- Use Azure AD [RBAC](/azure/role-based-access-control/overview) to manage data-plane access to resources, where possible. Examples are Azure Key Vault, a storage account, or a SQL database.
- Deploy Azure AD conditional-access policies for any user with rights to Azure environments. Doing so provides another mechanism to help protect a controlled Azure environment from unauthorized access.
- Enforce multi-factor authentication for any user with rights to the Azure environments. Multi-factor authentication enforcement is a requirement of many compliance frameworks. It greatly lowers the risk of credential theft and unauthorized access.
- Use [Azure AD Privileged Identity Management (PIM)](/azure/active-directory/privileged-identity-management/pim-configure) to establish zero standing access and least privilege. Map your organization's roles to the minimum level of access needed. Azure AD PIM can either be an extension of existing tools and processes, use Azure native tools as outlined, or use both as needed.
- Use Azure-AD-only groups for Azure control-plane resources in Azure AD PIM when you grant access to resources.
  - Add on-premises groups to the Azure-AD-only group if a group management system is already in place.
- Use Azure AD PIM access reviews to periodically validate resource entitlements. Access reviews are part of many compliance frameworks. As a result, many organizations will already have a process in place to address this requirement.
- Integrate Azure AD logs with the platform-central [Azure Monitor](/azure/active-directory/reports-monitoring/concept-activity-logs-azure-monitor). Azure Monitor allows for a single source of truth around log and monitoring data in Azure, which gives organizations cloud-native options to meet requirements around log collection and retention.
- If any data sovereignty requirements exist, custom user policies can be deployed to enforce them.
- Use custom RBAC role definitions within the Azure AD tenant while you consider the following key roles:

| Role | Usage | Actions | No actions |
|---|---|---|---|
| Azure platform owner (such as the built-in Owner role)               | Management group and subscription lifecycle management                                                           | `build-in owner role`                                                                                                                                                                                                                  |                                                                                                                                                                                         |
| Network management (such as the built-in Network Contributor)         | Platform-wide global connectivity management: virtual networks, UDRs, NSGs, NVAs, VPN, Azure ExpressRoute, and others            | `build-in Network contributor`                              |                                                                                                                                                                               |
| Security operations (SecOps)       | Security administrator role with a horizontal view across the entire Azure estate and the Azure Key Vault purge policy | `*/read`, ` */register/action`,, ` Microsoft.Authorization/policyAssignments/*`, ` Microsoft.Authorization/policyDefinitions/*`, ` Microsoft.Authorization/policyExemptions/*`, ` Microsoft.Authorization/policySetDefinitions/*`, ` Microsoft.Insights/alertRules/*`,  ` Microsoft.Resources/deployments/*`, ` Microsoft.Security/*`, ` Microsoft.Support/*`, ` Microsoft.KeyVault/locations/deletedVaults/purge/action`, ` Microsoft.PolicyInsights/*` |                                                                            |
| Subscription owner                 | Delegated role for subscription owner derived from subscription Owner role, no modify rights on role assignment and routes                                       | `*`                                                                                                                                                                                                                  | `Microsoft.Authorization/*/write`, `Microsoft.Authorization/*/Delete`, `Microsoft.Authorization/elevateAccess/Action`, `Microsoft.Network/vpnGateways/*`, `Microsoft.Network/expressRouteCircuits/*`, `Microsoft.Network/routeTables/write`, `Microsoft.Network/routeTables/delete`, `Microsoft.Network/routeTables/routes/write`, `Microsoft.Network/routeTables/routes/delete`, `Microsoft.Network/vpnSites/*` |
| Subscription owner no network modify rights               | Delegated role for subscription owner derived from subscription contributor role, no modify rights on role assignment and routes and no network write delete permissions                                      | `*`                                                                                                                                                                                                                  | `Microsoft.Authorization/*/write`, `Microsoft.Authorization/*/Delete`, `Microsoft.Authorization/elevateAccess/Action`, `Microsoft.Network/virtualNetworks/write`, `Microsoft.Network/virtualNetworks/delete`, `Microsoft.Network/vpnGateways/*`, `Microsoft.Network/azurefirewalls/*`, `Microsoft.Network/expressRouteCircuits/*`, `Microsoft.Network/routeTables/write`, `Microsoft.Network/routeTables/delete`, `Microsoft.Network/routeTables/routes/write`, `Microsoft.Network/routeTables/routes/delete`, `Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write`, `Microsoft.Network/virtualNetworks/virtualNetworkPeerings/delete`, `Microsoft.Network/vpnSites/*` |
| Application owners (DevOps/AppOps) | Contributor role granted for application/operations team at resource group level, can also be applied on subscription. This role derived from contributor role with additional restrictions to modify network resources. Consider when using this role only on the resource group level to add the delete resource group right to the notactions  | `*`                                                                                                                                                                                                                   | `Microsoft.Authorization/*/write`, `Microsoft.Authorization/*/Delete`, `Microsoft.Authorization/elevateAccess/Action`, `Microsoft.Network/publicIPAddresses/write`, `Microsoft.Network/virtualNetworks/write`, `Microsoft.Network/virtualNetworks/delete`, `Microsoft.KeyVault/locations/deletedVaults/purge/action`, `Microsoft.Blueprint/blueprintAssignments/write`, `Microsoft.Blueprint/blueprintAssignments/delete`, `Microsoft.Compute/galleries/share/action`, `Microsoft.Network/vpnGateways/*`, `Microsoft.Network/azurefirewalls/*`, `Microsoft.Network/expressRouteCircuits/*`, `Microsoft.Network/routeTables/write`, `Microsoft.Network/routeTables/delete`, `Microsoft.Network/routeTables/routes/write`, `Microsoft.Network/routeTables/routes/delete`, `Microsoft.Network/networkSecurityGroups/securityRules/write`, `Microsoft.Network/networkSecurityGroups/securityRules/delete`, `Microsoft.Network/networkSecurityGroups/write`, `Microsoft.Network/networkSecurityGroups/delete`, `Microsoft.Network/networkSecurityGroups/join/action`, `Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write`, `Microsoft.Network/virtualNetworks/virtualNetworkPeerings/delete`, `Microsoft.Network/virtualNetworks/subnets/write`, `Microsoft.Network/virtualNetworks/subnets/delete`, `Microsoft.Network/vpnSites/*`                                     |
| Application owners hybrid network access (DevOps/AppOps) | Give application owners the right to create network interfaces in the hybrid network. This role should be applied only to the hybrid network resource group. This should only be used in when Application owner (DevOps/AppOps) is used on resource group level, this is not needed when this role is applied on subscription level | `Microsoft.Authorization/*/read`, `Microsoft.Insights/alertRules/*`, `Microsoft.Network/applicationGateways/backendAddressPools/join/action`, `Microsoft.Network/applicationSecurityGroups/read`, `Microsoft.Network/applicationSecurityGroups/joinIpConfiguration/action`, `Microsoft.Network/loadBalancers/backendAddressPools/join/action`, `Microsoft.Network/loadBalancers/inboundNatPools/join/action`, `Microsoft.Network/loadBalancers/inboundNatRules/join/action`, `Microsoft.Network/loadBalancers/probes/join/action`, `Microsoft.Network/loadBalancers/read`, `Microsoft.Network/networkInterfaces/*`, `Microsoft.Network/locations/*`, `Microsoft.Network/networkSecurityGroups/join/action`, `Microsoft.Network/networkSecurityGroups/read`, `Microsoft.Network/publicIPAddresses/join/action`, `Microsoft.Network/publicIPAddresses/read`, `Microsoft.Network/virtualNetworks/read`, `Microsoft.Network/virtualNetworks/subnets/read`, `Microsoft.Network/virtualNetworks/subnets/join/action`, `Microsoft.ResourceHealth/availabilityStatuses/read`, `Microsoft.Resources/deployments/*`, `Microsoft.Resources/subscriptions/resourceGroups/read`, `Microsoft.Support/*`                                                                                                                                                                                                                 |                                     |


- Use Azure Security Center just-in-time access for all infrastructure as a service (IaaS) resources to enable network-level protection for ephemeral user access to IaaS virtual machines.
- Use Azure-AD-managed identities for Azure resources to avoid authentication based on user names and passwords. Because many security breaches of public cloud resources originate with credential theft embedded in code or other text sources, enforcing managed identities for programmatic access greatly reduces the risk of credential theft.
- Use privileged identities for automation runbooks that require elevated access permissions. Automated workflows that violate critical security boundaries should be governed by the same tools and policies users of equivalent privilege are.
- Don't add users directly to Azure resource scopes. Instead add users to defined roles, which are then assigned to resource scopes. Direct user assignments circumvent centralized management, greatly increasing the management required to prevent unauthorized access to restricted data.

### Plan for authentication inside a landing zone

A critical design decision that an enterprise organization must make when adopting Azure is whether to extend an existing on-premises identity domain into Azure or to create a brand new one. Requirements for authentication inside the landing zone should be thoroughly assessed and incorporated into plans to deploy Active Directory Domain Services (AD DS) in Windows Server, Azure AD Domain Services (Azure AD DS), or both. Most Azure environments will use at least Azure AD for Azure fabric authentication and AD DS local host authentication and group policy management.

**Design considerations:**

- Consider centralized and delegated responsibilities to manage resources deployed inside the landing zone.
- Applications that rely on domain services and use older protocols can use [Azure AD DS](/azure/active-directory-domain-services).

**Design recommendations:**

- Use centralized and delegated responsibilities to manage resources deployed inside the landing zone based on role and security requirements.
- Privileged operations such as creating service principal objects, registering applications in Azure AD, and procuring and handling certificates or wildcard certificates require special permissions. Consider which users will be handling such requests and how to secure and monitor their accounts with the degree of diligence required.
- If an organization has a scenario where an application that uses integrated Windows authentication must be accessed remotely through Azure AD, consider using [Azure AD Application Proxy](/azure/active-directory/manage-apps/application-proxy).
- There's a difference between Azure AD, Azure AD DS, and AD DS running on Windows Server. Evaluate your application needs, and understand and document the authentication provider that each one will be using. Plan accordingly for all applications.
- Evaluate the compatibility of workloads for AD DS on Windows Server and for Azure AD DS.
- Ensure your network design allows resources that require AD DS on Windows Server for local authentication and management to access the appropriate domain controllers.
  - For AD DS on Windows Server, consider shared services environments that offer local authentication and host management in a larger enterprise-wide network context.
- Deploy Azure AD DS within the primary region because this service can only be projected into one subscription.
- Use managed identities instead of service principals for authentication to Azure services. This approach reduces exposure to credential theft.
