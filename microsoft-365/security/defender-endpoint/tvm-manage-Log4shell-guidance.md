---
title: Learn how to mitigate the Log4Shell vulnerability in Microsoft Defender for Endpoint - threat and vulnerability management
description: Learn how to mitigate the Log4Shell vulnerability in Microsoft Defender for Endpoint 
keywords: tvm, lo4j
ms.prod: m365-security
ms.mktglfcycl: deploy
ms.sitesec: library
ms.pagetype: security
f1.keywords:
- NOCSH
ms.author: siosulli
author: siosulli
ms.localizationpriority: medium
manager: dansimp
audience: ITPro
ms.collection:
- M365-security-compliance
- m365initiative-m365-defender
- m365-initiative-defender-endpoint
ms.custom: admindeeplinkDEFENDER
ms.topic: conceptual
ms.technology: m365d
---

# Learn how to manage the Log4Shell vulnerability in Microsoft Defender for Endpoint

**Applies to:**

- [Microsoft Defender for Endpoint Plan 2](https://go.microsoft.com/fwlink/?linkid=2154037)
- [Threat and vulnerability management](next-gen-threat-and-vuln-mgt.md)
- [Microsoft 365 Defender](https://go.microsoft.com/fwlink/?linkid=2118804)

The Log4Shell vulnerability is a remote code execution (RCE) vulnerability found in the Apache Log4j 2 logging library. As Apache Log4j 2 is commonly used by many software applications and online services, it represents a complex and high-risk situation for companies across the globe. Referred to as “Log4Shell” ([CVE-2021-44228](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2021-44228), [CVE-2021-45046](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-45046) ) it introduces a new attack vector that attackers can exploit to extract data and deploy ransomware in an organization.

> [!NOTE]
> Refer to the blogs [Guidance for preventing, detecting, and hunting for exploitation of the Log4j 2 vulnerability and](https://www.microsoft.com/security/blog/2021/12/11/guidance-for-preventing-detecting-and-hunting-for-cve-2021-44228-log4j-2-exploitation/) [Microsoft Security Response Center](https://msrc-blog.microsoft.com/2021/12/11/microsofts-response-to-cve-2021-44228-apache-log4j2/) for guidance and technical information about the vulnerability and product specific mitigation recommendations to protect your organization.

## Overview of discovery, monitoring and mitigation capabilities

Threat and vulnerability management provides you with the following capabilities to help you identify, monitor, and mitigate your organizational exposure to the Log4Shell vulnerability:

- **Discovery**: Detection of exposed devices, both Microsoft Defender for Endpoint onboarded devices as well as devices that have been discovered but are not yet onboarded, is based on vulnerable software and vulnerable files detected on disk.
- **Threat awareness:** A consolidated view to assess your organizational exposure. This view shows your exposure at the device level and software level, and provides access to details on vulnerable files like, the last time it was seen, the last time it was executed and the last time it was executed with open ports. You can use this information to prioritize your remediation actions. It can take up to 24 hours for data related to exposed devices to appear on the dashboard.
- **Mitigation options:** Apply mitigation options to help lower your exposure risk.
- **Advanced hunting:** Use advanced hunting to return details for vulnerable log4j files identified on disk.

> [!NOTE]
> These capabilities are supported on Windows 10 & Windows 11, Windows Server, Linux and macOS.
>
> Support on Linux requires Microsoft Defender for Endpoint Linux client version 101.52.57 (30.121092.15257.0) or later.
>
> Support on macOS requires Microsoft Defender for Endpoint macOS client version 20.121111.15416.0 or later.
>
>For more information on supported versions, see [Supported operating systems platforms and capabilities](tvm-supported-os.md).

## Exposed devices discovery

Embedded threat and vulnerability management capabilities, along with enabling Log4j detection, in the Microsoft 365 Defender portal, will help you discover devices exposed to the Log4Shell vulnerability.

Onboarded devices, are assessed using existing embedded threat and vulnerability management capabilities that can discover vulnerable software and files.

For detection on discovered but not yet onboarded devices, Log4j detection must be enabled. This will initiate probes in the same way device discovery actively probes your network. This includes probing from multiple onboarded endpoints (Windows 10+ and Windows Server 2019+ devices) and only probing within subnets, to detect devices that are vulnerable and remotely exposed to CVE-2021-44228.

To enable Log4 detection:

1. Go to **Settings** > **Device discovery** > **Discovery setup**
2. Select **Enable Log4j2 detection (CVE-2021-44228)**
3. Select **Save**

:::image type="content" source="images/enable_log4j.png" alt-text="Setting to enable log4j2 detection" lightbox="images/enable_log4j.png":::

Running these probes will trigger the standard Log4j flow without causing any harmful impact on either the device being probed or the probing device. The probing itself is done by sending multiple HTTP requests to discovered devices, targeting common web application ports (for example - 80,8000,8080,443,8443) and URLs. The request contains HTTP headers with a JNDI payload that triggers a DNS request from the probed machine.

For example, User-Agent: ${jndi:dns://192.168.1.3:5353/MDEDiscoveryUser-Agent} where 192.168.1.3 is the IP of the probing machine.

> [!NOTE]
> Enabling Log4j2 detection also means onboarded devices will use self-probing to detect local vulnerabilities.

## Vulnerable software and files detection

Threat and vulnerability management provides layers of detection to help you discover:

- **Vulnerable software**: Discovery is based on installed application Common Platform Enumerations (CPE) that are known to be vulnerable to Log4j remote code execution.
- **Vulnerable files:** Both files in memory and files in the file system are assessed. These files can be Log4j-core jar files with the known vulnerable version or an Uber-JAR that contains either a vulnerable jndi lookup class or a vulnerable log4j-core file. Specifically, it:

  - determines if a JAR file contains a vulnerable Log4j file by examining JAR files and searching for the following file:
      \\META-INF\\maven\\org.apache.logging.log4j\\log4j-core\\pom.properties - if this file exists, the Log4j version is read and extracted.
  - searches for the JndiLookup.class file inside the JAR file by looking for paths that contain the string “/log4j/core/lookup/JndiLookup.class” - if the JndiLookup.class file exists, threat and vulnerability management determines if this JAR contains a Log4j file with the version defined in pom.properties.
  - searches for any vulnerable Log4j-core JAR files embedded within a nested-JAR by searching for paths that contain any of these strings:
    - lib/log4j-core-
    - WEB-INF/lib/log4j-core-
    - App-INF/lib/log4j-core-

This table describes the search capabilities supported platforms and versions:

|Capability|File Type|Windows10+,<br>server2019+|Server 2012R2,<br>server2016|Server 2008R2|Linux + macOS|
|:---|:---|:---|:---|:---|:---|
|Search In Memory  | Log4j-core | Yes |Yes<sup>[1]| - | Yes |
| |Uber-JARs | Yes |Yes<sup>[1]| - | Yes |
| Search all files on disk  |Log4j-core | Yes |Yes<sup>[1]| Yes | - |
| | Uber-JARs|Yes |Yes<sup>[1]| - | -|

(1) Capabilities are available when [KB5005292](https://support.microsoft.com/topic/microsoft-defender-for-endpoint-update-for-edr-sensor-f8f69773-f17f-420f-91f4-a8e5167284ac) is installed on Windows Server 2012 R2 and 2016.

## Learn about your Log4Shell exposure and mitigation options

### Threat and vulnerability management dashboard

Use the threat and vulnerability management dashboard to see your current exposure.

1. In the Microsoft 365 Defender portal, go to **Vulnerability management** > **Dashboard** > **Threat awareness:**
:::image type="content" source="images/awareness_dashboard.png" alt-text="The threat awareness widget on the vulnerability management dashboard" lightbox="images/awareness_dashboard.png":::
2. Select **View vulnerability details** to see the consolidated view of your organizational exposure.
:::image type="content" source="images/view_vulnerability_details.png" alt-text="The vulnerability details page for CVE-2021-44228 (Log4j)" lightbox="images/view_vulnerability_details.png":::
3. Choose the relevant tab to see your exposure broken down by:
    - Exposed devices – onboard
    - Exposed devices – not onboarded
    - Vulnerable files
    - Vulnerable software

### Log4Shell vulnerability mitigation

The log4Shell vulnerability can be mitigated by preventing JNDI lookups on Log4j versions 2.10 - 2.14.1 with default configurations. To create this mitigation action, from the **Threat awareness dashboard**:

1. Select **View vulnerability details**
2. Select **Mitigation options**

You can choose to apply the mitigation to all exposed devices or select specific onboarded devices. To complete the process and apply the mitigation on devices, select **Create mitigation action**.

:::image type="content" source="images/mitigation_options.png" alt-text="Mitigation options for CVE-2021-44228" lightbox="images/mitigation_options.png":::

### Mitigation status

The mitigation status indicates whether the workaround mitigation to disable JDNI lookups has been applied to the device. You can view the mitigation status for each affected device in the Exposed devices tabs. This can help prioritize mitigation and/or patching of devices based on their mitigation status.

:::image type="content" source="images/mitigation_status.png" alt-text="Possible mitigation statuses" lightbox="/mitigation_status.png":::

The table below lists the potential mitigation statuses:

| Mitigation status | Description |
|:---|:---|
| Workaround applied | _Windows_: The LOG4J_FORMAT_MSG_NO_LOOKUPS environment variable was observed before latest device reboot. <br/><br/> _Linux + macOS_: All running processes have LOG4J_FORMAT_MSG_NO_LOOKUPS=true in its environment variables. |
| Workaround pending reboot | The LOG4J_FORMAT_MSG_NO_LOOKUPS environment variable is set, but no following reboot detected. |
| Not applied | _Windows_: The LOG4J_FORMAT_MSG_NO_LOOKUPS environment variable was not observed. <br/><br/> _Linux + macOS_: Not all running processes have LOG4J_FORMAT_MSG_NO_LOOKUPS=true in its environment variables, and mitigation action was not applied on device. |
| Partially mitigated | _Linux + macOS_: Although mitigation action was applied on device, not all running processes have LOG4J_FORMAT_MSG_NO_LOOKUPS=true in its environment variables. |
|Not applicable | Devices that have vulnerable files that are not in the version range of the mitigation. |
|Unknown | The mitigation status couldn’t be determined at this time. |

> [!NOTE]
> It may take a few hours for the updated mitigation status of a device to be reflected.

### Revert mitigations applied for the Log4Shell vulnerability

In cases where the mitigation needs to be reverted, follow these steps:

**_For Windows:_**

1. Open an elevated PowerShell window
2. Run the following command:

 ```Powershell
   [Environment]::SetEnvironmentVariable("LOG4J\_FORMAT\_MSG\_NO\_LOOKUPS", $null,[EnvironmentVariableTarget]::Machine)
```

The change will take effect after the device restarts.

**_For Linux:_**

1. Open the file /etc/environment and delete the line LOG4J\_FORMAT\_MSG\_NO\_LOOKUPS=true
2. Delete the file /etc/systemd/system.conf.d/log4j\_disable\_jndi\_lookups.conf
3. Delete the file /etc/systemd/user.conf.d/log4j\_disable\_jndi\_lookups.conf

The change will take effect after the device restarts.

**_For macOS:_**

Remove the file setenv.LOG4J\_FORMAT\_MSG\_NO\_LOOKUPS.plist from the following folders:

  - */Library/LaunchDaemons/*
  - */Library/LaunchAgents/*
  - */Users/\[username\]/Library/LaunchAgents/ - for all users*

The change will take effect after the device restarts.

### Apache Log4j security recommendations

To see active security recommendation related to Apache log4j, select the **Security recommendations** tab from the vulnerability details page. In this example, if you select **Update Apache Log4j** you'll see another flyout with more information:

:::image type="content" source="images/update_apache_log4j.png" alt-text="Update apache log4j security recommendation" lightbox="images/update_apache_log4j.png":::

Select **Request remediation** to create a remediation request.

## Explore the vulnerability in the Microsoft 365 Defender portal

Once exposed devices, files and software are found, relevant information will also be conveyed through the following experiences in the Microsoft 365 Defender portal:

### Security recommendations

Search for **CVE-2021-44228** to see security recommendations addressing the Log4Shell vulnerability:

:::image type="content" source="images/security_recommendations_log4j.png" alt-text="The log4j vulnerability on the security recommendations page" lightbox="images/security_recommendations_log4j.png":::

### Software inventory

 On the software inventory page, search for **CVE-2021-44228** to see details about the Log4j software installations and exposure:

:::image type="content" source="images/software_inventory_log4j.png" alt-text="The log4j vulnerability on the software inventory page" lightbox="images/software_inventory_log4j.png":::

### Weaknesses

On the weaknesses page, search for **CVE-2021-44228** to see information about the Log4Shell vulnerability:

:::image type="content" source="images/weaknesses_log4j.png" alt-text="The log4j vulnerability on the weaknesses page" lightbox="images/weaknesses_log4j.png":::

## Use advanced hunting

You can use the following advanced hunting query to identify vulnerabilities in installed software on devices:

 ```text
    DeviceTvmSoftwareVulnerabilities
    | where CveId in ("CVE-2021-44228", "CVE-2021-45046")
 ```

You can use the following advanced hunting query to identify vulnerabilities in installed software on devices to surface file-level findings from the disk:

 ```text
    DeviceTvmSoftwareEvidenceBeta
    | mv-expand DiskPaths
    | where DiskPaths contains "log4j"
    | project DeviceId, SoftwareName, SoftwareVendor, SoftwareVersion, DiskPaths
 ```

## Related articles

- [Threat and vulnerability management overview](http://next-gen-threat-and-vuln-mgt.md)
- [Security recommendations](tvm-security-recommendation.md)