---
type: machine
name: Metasploitable2
os: Ubuntu 8.04 (Linux, i386)
role: Уязвимая жертва
ip: 192.168.56.106
difficulty: Easy
tags: [lab, machine, target, linux]
---

# 🎯 Metasploitable2

> [!info] ПАСПОРТ
> **ОС:** Ubuntu 8.04 (2008)
> **Роль:** Уязвимая жертва
> **IP:** 192.168.56.106
> **Логин:** `msfadmin` / `msfadmin`

## ⚙️ Настройки VM
- Адаптер: PCnet-FAST III
- Загрузка: IDE (не SATA!)
- GRUB: параметр `noapic` (иначе kernel panic)

## 🔓 Уязвимости

| Порт | Сервис | Версия | CVE |
|---|---|---|---|
| 21 | FTP | vsftpd 2.3.4 | CVE-2011-2523 |
| 22 | SSH | OpenSSH 4.7p1 | — |
| 23 | Telnet | Linux telnetd | — |
| 80 | HTTP | Apache 2.2.8 | много |
| 445 | SMB | Samba 3.x | CVE-2007-2447 |
| 3306 | MySQL | 5.0.51a | — |
| 6667 | IRC | UnrealIRCd | Backdoor |

## 📝 Заметки
- **Не поддерживает** современный Wazuh-агент (слишком старая).
- Идеальна для **атак** ([[msfconsole]]).
- Детект атак — на [[Ubuntu-Server]] через Wazuh.

## 🔗 Связи
- [[IP-Table]]
- [[01-first-scan]] ← первый write-up
- [[msfconsole.md]]