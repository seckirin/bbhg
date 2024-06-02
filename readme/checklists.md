# Checklists

[BBGH](../) > [**Checklists**](checklists.md)

## Reconnaissance

在对目标进行侦查之前，我们首先要做的是阅读[安全响应中心 (SRC)](../awesome-bugbounty/security-response-center.md) 通告。

### Company

1. 基本信息
2. 网络资产
3. 收购信息

* [ ] [Basic Information](../reconnaissance/company-based/#basic-information)
  * Enterprise Name
  * Registered Address
  * Phone Number
  * Email Domain
  * ICP License Number
* [ ] [Network Assets Enumeration](../reconnaissance/company-based/internet-assets-enumeration.md)
  * ASN / CIDR
  * Domain Enumeration
  * Mobile Application
  * WeChat Applet
* [ ] [Acquisition Information](../reconnaissance/company-based/#acquisition-information)
  * Controlling Subsidiaries
  * Shareholding Ratio

### 1.2 Domain

* [ ] [Domain Basics Information](../reconnaissance/domain-based/#domain-basics-information)
* [ ] [Subdomain Enumeration](../reconnaissance/domain-based/subdomain-enumeration.md)
* [ ] [Get DNS Record](../reconnaissance/domain-based/#get-dns-record)
* [ ] [Extract IP Address](../reconnaissance/domain-based/#extract-ip-address)
* [ ] [Port Scanning](../reconnaissance/domain-based/#port-scanning)
* [ ] [Website Probing](../reconnaissance/domain-based/#website-probing)
* [ ] [Service Identification](../reconnaissance/domain-based/#service-identification)

### 1.3 Website

* [ ] Fingerprint Web Server
* [ ] Review Metafiles
* [ ] Enumerate Applications on Webserver
* [ ] Review comments on source code
* [ ] Identify Application Entry Points
* [ ] Map Execution Paths Through Application
* [ ] Fingerprint Web Application Framework
* [ ] Map Application Architecture
