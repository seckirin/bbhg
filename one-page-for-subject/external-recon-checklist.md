# External Recon Checklist

* **Large scope:** Based on companies or groups.
* **Medium scope:** Based on a single domain.
* **Small scope:** Based on single Website.

**The first step:** View the [SRC](src-navigation.md) announcement.

### Company-based Recon

* [ ] [Basic Information](../external-recon/company-based-recon/#basic-information)
  * Enterprise Name
  * Registered Address
  * Phone Number
  * Email Domain
  * ICP License Number
* [ ] [Network Assets Enumeration](../external-recon/company-based-recon/network-assets-enumeration.md)
  * ASN / CIDR
  * Domain Enumeration
  * Mobile Application
  * WeChat Applet
* [ ] [Acquisition Information](../external-recon/company-based-recon/#acquisition-information)
  * Controlling Subsidiaries
  * Shareholding Ratio
* [ ] Assets Ownership Verification

### Domain-based Recon

* [ ] Domain Basics Information
* [ ] Subdomain Enumeration
* [ ] [Get DNS Record](../external-recon/domain-based-recon/#get-dns-record)
* [ ] Extract IP Address (Exclude CDN and virtual hosting)
* [ ] Generate CIDR
* [ ] Website Probing
* [ ] Port Scanning
* [ ] Service Identification

### Website-based Recon

* [ ] Fingerprint Web Server
* [ ] Review Metafiles
* [ ] Enumerate Applications on Webserver
* [ ] Review comments on source code
* [ ] Identify Application Entry Points
* [ ] Map Execution Paths Through Application
* [ ] Fingerprint Web Application Framework
* [ ] Map Application Architecture
