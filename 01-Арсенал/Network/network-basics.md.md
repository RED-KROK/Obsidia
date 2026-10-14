---
type: tool
name: Network Basics (ifconfig, ip)
category: Network / Diagnostics
platform: Linux
status: mastered
tags: [arsenal, network, diagnostics, linux, blue-team, red-team]
mitre:
  - T1016      # System Network Configuration Discovery
  - T1016.001  # Internet Connection Discovery
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** ifconfig, ip
> **Категория:** Network / Diagnostics
> **Платформа:** Linux
> **Назначение:** Просмотр и управление сетевыми интерфейсами
> **Статус:** 🟢 Освоено
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**ifconfig** — старая утилита из `net-tools`. Показывает и управляет сетевыми интерфейсами. **Устаревает**, но всё ещё встречается.

**ip** — современная замена (из `iproute2`). Мощнее, поддерживает IPv6, namespace, маршруты.

> [!tip] Правило Легиона
> Изучай **ip**, но знай **ifconfig**. В старых системах (Metasploitable 2) — только ifconfig.

---

## 🚀 II. IFCOFIG — ПРОСМОТР

```bash
ifconfig                  # Все активные интерфейсы
ifconfig eth0             # Конкретный интерфейс
ifconfig -a               # ВСЕ интерфейсы (включая выключенные)
```

**Пример вывода:**
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 00:1a:2b:3c:4d:5e  txqueuelen 1000  (Ethernet)
```

| Поле | Значение |
|---|---|
| `UP` | Интерфейс активен |
| `inet` | IPv4-адрес |
| `ether` | MAC-адрес |
| `mtu` | Максимальный размер пакета |

---

## 🚀 III. IP — ОСНОВНЫЕ ПОДКОМАНДЫ

| Команда | Что делает |
|---|---|
| `ip a` / `ip addr` | Управление адресами |
| `ip link` | Управление интерфейсами |
| `ip route` | Управление маршрутами |

### Просмотр адресов

```bash
ip a                    # Краткая форма (аналог ifconfig)
ip addr show            # Полная форма
ip addr show eth0       # Только для eth0
```

**Пример вывода:**
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
    inet6 fe80::21a:2bff:fe3c:4d5e/64 scope link
```

| Поле | Значение |
|---|---|
| `state UP` | Интерфейс активен |
| `inet .../24` | IPv4 с маской |
| `inet6` | IPv6-адрес |
| `link/ether` | MAC-адрес |

### Управление интерфейсами

```bash
ip link set eth0 up             # Включить
ip link set eth0 down           # Выключить
ip link show                    # Показать все
```

### Назначить IP

```bash
sudo ip addr add 192.168.56.40/24 dev eth0
```

> [!warning] До перезагрузки
> `ip addr add` работает **до перезагрузки**. Чтобы сохранить — правь конфиги (`/etc/netplan/` в Ubuntu, `/etc/network/interfaces` в Debian/Kali).

---

## 🎯 IV. СРАВНЕНИЕ ifconfig И ip

| Функция | ifconfig | ip |
|---|---|---|
| Показать интерфейсы | `ifconfig` | `ip a` / `ip link` |
| Вкл/выкл | `ifconfig eth0 up/down` | `ip link set eth0 up/down` |
| Назначить IP | `ifconfig eth0 192.168.1.100` | `ip addr add 192.168.1.100/24 dev eth0` |
| IPv6 | ❌ | ✅ |
| Маршруты | ❌ (нужен `route`) | `ip route` |

> [!tip] Совет
> Всегда используй `ip` в новых системах. `ifconfig` — только для legacy (Metasploitable 2).

---

## 🛡️ V. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1016** — System Network Configuration Discovery

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Auditd / Sysmon** | Запуск `ifconfig` или `ip a` на сервере |
| **Wazuh Agent** | Необычные команды в `command` |
| **Bash history** | `ifconfig`, `ip a`, `ip route` в non-admin сессиях |

> [!info] На сервере
> `ifconfig` и `ip a` на **сервере** — необычно. На рабочей станции — норма. Мониторь **аномалии**.

---

## 🩺 VI. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `ifconfig: command not found` | `sudo apt install net-tools` |
| `ip: command not found` | `sudo apt install iproute2` |
| Интерфейс не виден | `ip link show` (включая выключенные) |
| Нет IP | `sudo ip addr add ...` или `dhclient` |

---

## 🔗 VII. СВЯЗИ

- **Инструменты:** [[nmap]], [[netstat]], [[ss]], [[tcpdump]]
- **Техники:** [[T1016]]
- **Лаба:** [[IP-Table]]
- **Защита:** [[Wazuh]]

---

## 📝 VIII. ШПАРГАЛКА

```bash
# ifconfig
ifconfig                  # все активные
ifconfig -a               # все
ifconfig eth0             # конкретный

# ip
ip a                      # адреса
ip link show              # интерфейсы
ip route                  # маршруты
ip link set eth0 up       # включить
ip addr add 192.168.1.100/24 dev eth0  # назначить IP
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Прежде чем сканировать чужие сети — знай свою. ifconfig и ip — это глаза Легиона.*