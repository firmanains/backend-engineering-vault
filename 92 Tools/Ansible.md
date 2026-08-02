---
title: Ansible
type: tool
level: intermediate
domain: tools
status: unread
difficulty: 2
est_minutes: 10
depth: orientation
volatility: low
implements: ["[[../70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management|Immutable Infrastructure vs Configuration Management]]"]
prerequisites: ["[[../70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management|Immutable Infrastructure vs Configuration Management]]"]
next: ["[[NATS]]"]
tags: [backend, tools, infrastructure]
created: 2026-08-02
---

## What It Is, In One Paragraph

Ansible adalah tool configuration management — menjalankan skrip declarative (playbook, format YAML) lewat SSH ke banyak mesin sekaligus untuk memastikan konfigurasinya sesuai yang didefinisikan, tanpa perlu agent terpasang permanen di mesin target (berbeda dari sebagian tool sejenis seperti Puppet atau Chef).

## The Concept It Implements

Ansible mengimplementasikan sisi "configuration management" dari [[../70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management|Immutable Infrastructure vs Configuration Management]] — pendekatan mengubah server yang sudah hidup secara bertahap, kontras dengan filosofi immutable yang mengganti server sepenuhnya.

## Kapan Ini Dipakai

Peran Ansible menyusut dibanding beberapa tahun lalu, seiring container image immutable jadi pendekatan dominan untuk deployment aplikasi baru — kalau seluruh infrastruktur sudah berjalan di container/Kubernetes, kebutuhan mengonfigurasi VM secara manual jauh berkurang. Nilai Ansible yang tersisa dan tetap relevan: mengelola **fleet VM lama** yang belum (atau tidak akan) bermigrasi ke container, dan **bootstrap task** yang terjadi sebelum infrastruktur container itu sendiri bisa berjalan (menyiapkan VM dasar, memasang Docker/Kubernetes itu sendiri di node baru).

Untuk 13 aplikasi yang sebagian masih berjalan di VM dengan Yii1/Yii2, Ansible tetap punya tempat nyata mengelola konfigurasi VM itu — tapi kesadaran jujur yang perlu dipegang: ini adalah pengelolaan legacy fleet yang akan menyusut seiring migrasi ke container berlanjut, bukan arah investasi jangka panjang untuk infrastruktur baru.

## Mental Model Singkat

Playbook (YAML) mendefinisikan **task** yang dijalankan berurutan terhadap **inventory** (daftar mesin target, dikelompokkan lewat host group). Ansible terhubung lewat SSH, menjalankan modul (unit kerja: install paket, salin file, restart service) di setiap mesin target.

```yaml
# playbook.yml
- hosts: webservers
  tasks:
    - name: Pastikan nginx terpasang
      apt:
        name: nginx
        state: present
```

## Contoh Konkret

```bash
ansible-playbook -i inventory.ini playbook.yml
ansible webservers -m ping -i inventory.ini  # cek konektivitas semua host
```

## Kapan Memilih Ini vs Alternatif

Pilih Ansible untuk mengelola VM fleet legacy yang belum bermigrasi ke container, atau untuk bootstrap task yang mendahului container runtime itu sendiri. Pilih [[Terraform]] untuk menyediakan (provisioning) resource cloud baru — Terraform dan Ansible sering dipasangkan: Terraform membuat VM, Ansible mengonfigurasinya. Untuk infrastruktur baru yang bisa sepenuhnya berjalan di container, pertimbangkan langsung membangun image immutable (Docker) daripada menginvestasikan waktu besar di playbook Ansible baru.

> [!warning] Jebakan
> Menganggap Ansible cocok untuk semua kebutuhan konfigurasi infrastruktur baru — untuk sistem yang bisa berjalan sepenuhnya di container, image immutable hampir selalu pilihan yang lebih tahan terhadap configuration drift dibanding playbook yang dijalankan berulang ke server yang sama.

## Version Caveat

Dokumentasi resmi docs.ansible.com adalah sumber kebenaran untuk modul dan sintaks playbook versi yang benar-benar dipakai.

## Connected Notes

- [[../70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management|Immutable Infrastructure vs Configuration Management]] — perbandingan filosofis lengkap yang jadi konteks kenapa peran Ansible menyusut.
- [[Terraform]] — sering dipasangkan: Terraform provisioning, Ansible konfigurasi.
- [[Docker]] — alternatif immutable yang jadi arah dominan untuk deployment aplikasi baru.

## Catatan Saya

*Kosong — diisi pembaca.*
