---
type: machine
name: Ubuntu Server
os: Ubuntu 22.04 LTS
role: Жертва Linux
ip: 192.168.56.20
difficulty: Medium
tags: [lab, machine, target, linux, wazuh-agent]
---

# 🎯 Ubuntu Server

> [!info] ПАСПОРТ
> **ОС:** Ubuntu 22.04 LTS
> **Роль:** Linux-жертва + Wazuh-агент
> **IP:** 192.168.56.20
> **Интерфейс:** `enp0s3`

## ⚙️ Настройки VM
- Адаптер: Intel PRO/1000
- Wazuh-агент: устанавливается
- SSH: включён

## 🔧 Восстановление IP

```bash
sudo ip link set enp0s3 up
sudo ip addr add 192.168.56.20/24 dev enp0s3