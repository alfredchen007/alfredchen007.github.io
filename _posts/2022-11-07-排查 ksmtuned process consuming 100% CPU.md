---
layout: post

title: "排查 ksmtuned process consuming 100% CPU"

date: 2022-11-07 12:59:48 +0300

description:  

cover: 

color: rgb(222,315,333)

tags: [linux]
---

 

### 问题起因

通过 TOP 发现 KSM 相关进程使用 CPU 过高，也通过https://access.redhat.com/solutions/5706451 等已验证的SOLUTION进行了解，发现出现此情况的基本为使用 Virtualization 版本或 VM 或 hyperV 等等虚拟化相关场景。但这边出现此问题的使用场景的服务器为 x86 物理机，并不是虚拟化相关

### 问题排查

重启服务，使用 root 用户执行

```shell
systemctl restart ksmtuned.service
```

- To deactivate KSM for a single session, use the `systemctl` utility to stop `ksm` and `ksmtuned` services.

  ```none
  # systemctl stop ksm
  
  # systemctl stop ksmtuned
  ```

- To deactivate KSM persistently, use the `systemctl` utility to disable `ksm` and `ksmtuned` services.

  ```none
  # systemctl disable ksm
  Removed /etc/systemd/system/multi-user.target.wants/ksm.service.
  # systemctl disable ksmtuned
  Removed /etc/systemd/system/multi-user.target.wants/ksmtuned.service.
  ```