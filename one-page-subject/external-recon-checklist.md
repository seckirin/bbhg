# External Recon Checklist

## Information Gathering

> **Large scope:** Based on companies or groups.
>
> **Medium scope:** Based on a single domain.
>
> **Small scope:** Based on single Website.

**The first step:** View the [SRC](src-navigation.md) announcement.

### Based on Company

1. [Company Basic Information](../external-reconnaissance/based-on-company.md#basic-information)
   * [ ] Company Name
   * [ ] Company Address
   * [ ] Company Contact Information
   * [ ] Company Email Root Domain
2. [Company Network Assets](../external-reconnaissance/based-on-company.md#network-assets)
   * [ ] ASN or CIDR
   * [ ] ICP Registered domains
   * [ ] ICP Unregistered domains
   * [ ] Mobile Application
   * [ ] WeChat Applet
3. Company employee information
   * [ ] Employee names
   * [ ] Employee positions
   * [ ] Employee contact information
4. Company business information
   * [ ] Type of business
   * [ ] Business process
   * [ ] Business contact information
5. Company acquisition information
   * [ ] Holding subsidiaries
   * [ ] Investor names
6. Verify assets attribution
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
