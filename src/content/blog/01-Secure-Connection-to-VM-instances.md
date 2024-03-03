---
author: Phawin
pubDatetime: 2024-03-03T03:55:00Z
title: SSH เข้าเครื่อง VM บน GCP อย่างปลอดภัย ด้วย Identity-Aware Proxy
slug: securely-ssh-into-vm-on-gcp
ogImage: ../../assets/images/01/feature.png
featured: true
draft: false
tags:
  - gcp
  - ops
  - security
description: ในยุคสมัยนี้ใครๆก็ใช้งาน VM บน Cloud การ SSH เข้าไปในนั้นเพื่อทำงานต่างๆจึงเป็นสิ่งที่หลีกเลี่ยงไม่ได้ บทความนี้นำเสนอวิธีเพิ่มความปลอดภัยในการใช้งาน VM Instance ของคุณ
---

![Feature](@assets/images/01/feature.png)

## Table of contents

## Introduction

ในยุคสมัยนี้ใครๆก็ใช้งาน VM บน Cloud การ SSH เข้าไปในนั้นเพื่อทำงานต่างๆจึงเป็นสิ่งที่หลีกเลี่ยงไม่ได้ โดยทั่วไปที่นิยมทำกันก็มักจะ Allow ให้เครื่อง VM สามารถเข้าผ่าน Internet ได้โดยตรง โดยบางทีก็มีการเปิด Access Port 22 ให้ทั้งโลก (0.0.0.0/0)

บางคนอยากให้ปลอดภัยหน่อยก็อาจจะจำกัดให้เข้าได้เฉพาะ IP ตัวเองเท่านั้น (ipaddress/24) แต่วีธีที่จำกัดให้บาง IP เข้าเท่านั้นก็มีข้อเสียคือ IP Address ที่เราใช้งานกันอยู่ที่บ้าน/ที่ทำงาน มีการเปลี่ยนอยู่แทบจะทุกวัน (ยกเว้น fixed ip internet) เราจึงจำเป็นต้องมีการเปลี่ยน Allow IP อยู่ทุกๆวัน

ถ้าเราไม่อยากให้เครื่อง VM เราสามารถเข้าได้จาก Internet เลยเพื่อลด Attack Surface ให้ได้มากที่สุด บน Google Cloud Platform สามารถทำได้ 5 วิธีดังนี้

1. [Other VMs on the network](https://cloud.google.com/solutions/connecting-securely#bastion)
2. Identity-Aware Proxy's TCP forwarding <-- วันนี้เราจะมาใช้วิธีนี้กัน
3. [The metadata server](https://cloud.google.com/firewall/docs/firewalls#gcp-metadata-server)
4. Google Cloud SDK
5. [Managed VPN gateway](https://cloud.google.com/solutions/connecting-securely#vpn)

## Identity-Aware Proxy

### อะไรคือ Identity-Aware Proxy

Identity-Aware Proxy เป็นเครื่องมือที่ช่วย Control Access ของ Application หรือ, Resource ต่างๆบน Cloud หรือ On Premise ผ่าน Identity บน Google Cloud IAM ได้โดยละเอียด ตามตัวอย่างดังภาพ

![VM Instances](@assets/images/01/secure-vm.png)
![Application](@assets/images/01/secure-application.png)

### ทำยังไง

1. สร้าง VM Instance ที่อยากใช้ขึ้นมาโดยไม่ต้อง Add External IPv4 เพื่อปิด Internet Access
   ![Disable External IPv4](@assets/images/01/disable-external-ipv4.png)
2. สร้าง Firewall Rule โดยให้อยู่ใน Network เดียวกันกับ VM Instance ที่สร้างขึ้นมาในข้อ 1)
   a. Allow Source IP: `35.235.240.0/20`
   b. Protocol: `tcp:22`
   c. Target -> Target Tags: `allow-internal-ssh`
   ![Create Firewall Rule](@assets/images/01/firewall-rule.png)
3. Edit Instance ข้อ 1) ใส่ Network Tag ชื่อ `allow-internal-ssh`
   ![Attach Network Tags](@assets/images/01/network-tags.png)

4. ทดลอง SSH เข้า Instance

   - 4.1. ผ่าน Column SSH ในหน้า (VM Instances)[https://console.cloud.google.com/compute/instances?hl=en]
   - 4.2. Gcloud Command
     `gcloud compute ssh <VM_NAME> --tunnel-through-iap`

5. เสร็จเรียบร้อย เราจะ SSH เข้าไปใน Instance ด้วย Indentity Google Cloud ที่ Login อยู่
   ![Logged into Instance](@assets/images/01/done.png)

## Summary

บทความนี้แนะนำวิธีการเข้าถึง VM บน Google Cloud Platform อย่างปลอดภัยโดยไม่ต้องเปิด Port 22 จาก Internet โดยใช้งานผ่าน Identity-Aware Proxy (IAP) แทนเพื่อช่วยเพิ่มความปลอดภัยและควบคุมการเข้าถึงผ่าน Identity บน Google Cloud IAM ได้อย่างละเอียด
