---
type: tool
name: Nmap
category: Recon / Network Scanner
platform: Linux, Windows, macOS
language: C
status: studying
tags: [arsenal, recon, network, scanning, red-team, blue-team]
mitre:
  - T1046      # Network Service Discovery
  - T1595.001  # Active Scanning: Scanning IP Blocks
  - T1595.002  # Active Scanning: Vulnerability Scanning
  - T1590.004  # Gather Network Information: Network Topology
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Nmap (Network Mapper)
> **Категория:** Recon / Network Scanner
> **Платформа:** Linux, Windows, macOS
> **Назначение:** Сканирование портов, определение сервисов, ОС, поиск уязвимостей
> **Статус:** 🟢 Базовый уровень освоен
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Nmap** — золотой стандарт сетевой разведки. Что умеет:
- Обнаружение **живых хостов** в сети.
- Сканирование **открытых портов** (TCP/UDP).
- Определение **версий сервисов** и **операционной системы**.
- Запуск **NSE-скриптов** для поиска уязвимостей.
- Обход **файерволов и IDS**.

**Формат вывода:**
```
PORT     STATE     SERVICE
22/tcp   open      ssh
80/tcp   open      http
443/tcp  filtered  https
```

| Поле | Значение |
|---|---|
| `PORT` | Номер порта + протокол |
| `STATE` | `open` — открыт, `closed` — закрыт, `filtered` — фильтруется |
| `SERVICE` | Программа/служба на порту |

---

## 🚀 II. БАЗОВЫЙ СИНТАКСИС

```bash
nmap [опции] <цель>
```

### Цели

```bash
nmap target.com              # Домен
nmap 192.168.1.1             # Один IP
nmap 192.168.1.0/24          # Вся подсеть
nmap 192.168.1.1-100         # Диапазон
nmap -iL targets.txt         # Из файла
```

---

## 🎯 III. ТИПЫ СКАНИРОВАНИЯ

| Флаг | Тип | Root? | Когда использовать |
|---|---|---|---|
| `-sS` | SYN scan | ✅ | Быстро и скрытно (стандарт) |
| `-sT` | TCP Connect | ❌ | Когда нет root |
| `-sU` | UDP scan | ✅ | Поиск UDP-сервисов (медленно) |
| `-sA` | ACK scan | ✅ | Проверка фильтрации |
| `-sW` | Window scan | ✅ | Определение открытых портов |
| `-sF` | FIN scan | ✅ | Обход фаерволов |
| `-sN` | NULL scan | ✅ | Обход фаерволов |
| `-sX` | Xmas scan | ✅ | Обход фаерволов |
| `-sM` | Maimon scan | ✅ | Обход фаерволов |

> [!tip] Главное правило
> **`-sS` (SYN)** — используй по умолчанию. **`-sT`** — если нет root. **`-sU`** — только для конкретных портов (53, 161).

---

## 🎯 IV. СКАНИРОВАНИЕ ПОРТОВ

```bash
nmap -p 80,443,8080 target.com              # Конкретные порты
nmap -p 1-1000 target.com                   # Диапазон
nmap -p- target.com                         # Все 65535 портов (долго!)
nmap -F target.com                          # Топ-100 портов
nmap --top-ports 200 target.com             # Топ-N портов
nmap -p http,https target.com               # По имени службы
nmap -p U:53,111,137,T:21-25,80 target.com  # TCP и UDP вместе
```

### Тайминги (`-T`)

| Флаг | Скорость | Стелс |
|---|---|---|
| `-T0` | Очень медленно | Максимальный |
| `-T1` | Медленно | Высокий |
| `-T3` | Норма (по умолч.) | Средний |
| `-T4` | Быстро | Низкий |
| `-T5` | Очень быстро | Почти нет |

> [!tip] Правило Легиона
> `-T4` — рабочий вариант. `-T5` — только по своей лабе. На реальных целях — `-T2` или `-T3`.

---

## 🎯 V. ОБНАРУЖЕНИЕ ХОСТОВ

```bash
nmap -sn 192.168.1.0/24              # Ping scan (без портов)
nmap -Pn target.com                  # Пропустить обнаружение
nmap -PS80,443 target.com            # TCP SYN ping
nmap -PA80,443 target.com            # TCP ACK ping
nmap -PU53 target.com                # UDP ping
nmap -PE target.com                  # ICMP echo ping
nmap -sn --traceroute target.com     # Discovery + traceroute
```

> [!info] Разница
> **`-sn`** — только обнаружение (без портов). **`-Pn`** — считать хост живым даже если не отвечает на ping (важно для Windows с брандмауэром).

---

## 🎯 VI. ОПРЕДЕЛЕНИЕ ВЕРСИЙ И ОС

```bash
nmap -sV target.com              # Версии служб
sudo nmap -O target.com          # Определение ОС (root)
nmap -A target.com               # Агрессивно: ОС + версии + скрипты + traceroute
nmap -sC target.com              # Скрипты по умолчанию
sudo nmap -A -T4 target.com      # Агрессивно + быстро
nmap -sV -sC target.com          # Версии + скрипты
```

> [!tip] Комбинация Легиона
> `nmap -sV -sC -O -T4 -p- <target>` — полное сканирование одного хоста.
> Займёт 10–30 минут, но даст максимум информации.

---

## 🎯 VII. NSE-СКРИПТЫ

NSE = Nmap Scripting Engine. Расширяет возможности Nmap.

### Категории

```bash
nmap --script safe target.com           # Безопасные
nmap --script default target.com        # По умолчанию
nmap --script vuln target.com           # Поиск уязвимостей
nmap --script exploit target.com        # Эксплойты
nmap --script auth target.com           # Авторизация
nmap --script discovery target.com      # Обнаружение
```

### Конкретные скрипты

```bash
nmap --script http-title target.com                # Заголовок HTTP
nmap --script ssh-brute target.com                 # SSH брутфорс
nmap --script ftp-anon target.com                  # Анонимный FTP
nmap --script ssl-enum-ciphers target.com          # SSL-шифры
nmap --script smb-os-discovery target.com          # SMB-обнаружение
```

### Параметры скриптов

```bash
nmap --script http-title --script-args http.useragent="Custom Agent" target.com
nmap --script ssh-brute --script-args userdb=users.txt,passdb=passwords.txt target.com
```

---

## 🎯 VIII. ВЫВОД И ФАЙЛЫ

```bash
nmap -oN scan.txt target.com          # Обычный текст
nmap -oX scan.xml target.com          # XML
nmap -oG scan.gnmap target.com        # Grep-формат
nmap -oA scan target.com              # Все форматы сразу
nmap -iL hosts.txt                    # Цели из файла
nmap --resume previous_scan.xml       # Продолжить
```

---

## 🎯 IX. ОБХОД ЗАЩИТЫ

```bash
sudo nmap -f target.com                              # Фрагментация пакетов
sudo nmap -mtu 24 target.com                         # Свой MTU
sudo nmap -D 192.168.1.100,192.168.1.200 target.com  # Поддельные IP (decoy)
sudo nmap --source-port 53 target.com                # Фиксированный src-порт
sudo nmap --data-length 200 target.com               # Случайные данные
sudo nmap --spoof-mac 00:11:22:33:44:55 target.com   # Подмена MAC
sudo nmap --spoof-mac 0 target.com                   # Случайный MAC
```

---

## 🎯 X. TRACEROUTE И АНАЛИЗ

```bash
nmap --traceroute target.com          # Трассировка маршрута
nmap --packet-trace target.com        # Отслеживание пакетов
nmap --reason target.com              # Причины состояний портов
```

---

## 🎯 XI. СЕТЕВАЯ НАСТРОЙКА

```bash
nmap -e eth0 target.com              # Указать интерфейс
nmap -S 192.168.1.50 target.com      # Исходный IP
nmap -r target.com                   # Последовательный обход портов
```

---

## 🎯 XII. ПОЛЕЗНЫЕ КОМБИНАЦИИ

```bash
# Быстрый осмотр сети
nmap -T4 -F 192.168.1.0/24

# Полная разведка одного хоста
nmap -sV -sC -O -T4 target.com

# Полное сканирование (все порты)
sudo nmap -A -T4 -p- target.com

# Безопасное сканирование
nmap --script safe -sV target.com

# Стелс-сканирование
sudo nmap -sS -T2 -f --data-length 50 target.com
```

---

## 🍯 XIII. ВКУСНЫЕ ПОРТЫ ДЛЯ АТАК

| Порт | Сервис | Популярные атаки |
|---|---|---|
| `21` | FTP | Анонимный вход, брутфорс |
| `22` | SSH | Брутфорс, старые версии |
| `23` | Telnet | Plaintext-пароли |
| `25` | SMTP | Relay, спам |
| `80` | HTTP | SQLi, XSS, LFI/RFI |
| `443` | HTTPS | TLS-уязвимости, веб-уязвимости |
| `445` | SMB | EternalBlue, pass-the-hash |
| `3306` | MySQL | SQLi, доступ без пароля |
| `3389` | RDP | BlueKeep, брутфорс |
| `6379` | Redis | RCE при мисконфиге |
| `27017` | MongoDB | Доступ без пароля |

> [!danger] Золотое правило пентеста
> Открыл порт → определи сервис → определи версию → ищи CVE → эксплуатируй.

---

## 🛡️ XIV. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1046** — Network Service Discovery
> - **T1595.001** — Scanning IP Blocks
> - **T1595.002** — Vulnerability Scanning

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Wazuh Rule 5104** | Host scan detected (level 6) |
| **Wazuh Rule 5105** | Multiple ports scanned (level 8) |
| **Wazuh Rule 5106** | Port scan detected (level 10) |
| **Suricata** | Сигнатуры scan (GPL SCAN) |
| **Zeek** | Scan detection |
| **Firewall logs** | Много отклонённых соединений с одного IP |
| **iptables / ufw** | Массовые DROP/REJECT |

### Признаки Nmap в логах

```
Sep 29 10:23:45 ubuntu kernel: [12345] iptables: DROP IN=eth0 SRC=192.168.56.107 DST=192.168.56.20 PROTO=TCP DPT=22
Sep 29 10:23:45 ubuntu kernel: [12346] iptables: DROP IN=eth0 SRC=192.168.56.107 DST=192.168.56.20 PROTO=TCP DPT=23
Sep 29 10:23:45 ubuntu kernel: [12347] iptables: DROP IN=eth0 SRC=192.168.56.107 DST=192.168.56.20 PROTO=TCP DPT=25
```

> [!danger] Красные флаги
> - **Много соединений** к разным портам с одного IP за секунды.
> - **Сканирование диапазона** портов (1–1000, 1–65535).
> - **Син-пакеты** без завершения handshake.
> - **Попытки** к «мёртвым» портам (23, 137, 139).
> - **Регулярные интервалы** — автоматизированное сканирование.

### Практика в лабе

1. С Kali:
   ```bash
   nmap -sS -T4 192.168.56.20
   nmap -sV 192.168.56.20
   nmap -p- 192.168.56.20
   ```
2. В Wazuh → **Security events** → фильтр `rule.id:5104` или `5105`.
3. Разбери алерт: Source IP, Target, Rule, MITRE.

> [!info] Как защититься
> - **Firewall** (iptables/ufw) — закрыть все ненужные порты.
> - **Fail2ban** — бан IP после N отклонённых соединений.
> - **Port knocking** — порты открываются только после правильной последовательности.
> - **IDS/IPS** (Suricata) — обнаружение сканирований.
> - **Honeypots** — ловушки для сканеров (T-Pot, Cowrie).

---

## 🩺 XV. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `nmap: command not found` | `sudo apt install nmap` |
| Сканирование занимает часы | Убери `-p-`, используй `-F` или `--top-ports 1000` |
| `-O` не работает | Нужен root: `sudo nmap -O` |
| Хост кажется мёртвым | Используй `-Pn` (Windows блокирует ping) |
| Медленно | `-T4`, но не на реальных целях |
| Не видит UDP-сервисы | `sudo nmap -sU -p 53,161` (только конкретные) |
| Файрвол блокирует | `-f`, `-D`, `--source-port 53` |

### Полезные проверки

```bash
nmap --version
nmap -h
nmap --iflist                  # Список интерфейсов
```

---

## 🔗 XVI. СВЯЗИ

- **Инструменты:** [[masscan]], [[rustscan]], [[httpx]], [[nikto]], [[msfconsole]], [[hydra]], [[ffuf]]
- **Техники:** [[T1046]], [[T1595.001]], [[T1595.002]], [[T1590.004]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]], [[Windows 10]]
- **Защита:** [[Wazuh]], [[Suricata]], [[Fail2ban]]

---

## 📚 XVII. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] `masscan` — сверхбыстрый сканер.
- [ ] `rustscan` — современная альтернатива.
- [ ] NSE-скрипты: написание собственных.
- [ ] Pivoting через Nmap (`--proxies`).
- [ ] Интеграция с Metasploit (`db_nmap`).
- [ ] Детект сканирований в Suricata.

---

## 📝 XVIII. ШПАРГАЛКА

```bash
# Обнаружение хостов
nmap -sn 192.168.1.0/24

# Быстрое сканирование
nmap -F -T4 target.com

# Полное сканирование портов
nmap -p- -T4 target.com

# Версии + скрипты + ОС
sudo nmap -sV -sC -O -T4 target.com

# Агрессивно (всё сразу)
sudo nmap -A -T4 target.com

# Сканирование из файла
nmap -iL targets.txt

# Сохранить во все форматы
nmap -oA scan target.com

# Поиск уязвимостей
nmap --script vuln target.com

# Стелс-режим
sudo nmap -sS -T2 -f --data-length 50 target.com

# Обход фаервола
sudo nmap -D 10.0.0.1,10.0.0.2 target.com
sudo nmap --source-port 53 target.com
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Nmap — это глаза Легиона. Прежде чем атаковать — увидь. Прежде чем увидеть — просканируй.*