---
type: writeup
target: Ubuntu Server
date: 2026-10-01
severity: High
tags: [writeup, red-team, blue-team, purple-team, hydra, wazuh, ssh]
mitre:
  - T1110.001
  - T1110.003
  - T1021.004
---

> [!info] 🎯 ДОСЬЕ ОПЕРАЦИИ
> **Цель:** Ubuntu Server (`192.168.56.20`)
> **Дата:** 01.10.2026
> **Оперативник:** Cor Vex (CV-7)
> **Критичность:** High
> **Тип:** Red → Blue (purple team)
> **Лаба:** [[IP-Table]]

---

## 📖 I. КОНТЕКСТ

> [!quote] Девиз этапа
> *«Атакуй, чтобы понять, как защищаться. Защищайся, чтобы понять, как атакуют.»*

**Первый полный цикл purple team.** Атака с [[Kali]] на [[Ubuntu-Server]], детект в [[Wazuh]] через агента.

**Схема:**
```
Kali (.107)  ──[SSH brute force]──▶  Ubuntu (.20)
                                        │
                                        ▼
                                   Wazuh Agent
                                        │
                                        ▼
                                   Wazuh SIEM (.109)
```

---

## 🧠 II. АТАКА (RED TEAM)

### 2.1. Инструмент
- **[[hydra]]** — онлайн-брутфорс.
- **Словарь:** `/usr/share/wordlists/rockyou.txt` (или кастомный).

### 2.2. Команда
```bash
sudo hydra -l user -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20 -t 4
```

**Параметры:**
- `-l user` — логин-цель.
- `-P` — файл с паролями.
- `-t 4` — 4 потока (тише, чтобы не забанили).
- `ssh://` — протокол.

### 2.3. Результат
- Hydra за **15 секунд** сгенерировала **множество** неудачных попыток входа.
- Пароль **не подобран** (цель — детект, не взлом).
- `Ctrl+C` — остановка.

### 2.4. Следы в логах Ubuntu
```bash
sudo grep "Failed password" /var/log/auth.log | tail -20
```

**Вывод:**
```
Oct  1 18:55:12 ubuntu-server sshd[2345]: Failed password for user from 192.168.56.107 port 54321 ssh2
Oct  1 18:55:12 ubuntu-server sshd[2346]: Failed password for user from 192.168.56.107 port 54322 ssh2
Oct  1 18:55:12 ubuntu-server sshd[2347]: Failed password for user from 192.168.56.107 port 54323 ssh2
...
```

**Источник:** `192.168.56.107` — **Kali**.

---

## 🛡️ III. ДЕТЕКТ (BLUE TEAM)

### 3.1. Правило Wazuh
| Параметр | Значение |
|---|---|
| **Rule ID** | `5710` |
| **Level** | `10` (High) |
| **Description** | `sshd: brute force trying to get access to the system` |
| **Группа правил** | `sshd, authentication_failed` |

### 3.2. Ключевые поля события
| Поле | Значение |
|---|---|
| `@timestamp` | 2026-10-01T18:55:xx |
| `agent.name` | `Ubuntu-server` |
| `agent.ip` | `192.168.56.20` |
| **`data.srcip`** | **`192.168.56.107`** ← атакующий |
| `data.srcuser` | `user` |
| `data.dstuser` | `user` |
| `rule.mitre.id` | `T1110` |
| `rule.mitre.tactic` | `Credential Access` |
| `rule.mitre.technique` | `Brute Force` |
| `full_log` | полная строка из `/var/log/auth.log` |

### 3.3. Как нашёл в панели

1. **Threat Hunting → Events**
2. Фильтр: `rule.id:(5710 or 5712)`
3. Сортировка по времени — свежие сверху.
4. Клик по строке → детали с `data.srcip`.

### 3.4. Цепочка детекта

```
SSH-атака (Kali)
   │
   ▼
/var/log/auth.log (Ubuntu)
   │
   ▼
Wazuh Agent (чтение лога)
   │
   ▼
Wazuh Manager (правило 5710)
   │
   ▼
Wazuh Indexer (сохранение)
   │
   ▼
Dashboard (алерт в UI)
```

---

## 🧩 IV. MITRE ATT&CK

| Техника | Название | Тактика |
|---|---|---|
| **T1110** | Brute Force | Credential Access |
| **T1110.001** | Password Guessing | Credential Access |
| **T1110.003** | Password Spraying | Credential Access |
| **T1021.004** | SSH | Lateral Movement |

---

## 📸 V. СКРИНШОТЫ

![[hydra-attack.png]]

![[wazuh-alert-5710.png]]

![[auth-log-failed.png]]

---

## 📚 VI. ВЫВОДЫ

### Что сработало
- **Агент Wazuh** мгновенно прочитал `/var/log/auth.log`.
- **Правило 5710** сработало на серию неудачных входов.
- **Панель** показала алерт за **секунды**.
- **`data.srcip`** чётко указал источник — `192.168.56.107`.

### Что можно улучшить
- **`netstat` отсутствует** на Ubuntu — агент не проверяет порты. Установить: `sudo apt install net-tools`.
- **nmap-скан не дал алертов** — потому что нет iptables-логов и IDS. Настроить логирование iptables или поставить [[Suricata]].
- **Словарь rockyou** — 140 МБ. Для быстрых тестов создать кастомный из 10 строк.

### Что изучить дальше
- [ ] Настроить iptables-логирование → детект nmap.
- [ ] Установить Suricata для сетевого детекта.
- [ ] Написать **кастомное правило** для более точного детекта.
- [ ] Настроить **active response** — автоматический бан IP через fail2ban.
- [ ] Атаковать Windows 10 (`hydra rdp://`) и поймать в Wazuh.

---

## 🔗 VII. СВЯЗИ

- **Инструменты:** [[hydra]], [[nmap]], [[Wazuh]]
- **Техники:** [[T1110]], [[T1110.001]], [[T1021.004]]
- **Машины:** [[Ubuntu-Server]], [[Kali]], [[Wazuh]]
- **Лаба:** [[IP-Table]]
- **Предыдущий write-up:** [[01-first-scan]]

---

## 🐍 VIII. КОДЕКС

Это **первый полный цикл**:
- Red: атака Hydra.
- Blue: детект Wazuh.
- Purple: синтез в отчёт.

**Прогресс Кодекса:** Неделя 1 → завершена.

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Vigilo — я бодрствую. Vexo — я тревожу. Vinco — я побеждаю.*
> *Cor inquietum, hostis intus.*