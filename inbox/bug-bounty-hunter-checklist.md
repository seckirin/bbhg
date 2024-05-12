# Bug Bounty Hunter Checklist

## Information Gathering

Large scale: Company, enterprise or group.

Medium scale: A single domain.

Small scale: A single website.

### Large scale

1. 公司基本信息
   * [ ] 公司名称
   * [ ] 公司地址
   * [ ] 公司联系方式
2. 公司网络资产
   * [ ] ASN 或 IP 段
   * [ ] 备案域名
   * [ ] 非备案域名
3. 公司员工信息
   * [ ] 员工姓名
   * [ ] 员工职位
   * [ ] 员工联系方式
4. 公司业务信息
   * [ ] 业务类型
   * [ ] 业务流程
   * [ ] 业务联系方式
5. 公司收购信息
   * [ ] 控股子公司
   * [ ] 投资人姓名

### Medium scale

* [ ] Subdomain Enumeration
* [ ] Get DNS records
* [ ] Website Probing
* [ ] Extract IP Address (Exclude CDN and virtual hosting)
* [ ] Port Scanning
* [ ] Service Identification

### Small Scale

* [ ] Fingerprint Web Server
* [ ] Review Metafiles
* [ ] Enumerate Applications on Webserver
* [ ] Review comments on source code
* [ ] Identify Application Entry Points
* [ ] Map Execution Paths Through Application
* [ ] Fingerprint Web Application Framework
* [ ] Map Application Architecture
