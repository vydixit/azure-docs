---
title: Auto-deploy agents for Azure Security Center | Microsoft Docs
description: This article describes how to set up auto provisioning of the Log Analytics agent and other agents and extensions used by Azure Security Center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: quickstart
ms.date: 10/08/2021
ms.author: memildin

---
# Configure auto provisioning for agents and extensions from Azure Security Center

Azure Security Center collects data from your resources using the relevant agent or extensions for that resource and the type of data collection you've enabled. Use the procedures below to ensure your resources have the necessary agents and extensions used by Security Center.

## Prerequisites
To get started with Security Center, you must have a subscription to Microsoft Azure. If you don't have a subscription, you can sign up for a [free account](https://azure.microsoft.com/pricing/free-trial/).

## Availability

| Aspect                  | Details                                                                                                                                                                                                                      |
|-------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Release state:          | **Feature**: Auto provisioning is generally available (GA)<br>**Agent and extensions**: Log Analytics agent for Azure VMs is GA, Microsoft Dependency agent is in preview, Policy Add-on for Kubernetes is GA, Guest Configuration agent is preview  |
| Pricing:                | Free                                                                                                                                                                                                                         |
| Supported destinations: | :::image type="icon" source="./media/icons/yes-icon.png"::: Azure machines<br>:::image type="icon" source="./media/icons/no-icon.png"::: Azure Arc machines<br>:::image type="icon" source="./media/icons/no-icon.png"::: Kubernetes nodes<br>:::image type="icon" source="./media/icons/no-icon.png"::: Virtual Machine Scale Sets |
| Clouds:                 | **Feature**:<br>:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/yes-icon.png"::: Azure Government, Azure China 21Vianet<br>**Agent and extensions**:<br>Log Analytics agent for Azure VMs is available on all clouds, Policy Add-on for Kubernetes is available on all clouds, Guest Configuration agent is only available on commercial clouds  |
|                         |                                                                                                                                                                                                                              |

## How does Security Center collect data?

Security Center collects data from your Azure virtual machines (VMs), virtual machine scale sets, IaaS containers, and non-Azure (including on-premises) machines to monitor for security vulnerabilities and threats. 

Data collection is required to provide visibility into missing updates, misconfigured OS security settings, endpoint protection status, and health and threat protection. Data collection is only needed for compute resources such as VMs, virtual machine scale sets, IaaS containers, and non-Azure computers. 

You can benefit from Azure Security Center even if you don’t provision agents. However, you'll have limited security and the capabilities listed above aren't supported.  

Data is collected using:

- The **Log Analytics agent**, which reads various security-related configurations and event logs from the machine and copies the data to your workspace for analysis. Examples of such data are: operating system type and version, operating system logs (Windows event logs), running processes, machine name, IP addresses, and logged in user.
- **Security extensions**, such as the [Azure Policy Add-on for Kubernetes](../governance/policy/concepts/policy-for-kubernetes.md), which can also provide data to Security Center regarding specialized resource types.

> [!TIP]
> As Security Center has grown, the types of resources that can be monitored has also grown. The number of extensions has also grown. Auto provisioning has expanded to support additional resource types by leveraging the capabilities of Azure Policy.

## Why use auto provisioning?
Any of the agents and extensions described on this page *can* be installed manually (see [Manual installation of the Log Analytics agent](#manual-agent)). However, **auto provisioning** reduces management overhead by installing all required agents and extensions on existing - and new - machines to ensure faster security coverage for all supported resources. 

We recommend enabling auto provisioning, but it's disabled by default.

## How does auto provisioning work?
Security Center's auto provisioning settings have a toggle for each type of supported extension. When you enable auto provisioning of an extension, you assign the appropriate **Deploy if not exists** policy. This policy type ensures the extension is provisioned on all existing and future resources of that type.

> [!TIP]
> Learn more about Azure Policy effects including deploy if not exists in [Understand Azure Policy effects](../governance/policy/concepts/effects.md).


## Enable auto provisioning of the Log Analytics agent and extensions <a name="auto-provision-mma"></a>

When automatic provisioning is on for the Log Analytics agent, Security Center deploys the agent on all supported Azure VMs and any new ones created. For the list of supported platforms, see [Supported platforms in Azure Security Center](security-center-os-coverage.md).

To enable auto provisioning of the Log Analytics agent:

1. From Security Center's menu, select **Pricing & settings**.
1. Select the relevant subscription.
1. In the **Auto provisioning** page, set the status of auto provisioning for the Log Analytics agent to **On**.

    :::image type="content" source="./media/security-center-enable-data-collection/enable-automatic-provisioning.png" alt-text="Enabling auto-provisioning of the Log Analytics agent." lightbox="./media/security-center-enable-data-collection/enable-automatic-provisioning.png":::

1. From the configuration options pane, define the workspace to use.

    :::image type="content" source="./media/security-center-enable-data-collection/log-analytics-agent-deploy-options.png" alt-text="Configuration options for auto provisioning Log Analytics agents to VMs." lightbox="./media/security-center-enable-data-collection/log-analytics-agent-deploy-options.png":::

    - **Connect Azure VMs to the default workspace(s) created by Security Center** - Security Center creates a new resource group and default workspace in the same geolocation, and connects the agent to that workspace. If a subscription contains VMs from multiple geolocations, Security Center creates multiple workspaces to ensure compliance with data privacy requirements.

        The naming convention for the workspace and resource group is: 
        - Workspace: DefaultWorkspace-[subscription-ID]-[geo] 
        - Resource Group: DefaultResourceGroup-[geo] 

        Security Center automatically enables a Security Center solution on the workspace per the pricing tier set for the subscription. 

        > [!TIP]
        > For questions regarding default workspaces, see:
        >
        > - [Am I billed for Azure Monitor logs on the workspaces created by Security Center?](./faq-data-collection-agents.yml#am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center-)
        > - [Where is the default Log Analytics workspace created?](./faq-data-collection-agents.yml#where-is-the-default-log-analytics-workspace-created-)
        > - [Can I delete the default workspaces created by Security Center?](./faq-data-collection-agents.yml#can-i-delete-the-default-workspaces-created-by-security-center-)

    - **Connect Azure VMs to a different workspace** - From the dropdown list, select the workspace to store collected data. The dropdown list includes all workspaces across all of your subscriptions. You can use this option to collect data from virtual machines running in different subscriptions and store it all in your selected workspace.  

        If you already have an existing Log Analytics workspace, you might want to use the same workspace (requires read and write permissions on the workspace). This option is useful if you're using a centralized workspace in your organization and want to use it for security data collection. Learn more in [Manage access to log data and workspaces in Azure Monitor](../azure-monitor/logs/manage-access.md).

        If your selected workspace already has a "Security" or "SecurityCenterFree" solution enabled, the pricing will be set automatically. If not, install a Security Center solution on the workspace:

        1. From Security Center's menu, open **Pricing & settings**.
        1. Select the workspace to which you'll be connecting the agents.
        1. Select **Azure Defender on** or **Azure Defender off**.

1. From the **Windows security events** configuration, select the amount of raw event data to store:
    - **None** – Disable security event storage. This is the default setting.
    - **Minimal** – A small set of events for when you want to minimize the event volume.
    - **Common** – A set of events that satisfies most customers and provides a full audit trail.
    - **All events** – For customers who want to make sure all events are stored.

    > [!TIP]
    > To set these options at the workspace level, see [Setting the security event option at the workspace level](#setting-the-security-event-option-at-the-workspace-level).
    > 
    > For more information of these options, see [Windows security event options for the Log Analytics agent](#data-collection-tier).

1. Select **Apply** in the configuration pane.

1. To enable automatic provisioning of an extension other than the Log Analytics agent: 

    1. If you're enabling auto provisioning for the Microsoft Dependency agent, ensure the Log Analytics agent is set to auto deploy.
    1. Toggle the status to **On** for the relevant extension.

        :::image type="content" source="./media/security-center-enable-data-collection/toggle-kubernetes-add-on.png" alt-text="Toggle to enable auto provisioning for K8s policy add-on.":::

    1. Select **Save**. The Azure Policy definition is assigned and a remediation task is created.

        |Extension  |Policy  |
        |---------|---------|
        |Policy Add-on for Kubernetes                      |[Deploy Azure Policy Add-on to Azure Kubernetes Service clusters](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fa8eff44f-8c92-45c3-a3fb-9880802d67a7)|
        |Microsoft Dependency agent (preview) (Windows VMs)|[Deploy Dependency agent for Windows virtual machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f1c210e94-a481-4beb-95fa-1571b434fb04)         |
        |Microsoft Dependency agent (preview) (Linux VMs)  |[Deploy Dependency agent for Linux virtual machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4da21710-ce6f-4e06-8cdb-5cc4c93ffbee)|
        |Guest Configuration agent (preview)               |[Deploy prerequisites to enable Guest Configuration policies on virtual machines](https://github.com/Azure/azure-policy/blob/64dcfa3033a3ff231ec4e73d2c1dad4db4e3b5dd/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json)|
        |||

1. Select **Save**. If a workspace needs to be provisioned, agent installation might take up to 25 minutes.

1. You'll be asked if you want to reconfigure monitored VMs that were previously connected to a default workspace:

    :::image type="content" source="./media/security-center-enable-data-collection/reconfigure-monitored-vm.png" alt-text="Review options to reconfigure monitored VMs.":::

    - **No** - your new workspace settings will only be applied to newly discovered VMs that don't have the Log Analytics agent installed.
    - **Yes** - your new workspace settings will apply to all VMs and every VM currently connected to a Security Center created workspace will be reconnected to the new target workspace.

   > [!NOTE]
   > If you select **Yes**, don't delete the workspace(s) created by Security Center until all VMs have been reconnected to the new target workspace. This operation fails if a workspace is deleted too early.


## Windows security event options for the Log Analytics agent <a name="data-collection-tier"></a> 

Selecting a data collection tier in Azure Security Center only affects the *storage* of security events in your Log Analytics workspace. The Log Analytics agent will still collect and analyze the security events required for Security Center’s threat protection, regardless of the level of security events you choose to store in your workspace. Choosing to store security events enables investigation, search, and auditing of those events in your workspace.

### Requirements 
Azure Defender is required for storing Windows security event data. [Learn more about Azure Defender](azure-defender.md).

Storing data in Log Analytics might incur additional charges for data storage. For more information, see the [pricing page](https://azure.microsoft.com/pricing/details/security-center/).

### Information for Azure Sentinel users 
Users of Azure Sentinel: note that security events collection within the context of a single workspace can be configured from either Azure Security Center or Azure Sentinel, but not both. If you're planning to add Azure Sentinel to a workspace that is already getting alerts from Azure Security Center, and is set to collect Security Events, you have two options:
- Leave the Security Events collection in Azure Security Center as is. You will be able to query and analyze these events in Azure Sentinel as well as in Azure Defender. You will not, however, be able to monitor the connector's connectivity status or change its configuration in Azure Sentinel. If this is important to you, consider the second option.
- Disable Security Events collection in Azure Security Center (by setting **Windows security events** to
**None** in the configuration of your Log Analytics agent). Then add the Security Events connector in Azure Sentinel. As with the first option, you will be able to query and analyze events in both Azure Sentinel and Azure Defender/ASC, but you will now be able to monitor the connector's connectivity status or change its configuration in - and only in - Azure Sentinel.

### What event types are stored for "Common" and "Minimal"?
These sets were designed to address typical scenarios. Make sure to evaluate which one fits your needs before implementing it.

To determine the events for the **Common** and **Minimal** options, we worked with customers and industry standards to learn about the unfiltered frequency of each event and their usage. We used the following guidelines in this process:

- **Minimal** - Make sure that this set covers only events that might indicate a successful breach and important events that have a very low volume. For example, this set contains user successful and failed login (event IDs 4624, 4625), but it doesn’t contain sign out which is important for auditing but not meaningful for detection and has relatively high volume. Most of the data volume of this set is the login events and process creation event (event ID 4688).
- **Common** - Provide a full user audit trail in this set. For example, this set contains both user logins and user sign outs (event ID 4634). We include auditing actions like security group changes, key domain controller Kerberos operations, and other events that are recommended by industry organizations.

Events that have very low volume were included in the common set as the main motivation to choose it over all the events is to reduce the volume and not to filter out specific events.

Here is a complete breakdown of the Security and App Locker event IDs for each set:

| Data tier | Collected event indicators |
| --- | --- |
| Minimal | 1102,4624,4625,4657,4663,4688,4700,4702,4719,4720,4722,4723,4724,4727,4728,4732,4735,4737,4739,4740,4754,4755, |
| | 4756,4767,4799,4825,4946,4948,4956,5024,5033,8001,8002,8003,8004,8005,8006,8007,8222 |
| Common | 1,299,300,324,340,403,404,410,411,412,413,431,500,501,1100,1102,1107,1108,4608,4610,4611,4614,4622, |
| |  4624,4625,4634,4647,4648,4649,4657,4661,4662,4663,4665,4666,4667,4688,4670,4672,4673,4674,4675,4689,4697, |
| | 4700,4702,4704,4705,4716,4717,4718,4719,4720,4722,4723,4724,4725,4726,4727,4728,4729,4733,4732,4735,4737, |
| | 4738,4739,4740,4742,4744,4745,4746,4750,4751,4752,4754,4755,4756,4757,4760,4761,4762,4764,4767,4768,4771, |
| | 4774,4778,4779,4781,4793,4797,4798,4799,4800,4801,4802,4803,4825,4826,4870,4886,4887,4888,4893,4898,4902, |
| | 4904,4905,4907,4931,4932,4933,4946,4948,4956,4985,5024,5033,5059,5136,5137,5140,5145,5632,6144,6145,6272, |
| | 6273,6278,6416,6423,6424,8001,8002,8003,8004,8005,8006,8007,8222,26401,30004 |

> [!NOTE]
> - If you are using Group Policy Object (GPO), it is recommended that you enable audit policies Process Creation Event 4688 and the *CommandLine* field inside event 4688. For more information about Process Creation Event 4688, see Security Center's [FAQ](./faq-data-collection-agents.yml#what-happens-when-data-collection-is-enabled-). For more information about these audit policies, see [Audit Policy Recommendations](/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations).
> -  To enable data collection for [Adaptive Application Controls](security-center-adaptive-application.md), Security Center configures a local AppLocker policy in Audit mode to allow all applications. This will cause AppLocker to generate events which are then collected and leveraged by Security Center. It is important to note that this policy will not be configured on any machines on which there is already a configured AppLocker policy. 
> - To collect Windows Filtering Platform [Event ID 5156](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=5156), you need to enable [Audit Filtering Platform Connection](/windows/security/threat-protection/auditing/audit-filtering-platform-connection) (Auditpol /set /subcategory:"Filtering Platform Connection" /Success:Enable)
>

### Setting the security event option at the workspace level

You can define the level of security event data to store at the workspace level.

1. From Security Center's menu in the Azure portal, select **Pricing & settings**.
1. Select the relevant workspace. The only data collection events for a workspace are the Windows security events described on this page.

    :::image type="content" source="media/security-center-enable-data-collection/event-collection-workspace.png" alt-text="Setting the security event data to store in a workspace.":::

1. Select the amount of raw event data to store and select **Save**.

## Manual agent provisioning <a name="manual-agent"></a>
 
To manually install the Log Analytics agent:

1. Disable auto provisioning.

1. Optionally, create a workspace.

1. Enable Azure Defender on the workspace on which you're installing the Log Analytics agent:

    1. From Security Center's menu, select **Pricing & settings**.

    1. Set the workspace on which you're installing the agent. Make sure the workspace is in the same subscription you use in Security Center and that you have read/write permissions for the workspace.

    1. Select **Azure Defender on**, and **Save**.

       >[!NOTE]
       >If the workspace already has a **Security** or **SecurityCenterFree** solution enabled, the pricing will be set automatically. 

1. To deploy agents on new VMs using a Resource Manager template, install the Log Analytics agent:

   - [Install the Log Analytics agent for Windows](../virtual-machines/extensions/oms-windows.md)
   - [Install the Log Analytics agent for Linux](../virtual-machines/extensions/oms-linux.md)

1. To deploy agents on your existing VMs, follow the instructions in [Collect data about Azure Virtual Machines](../azure-monitor/vm/monitor-virtual-machine.md) (the section **Collect event and performance data** is optional).

1. To use PowerShell to deploy the agents, use the instructions from the virtual machines documentation:

    - [For Windows machines](../virtual-machines/extensions/oms-windows.md?toc=%2fazure%2fazure-monitor%2ftoc.json#powershell-deployment)
    - [For Linux machines](../virtual-machines/extensions/oms-linux.md?toc=%2fazure%2fazure-monitor%2ftoc.json#azure-cli-deployment)

> [!TIP]
> For instructions on how to onboard Security Center using PowerShell, see [Automate onboarding of Azure Security Center using PowerShell](security-center-powershell-onboarding.md).


## Automatic provisioning in cases of a pre-existing agent installation <a name="preexisting"></a> 

The following use cases specify how automatic provision works in cases when there is already an agent or extension installed. 

- **Log Analytics agent is installed on the machine, but not as an extension (Direct agent)** - If the Log Analytics agent is installed directly on the VM (not as an Azure extension), Security Center will install the Log Analytics agent extension, and might upgrade the Log Analytics agent to the latest version.
The agent installed will continue to report to its already configured workspace(s), and additionally will report to the workspace configured in Security Center (Multi-homing is supported on Windows machines).
If the configured workspace is a user workspace (not Security Center's default workspace), then you will need to install the "Security" or "SecurityCenterFree" solution on it for Security Center to start processing events from VMs and computers reporting to that workspace.

    For Linux machines, Agent multi-homing is not yet supported - hence, if an existing agent installation is detected, automatic provisioning will not occur and the machine's configuration will not be altered.

    For existing machines on subscriptions onboarded to Security Center before 17 March 2019, when an existing agent will be detected, the Log Analytics agent extension will not be installed and the machine will not be affected. For these machines, see to the "Resolve monitoring agent health issues on your machines" recommendation to resolve the agent installation issues on these machines.
  
- **System Center Operations Manager agent is installed on the machine** - Security center will install the Log Analytics agent extension side by side to the existing Operations Manager. The existing Operations Manager agent will continue to report to the Operations Manager server normally. The Operations Manager agent and Log Analytics agent share common run-time libraries, which will be updated to the latest version during this process. If Operations Manager agent version 2012 is installed, **do not** enable automatic provisioning.

- **A pre-existing VM extension is present**:
    - When the Monitoring Agent is installed as an extension, the extension configuration allows reporting to only a single workspace. Security Center does not override existing connections to user workspaces. Security Center will store security data from the VM in the workspace already connected, provided that the "Security" or "SecurityCenterFree" solution has been installed on it. Security Center may upgrade the extension version to the latest version in this process.
    - To see to which workspace the existing extension is sending data to, run the test to [Validate connectivity with Azure Security Center](/archive/blogs/yuridiogenes/validating-connectivity-with-azure-security-center). Alternatively, you can open Log Analytics workspaces, select a workspace, select the VM, and look at the Log Analytics agent connection.
    - If you have an environment where the Log Analytics agent is installed on client workstations and reporting to an existing Log Analytics workspace, review the list of [operating systems supported by Azure Security Center](security-center-os-coverage.md) to make sure your operating system is supported. For more information, see [Existing log analytics customers](./faq-azure-monitor-logs.yml).
 

## Disable auto provisioning <a name="offprovisioning"></a>

When you disable auto provisioning, agents will not be provisioned on new VMs.

To turn off automatic provisioning of an agent:

1. From Security Center's menu in the portal, select **Pricing & settings**.
1. Select the relevant subscription.
1. Select **Auto provisioning**.
1. Toggle the status to **Off** for the relevant agent.

    :::image type="content" source="./media/security-center-enable-data-collection/agent-toggles.png" alt-text="Toggles to disable auto provisioning per agent type.":::

1. Select **Save**. When auto provisioning is disabled, the default workspace configuration section is not displayed:

    :::image type="content" source="./media/security-center-enable-data-collection/empty-configuration-column.png" alt-text="When auto provisioning is disabled, the configuration cell is empty":::


> [!NOTE]
>  Disabling automatic provisioning does not remove the Log Analytics agent from Azure VMs where the agent was provisioned. For information on removing the OMS extension, see [How do I remove OMS extensions installed by Security Center](./faq-data-collection-agents.yml#how-do-i-remove-oms-extensions-installed-by-security-center-).
>


## Troubleshooting

-	To identify auto provision installation issues, see [Monitoring agent health issues](security-center-troubleshooting-guide.md#mon-agent).
-  To identify monitoring agent network requirements, see [Troubleshooting monitoring agent network requirements](security-center-troubleshooting-guide.md#mon-network-req).
-	To identify manual onboarding issues, see [How to troubleshoot Operations Management Suite onboarding issues](https://support.microsoft.com/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues).



## Next steps
This page explained how to enable auto provisioning for the Log Analytics agent and other Security Center extensions. It also described how to define a Log Analytics workspace in which to store the collected data. Both operations are required to enable data collection. Storing data in Log Analytics, whether you use a new or existing workspace, might incur more charges for data storage. For pricing details in your currency of choice and according to your region, see [Security Center pricing](https://azure.microsoft.com/pricing/details/security-center/).