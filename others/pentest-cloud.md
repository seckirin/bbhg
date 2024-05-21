# Cloud Service Fingerprint

## 云服务特征

```bash
# AWS (Amazon Web Services)
# AK 通常以 AKIA 开头。20 个随机的大写字母和数字组成的字符，例如 AKHDNAPO86BSHKDIRYTE
^AKIA[A-Za-z0-9]{16}$
# SK: 40 个随机的大小写字母组成的字符，例如 S836fh/J73yHSb64Ag3Rkdi/jaD6sPl6/antFtU（无法找回丢失的 Secret Access Key ID）。

# 阿里云 (Alibaba Cloud)
# AK 通常以 LTAI 开头。长度为16-24个字符，由大写字母和数字组成。
^LTAI[A-Za-z0-9]{12,20}$
# Access Key Secret 长度为30个字符，由大写字母、小写字母和数字组成。

# 腾讯云 (Tencent Cloud)
# AK 通常以 AKID 开头。长度为17个字符，由字母和数字组成。
^AKID[A-Za-z0-9]{13,20}$
# SK 长度为40个字符，由字母和数字组成。
```

