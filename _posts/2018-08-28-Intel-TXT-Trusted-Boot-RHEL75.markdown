---
layout: post
title: Intel TXT on RHEL 7.5
---

Support for TPM 2.0 is now available in RHEL 7.5.

This post covers a basic set up of Intel TXT on Red Hat enterprise linux 7.5.

If you're interested a bit more about TPM 2.0 tools and running a simulator, I suggest you jump over to my [earlier post](http://lukehinds.com/2018/08/28/TPM2-On-RHEL75.html) on the subject.

First off, you need of course a TPM 2.0 chip installed on your target device. There is no way we can cover all vendor approaches to this in a small post such as this, so please consult the relevant documentation.

Once the TPM 2.0 and Intel TXT is enabled in your bios, confirm the chip
is picked up by the kernel:

```
# dmesg | grep -i tpm                                          
[0.000000] ACPI: SSDT 000000007b6b9000 003A7 (v02 DELL   Tpm2Tabl 00001000 INTL 20121114)
[0.000000] ACPI: TPM2 000000007b6b8000 00038 (v04 DELL   EDK2     00000002      01000013)
[1.197420] tpm_tis MSFT0101:00: 2.0 TPM (device-id 0xFE, rev-id 2)
```

Next we will need to install the `tboot` (Trusted Boot Package)

`# yum install tboot`

One nice thing about the `tboot` package, not only will it load `/boot/tboot.gz` for you, it will also configure grub2 to provide a tboot option:

![tboot](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/tboot.png)

Now select the `tboot` option to boot into the newly configured grub2 entry.

Once you server is up, we use the `txt-stat` command to query the result:

```
[root@nfvsdn-04 ~]# txt-stat | grep measured
TXT measured launch: TRUE
TBOOT: measured launch succeeded
```

Pretty simple huh? As it should be!
