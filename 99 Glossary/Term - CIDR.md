---
title: Term - CIDR
type: term
level: junior
domain: foundations
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, foundations, infrastructure]
created: 2026-08-02
---

**CIDR (Classless Inter-Domain Routing)** adalah notasi menyatakan rentang alamat IP secara ringkas — misalnya `10.0.0.0/16` berarti seluruh alamat yang 16 bit pertamanya sama dengan `10.0.0.0`, mencakup 65.536 alamat sekaligus. Angka setelah garis miring (prefix length) menentukan berapa bit di depan yang tetap (menentukan "jaringan"), sisanya bebas bervariasi (menentukan alamat individual di dalam jaringan itu).

Ini kenapa istilah ini penting dipahami: notasi CIDR muncul di mana-mana dalam konfigurasi infrastruktur — mendefinisikan subnet cloud, aturan firewall/security group yang mengizinkan akses dari rentang IP tertentu, dan rentang pod/service di Kubernetes — memahami cara membaca `/24` versus `/16` (semakin kecil angkanya, semakin besar rentang alamat yang dicakup) adalah keterampilan dasar operasional.

## Muncul Di

- [[../10 Foundations/_Overview|Foundations Overview]] — relevan sebagai pengetahuan jaringan dasar yang mendasari konfigurasi infrastruktur.
- [[../70 Infrastructure and Delivery/Kubernetes Core Concepts - Pods, Deployments, Services, Ingress|Kubernetes Core Concepts - Pods, Deployments, Services, Ingress]] — rentang alamat CIDR dipakai mengonfigurasi jaringan cluster Kubernetes.

## Catatan Saya

*Kosong — diisi pembaca.*
