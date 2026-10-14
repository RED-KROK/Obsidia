---
type: lab
tags:
  - lab
  - network
  - ip
  - hydra
created: 2026-09-29
---
# 🗺️ IP-Table: Карта лаборатории

> [!info] Сеть Host-only
> **Подсеть:** `192.168.56.0/24`
> **Адаптер:** `VirtualBox Host-Only Ethernet Adapter`

---

## 📊 Машины

| Машина | IP | Роль | Логин | Пароль |
|---|---|---|---|---|
| **Хост (шлюз)** | 192.168.56.1 | VirtualBox | — | — |
| **DHCP VirtualBox** | 192.168.56.100 | Служебный | — | — |
| **Wazuh** | 192.168.56.109 | SIEM | `admin` | `admin` |
| **Ubuntu Server** | 192.168.56.20 | Жертва Linux | `user` | ? |
| **Windows 10** | 192.168.56.30 | Жертва Windows | `Admin` | ? |
| **Kali** | 192.168.56.107 | Атакующий | `kali` | `kali` |
| **Metasploitable2** | 192.168.56.106 | Уязвимая жертва | `msfadmin` | `msfadmin` |

> [!warning] IP могут меняться
> Kali, Wazuh и Meta2 получают IP по DHCP. Если слетели — проверь `ip a` (Linux) и обнови таблицу.

---

## 🔧 Как восстановить IP вручную

Если IP слетел после перезагрузки:

### Ubuntu / Wazuh / Kali
```bash
sudo ip link set eth0 up
sudo ip addr add 192.168.56.XX/24 dev eth0
```

### Metasploitable2
```bash
sudo ifconfig eth0 192.168.56.106 netmask 255.255.255.0 up
```

### Windows
Панель управления → Сеть → Ethernet → IPv4 → Свойства:
- IP: `192.168.56.30`
- Маска: `255.255.255.0`

---

## 🌐 Проверка связи

```bash
# Быстрая проверка всех хостов
sudo nmap -sn 192.168.56.0/24

# Проверка конкретной цели
ping -c 3 192.168.56.106
```

---

## 🔗 Связи

- [[00-План Обучения/База|База]]
- [[Metasploitable2.md]]
- [[Ubuntu-Server]]
- [[Windows10]]
- [[Wazuh]]

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*