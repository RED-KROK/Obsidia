---
type: writeup
target: Metasploitable2
date: 2026-09-29
severity: Low
tags:
  - writeup
  - red-team
  - recon
  - nmap
  - hydra
mitre:
  - T1046
  - T1595.001
  - T1595.002
---

> [!info] 🎯 ДОСЬЕ ОПЕРАЦИИ
> **Цель:** Metasploitable2
> **IP:** 192.168.56.106
> **Дата:** 29.09.2026
> **Оперативник:** Cor Vex (CV-7)
> **Критичность:** Low (только разведка)
> **Лаба:** [[IP-Table]]

---

## 📖 I. РАЗВЕДКА (RECON)

> [!quote] Девиз этапа
> *«Знай врага, прежде чем касаться его.»*

### 1.1. Проверка связи

```bash
ping -c 3 192.168.56.106
```

**Результат:** 3/3 пакета, 0% loss, средний RTT 4.3 мс.

> [!success] Цель жива
> Связь есть, можно сканировать.

### 1.2. Поиск хостов в подсети

```bash
sudo nmap -sn 192.168.56.0/24
```

**Найдено:**

| IP | Роль |
|---|---|
| 192.168.56.1 | Хост (шлюз VirtualBox) |
| 192.168.56.100 | DHCP VirtualBox |
| 192.168.56.106 | **Metasploitable2** (цель) |
| 192.168.56.107 | Kali (атакующий) |
| 192.168.56.109 | Wazuh (SIEM) |

### 1.3. Сканирование портов

```bash
sudo nmap -sV 192.168.56.106
sudo nmap -sV -sC -p 21,22,80,445 192.168.56.106
```

**Обнаружено:** 23 открытых порта.

---

## 📊 II. КАРТА УЯЗВИМОСТЕЙ

### Критичные порты

| Порт | Сервис | Версия | Уязвимость | Риск |
|---|---|---|---|---|
| **21/tcp** | FTP | vsftpd 2.3.4 | **CVE-2011-2523** (backdoor) | 🔴 Critical |
| **22/tcp** | SSH | OpenSSH 4.7p1 | Устаревшая версия | 🟠 High |
| **23/tcp** | Telnet | Linux telnetd | Plaintext-передача | 🔴 Critical |
| **25/tcp** | SMTP | Postfix smtpd | Возможен relay | 🟡 Medium |
| **80/tcp** | HTTP | Apache 2.2.8 | Устаревшая, много CVE | 🟠 High |
| **139/tcp** | SMB | Samba 3.x | Usermap exploit | 🔴 Critical |
| **445/tcp** | SMB | Samba 3.x | Usermap exploit | 🔴 Critical |
| **3306/tcp** | MySQL | 5.0.51a | Устаревшая | 🟠 High |
| **5432/tcp** | PostgreSQL | 8.3.0-8.3.7 | Устаревшая | 🟡 Medium |
| **6667/tcp** | IRC | UnrealIRCd | **Backdoor** | 🔴 Critical |
| **8180/tcp** | HTTP | Apache Tomcat 1.1 | Устаревшая | 🟠 High |

### Дополнительные находки (NSE-скрипты)

#### FTP (порт 21)

```
ftp-anon: Anonymous FTP login allowed (FTP code 230)
ftp-syst: vsFTPd 2.3.4 - secure, fast, stable
```

> [!danger] Анонимный FTP открыт
> Любой может залогиниться под `anonymous` без пароля.

#### SSH (порт 22)

```
ssh-hostkey:
  1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
```

#### HTTP (порт 80)

```
http-title: Metasploitable2 - Linux
http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
```

#### SMB (порт 445)

```
account_used: guest
authentication_level: user
message_signing: disabled (dangerous, but default)
smb2-time: Protocol negotiation failed
```

> [!warning] SMB с анонимным доступом
> `message_signing: disabled` — открывает возможность для **SMB relay attack**.

#### OS Discovery

```
OS: Unix (Samba 3.0.20-Debian)
Computer name: metasploitable
Domain name: localdomain
FQDN: metasploitable.localdomain
```

---

## 🧠 III. ВЫВОДЫ РАЗВЕДКИ

### Что найдено

1. **Цель крайне уязвима** — собрана специально как «тренировочная».
2. **5+ критичных бэкдоров:**
   - vsftpd 2.3.4 (порт 21)
   - UnrealIRCd (порт 6667)
   - Samba usermap (порты 139/445)
3. **Открыт анонимный FTP** — можно заливать/скачивать файлы.
4. **Telnet активен** — plaintext, легко перехватить.
5. **Wazuh пока не видит** — нет агента на Meta2 (она слишком старая).

### Приоритетные цели для следующей атаки

| Приоритет | Вектор | CVE | Инструмент |
|---|---|---|---|
| 1 | vsftpd backdoor | CVE-2011-2523 | [[msfconsole]] |
| 2 | UnrealIRCd backdoor | — | [[msfconsole]] |
| 3 | Samba usermap | CVE-2007-2447 | [[msfconsole]] |
| 4 | SSH брутфорс | — | [[hydra]] |
| 5 | Telnet брутфорс | — | [[hydra]] |

---

## 🛡️ IV. ДЕТЕКТ В WAZUH

> [!warning] Агента на Meta2 нет
> Metasploitable2 — Ubuntu 8.04 (2008). Современный Wazuh-агент туда не встанет.
> Детект будет на **Ubuntu 22.04** (следующая миссия).

### Теоретический детект (если бы агент был)

| Источник | Что искать |
|---|---|
| **Wazuh Rule 5104** | Host scan detected (level 6) |
| **Wazuh Rule 5105** | Multiple ports scanned (level 8) |
| **Suricata** | Сигнатуры scan (GPL SCAN) |
| **iptables** | Много DROP/REJECT с одного IP |

### MITRE-маппинг

- **T1046** — Network Service Discovery
- **T1595.001** — Active Scanning: Scanning IP Blocks
- **T1595.002** — Active Scanning: Vulnerability Scanning

---

## 📸 V. СКРИНШОТЫ

![[nmap-scan-full.png]]

![[nmap-nse-scripts.png]]

---

## 📚 VI. ВЫВОДЫ

### Что сработало
- `nmap -sV` быстро определил версии сервисов.
- `-sC` запустил базовые NSE-скрипты (FTP anon, SMB info).
- Все ключевые уязвимости обнаружены за 25 секунд.

### Что не сработало
- Детекта в Wazuh нет (Meta2 — слишком старая).
- Часть портов (6200, 513) показались только при полном сканировании.

### Что изучить дальше
- [ ] Подключить Wazuh-агента на Ubuntu Server
- [ ] Повторить скан → поймать алерт в Wazuh
- [ ] Изучить NSE-скрипты подробнее (`--script vuln`)
- [ ] Перейти к эксплуатации: [[msfconsole]] → vsftpd_234_backdoor

---

## 🔗 VII. СВЯЗИ

- **Инструменты:** [[nmap]], [[msfconsole]], [[hydra]]
- **Техники:** [[T1046]], [[T1595.001]], [[T1595.002]]
- **Машина:** [[Metasploitable2.md]]
- **Лаба:** [[IP-Table]]
- **Следующий write-up:** [[02-wazuh-ubuntu-agent]]

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Cor inquietum, hostis intus.*