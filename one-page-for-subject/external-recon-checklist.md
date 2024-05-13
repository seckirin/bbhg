# External Recon Checklist

## Information Gathering

> **Large scope:** Based on companies or groups.
>
> **Medium scope:** Based on a single domain.
>
> **Small scope:** Based on single Website.

**The first step:** View the [SRC](src-navigation.md) announcement.

### Based on Company

1. [Basic Information](../external-reconnaissance/based-on-company.md#basic-information)
   * [ ] Enterprise Name
   * [ ] Registered Address
   * [ ] ICP License Number
   * [ ] Phone Number
   * [ ] Email Domain
2. [Network Assets](../external-reconnaissance/based-on-company.md#network-assets)
   * [ ] ASN and CIDR
   * [ ] Root Domain
   * [ ] Mobile Application
   * [ ] WeChat Applet
3. Acquisition Information
   * [ ] Holding Offspring Company
   * [ ] Holding Ratio
4. Verify Assets Attribution
   * [ ] Return to the First Item

### Based on Domain

* [ ] Subdomain Enumeration
* [ ] Get DNS records
* [ ] Website Probing
* [ ] Extract IP Address (Exclude CDN and virtual hosting)
* [ ] Port Scanning
* [ ] Service Identification

### Based on Website

* [ ] Fingerprint Web Server
* [ ] Review Metafiles
* [ ] Enumerate Applications on Webserver
* [ ] Review comments on source code
* [ ] Identify Application Entry Points
* [ ] Map Execution Paths Through Application
* [ ] Fingerprint Web Application Framework
* [ ] Map Application Architecture
