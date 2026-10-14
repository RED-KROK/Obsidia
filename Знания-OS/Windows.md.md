---
type: knowledge
topic: OS / Windows
platform: Windows
status: studying
tags: [knowledge, os, windows, blue-team, red-team]
mitre:
  - T1082      # System Information Discovery
  - T1083      # File and Directory Discovery
created: 2025-09-29
---

> [!info] 📚 БАЗА ЗНАНИЙ
> **Тема:** Windows Basics
> **Для чего:** Понимание файловой системы, логов, путей — база для Blue Team
> **Уровень:** 🟢 Начальный

---

## 📖 I. WINDOWS vs LINUX: СООТВЕТСТВИЕ

| Linux | Windows | Назначение |
|---|---|---|
| `/home/user` | `C:\Users\user` | Домашние каталоги |
| `/etc` | `C:\ProgramData` + Реестр | Конфиги |
| `/bin`, `/usr/bin` | `C:\Windows\System32` | Исполняемые файлы |
| `/dev/sda1` | `\\.\PhysicalDrive0` | Устройства хранения |

> [!tip] Как запомнить
> Windows прячет конфиги в **реестре** + `ProgramData`. Linux держит всё в **/etc** текстом. Поэтому в Linux всё видно, а в Windows — надо знать, где искать.

---

## 📖 II. КЛЮЧЕВЫЕ ЛОГИ WINDOWS

| Путь | Что внутри |
|---|---|
| `C:\Windows\Logs\CBS\CBS.log` | Component-Based Servicing — установка обновлений |
| `C:\Windows\System32\winevt\Logs\` | **Event Logs** (главное!) |
| `C:\Windows\Logs\WindowsUpdate\` | Логи обновлений |
| `C:\Windows\INF\setupapi.dev.log` | Логи установки драйверов |

### Главный Event Log для SOC

**Файлы в `C:\Windows\System32\winevt\Logs\`:**
- `Security.evtx` — безопасность (входы, привилегии).
- `System.evtx` — системные события.
- `Application.evtx` — приложения.

> [!important] Ключевые Event ID (запомни!)
> | ID | Событие |
> |---|---|
> | **4624** | Успешный вход |
> | **4625** | Неудачный вход |
> | **4688** | Создание процесса |
> | **4720** | Создан пользователь |
> | **4726** | Удалён пользователь |
> | **4728** | Добавлен в группу |
> | **4672** | Специальные привилегии |
> | **1102** | Очистка Security-лога |

---

## 🛡️ III. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1082** — System Information Discovery
> - **T1083** — File and Directory Discovery
> - **T1562.002** — Impair Defenses: Disable Windows Event Logging

### Что видит защитник (Wazuh + Windows Agent)

| Источник | Что искать |
|---|---|
| **Security Event Log** | 4625 (много подряд = брутфорс), 4720 (новые юзеры) |
| **Sysmon Event ID 1** | Необычные процессы (`cmd.exe`, `powershell.exe -enc`) |
| **Sysmon Event ID 11** | Создание файлов в `C:\Windows\Temp\`, `C:\Users\Public\` |
| **Sysmon Event ID 13** | Изменения реестра (persistence) |
| **Wazuh Rule 60106** | Windows: попытки подбора пароля |
| **Wazuh Rule 60108** | Windows: новое приложение в автозагрузке |

### Практика в лабе

1. На Windows запусти:
   ```
   Event Viewer → Windows Logs → Security → фильтр 4625
   ```
2. Симулируй неудачный вход (несколько раз).
3. В Wazuh → **Security events** → фильтр `rule.id:60106`.
4. Разбери: Source IP, Target User, Rule, MITRE.

---

## 🔗 IV. СВЯЗИ

- **Инструменты:** [[Wazuh]], [[Sysmon]], [[msfconsole]]
- **Техники:** [[T1082]], [[T1083]], [[T1562.002]]
- **Лаба:** [[IP-Table]]
- **Машины:** [[Windows 10]]

---

## 📝 V. ШПАРГАЛКА

```
# Где искать логи
C:\Windows\Logs\CBS\CBS.log              # обновления
C:\Windows\System32\winevt\Logs\         # Event Logs

# Сравнение с Linux
/home/user       → C:\Users\user
/etc             → C:\ProgramData + реестр
/bin, /usr/bin   → C:\Windows\System32
/dev/sda1        → \\.\PhysicalDrive0

# Ключевые Event ID
4624 — успешный вход
4625 — неудачный вход
4688 — процесс
4720 — новый юзер
1102 — очистка лога
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Windows не прячет логи — он их складывает в Event Log. Твоя задача — не потерять.*