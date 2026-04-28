---
title: "Shortcodes Test"
description: "Sanity-check page for the imported Blowfish shortcodes after the Hextra hx: prefix migration."
---

{{< lead >}}
A quick page to verify imported Blowfish shortcodes render correctly under Hextra's prefixed Tailwind.
{{< /lead >}}

## Accordion

{{< accordion mode="collapse" separated=true >}}
  {{< accordionItem title="Summary" icon="expand" open=true >}}
This page documents every certification and competitive achievement I have earned as a Cloud Security Engineer - from foundational security fundamentals through to advanced offensive security and cloud-native architecture.

My certification stack: **OSCP/OSCP+** (Offensive Security Certified Professional - first attempt, 100/100), **AWS Certified Security Specialty (SCS-C02)**, **AWS Certified Solutions Architect Associate (SAA-C03)**, **CompTIA CySA+**, **CompTIA Security+**, and **eJPT**. Together these span offensive security, cloud security architecture, threat detection, and security operations.

My competitive rankings validate these credentials against a global field: **4th Worldwide in the Amazon AppSec CTF 2025** (HackTheBox), **18th Worldwide in the Wiz Cloud Security Championship** (600+ participants), **878th Worldwide on HackTheBox**, **100th Worldwide on CTFTime**, **780th on YesWeHack**, and **Top 10% Worldwide on TryHackMe**.


  {{< /accordionItem >}}
{{< /accordion >}}

## Badge

{{< badge >}}
First-attempt pass
{{< /badge >}}

## Alert

{{< alert "circle-info" >}}
This site was migrated from Blowfish to Hextra. All shortcodes were ported to use Hextra's `hx:` Tailwind prefix.
{{< /alert >}}

## Lead paragraph

{{< lead >}}
Lead text appears slightly larger and lighter to introduce a section.
{{< /lead >}}
