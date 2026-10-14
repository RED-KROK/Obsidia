---
type: tool
name: Wapiti
category: Web / Vulnerability Scanner
platform: Linux, Windows, macOS
language: Python
status: studying
tags: [arsenal, web, scanner, vulnerabilities, black-box, red-team]
mitre:
  - T1595.002  # Active Scanning: Vulnerability Scanning
  - T1190      # Exploit Public-Facing Application
  - T1059      # Command and Scripting Interpreter
  - T1505.003  # Server Software Component: Web Shell
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Wapiti
> **Категория:** Web Vulnerability Scanner (Black-box)
> **Платформа:** Linux, Windows, macOS
> **Язык:** Python
> **Назначение:** Автоматический поиск уязвимостей в веб-приложениях
> **Метод:** Black-box — сканирует сайт, как чёрный ящик, без доступа к коду
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Wapiti** — сканер веб-уязвимостей методом **«чёрного ящика»** (black-box). Он **не анализирует исходный код** — он ведёт себя как пользователь:
1. Обходит сайт по ссылкам.
2. Заполняет формы.
3. Отправляет **вредоносные payload'ы**.
4. Анализирует ответы и **определяет уязвимости**.

**Что ищет:**
- SQL-инъекции (SQLi).
- XSS (Cross-Site Scripting).
- LFI/RFI (Local/Remote File Inclusion).
- Command Injection.
- XXE.
- Open Redirect.
- Backup-файлы.
- Опасные HTTP-методы.
- Утечки информации.

> [!warning] Юридически
> Только для своей лабы или с письменного разрешения. Сканирование чужих сайтов — статья.

---

## 🚀 II. БАЗОВЫЙ СИНТАКСИС

```bash
wapiti -u <URL> [опции]
```

Пример:
```bash
wapiti -u http://target.com
```

**Что произойдёт:** Wapiti просканирует сайт, обойдёт ссылки, проверит формы и выдаст **отчёт с найденными уязвимостями**.

> [!warning] По умолчанию — не всё
> Wapiti **не запускает все модули** по умолчанию. Чтобы включить всё — используй `-m all`.

---

## 🎯 III. ОСНОВНЫЕ ОПЦИИ

### Цель и глубина

| Флаг | Что делает |
|---|---|
| `-u <URL>` | Целевой URL (обязательно) |
| `-m all` | Включить **ВСЕ** модули сканирования |
| `-m "sql,xss"` | Только конкретные модули |
| `--scope page` | Сканировать только одну директорию |
| `--scope folder` | Только текущая папка (не лезет в `/admin/`) |
| `--scope url` | Только указанный URL (самый узкий) |
| `-l <1-5>` | Уровень глубины (по умолч. 2) |

### Авторизация

| Флаг | Что делает |
|---|---|
| `--cookie "PHPSESSID=abc123"` | Передать cookies (для авторизованного сканирования) |
| `--auth-type basic` | Basic/Digest/NTLM авторизация |
| `--auth-cred "admin:pass"` | Учётные данные |

### Отчёты

| Флаг | Что делает |
|---|---|
| `-f html` | Формат отчёта (HTML — красиво) |
| `-f json` | JSON — для автоматизации |
| `-o /path/report.html` | Куда сохранить отчёт |
| `--flush-attacks` | Удалить старые результаты перед новым запуском |

### Скорость и лимиты

| Флаг | Что делает |
|---|---|
| `--max-links-per-page <N>` | Ограничить ссылок на странице |
| `--max-scan-time <сек>` | Максимальное время сканирования |
| `--forms` | Автоматически заполнять формы (медленно, но глубоко) |
| `--no-follow-redirects` | Не следовать редиректам |

### Обход защиты

| Флаг | Что делает |
|---|---|
| `--header "X-Forwarded-For: 127.0.0.1"` | Свой заголовок (обход WAF, IP-based защит) |
| `--user-agent "Mozilla/5.0"` | Подмена User-Agent |
| `--proxy http://127.0.0.1:8080` | Прокси (Burp, ZAP, mitmproxy) |
| `--verbose` | Подробный вывод (отладка) |

---

## 🎯 IV. ОСНОВНЫЕ СЦЕНАРИИ

### 1. Базовый запуск (быстрый тест)

```bash
wapiti -u http://target.com
```

Первый прогон — понять, что за сайт.

### 2. Полное сканирование (все модули)

```bash
wapiti -u http://target.com -m all
```

Максимум модулей, но дольше.

### 3. Быстрая проверка только SQLi и XSS

```bash
wapiti -u http://target.com -m "sql,xss" -f html -o report.html
```

Когда нужно быстро проверить главные уязвимости.

### 4. С авторизацией через Cookie

```bash
wapiti -u http://target.com -m all \
  --cookie "PHPSESSID=abc123; security=low" \
  --forms
```

> [!tip] Где взять Cookie
> Войди в приложение в браузере → DevTools (F12) → **Application** → **Cookies**. Скопируй значения сюда.

### 5. Basic Auth

```bash
wapiti -u http://target.com \
  --auth-type basic \
  --auth-cred "admin:password" \
  -m all
```

### 6. Через прокси Burp (видеть все запросы)

```bash
wapiti -u http://target.com -m all \
  --proxy http://127.0.0.1:8080 \
  --max-scan-time 1800
```

Ты видишь все запросы в Burp → можешь анализировать вручную.

### 7. Обход WAF

```bash
wapiti -u http://target.com -m all \
  --user-agent "Mozilla/5.0" \
  --header "X-Forwarded-For: 8.8.8.8"
```

Подмена User-Agent + заголовок для обхода IP-based защиты.

### 8. Только текущая папка

```bash
wapiti -u http://target.com/admin --scope folder -m all
```

Не уйдёт за пределы `/admin/`.

### 9. Поиск открытых редиректов

```bash
wapiti -u http://target.com -m redirect --no-follow-redirects
```

Не переходит по редиректам — фиксирует их как потенциальные open redirect.

---

## 🎛️ V. МОДУЛИ (`-m`)

| Модуль | Что ищет |
|---|---|
| `sql` | SQL-инъекции |
| `xss` | Cross-Site Scripting |
| `file` | LFI / RFI / Path traversal |
| `exec` | Command Injection |
| `crlf` | CRLF Injection |
| `xxe` | XML External Entity |
| `redirect` | Open Redirect |
| `ssrf` | Server-Side Request Forgery |
| `backup` | Backup-файлы (`.bak`, `.old`) |
| `brute` | Брутфорс директорий |
| `nikto` | Запустить Nikto изнутри |
| `wapp` | Определение технологий |
| `all` | **Все модули** |

**Пример комбинированного запуска:**
```bash
wapiti -u http://target.com -m "sql,xss,file,exec"
```

---

## 🎯 VI. УРОВНИ ГЛУБИНЫ (`-l`)

| Уровень | Описание |
|---|---|
| `1` | Быстро, поверхностно |
| `2` | По умолчанию |
| `3` | Глубже, дольше |
| `4` | Очень глубоко |
| `5` | Максимум (может занять часы) |

> [!tip] Правило Легиона
> Начинай с `-l 1` для разведки. Потом — `-l 2-3` для глубины. `-l 5` — только на своей лабе.

---

## 🏴 VII. ПРИМЕНЕНИЕ В ПЕНТЕСТЕ

### Классический сценарий

```bash
# 1. Разведка
nmap -sV -p 80,443 target.com
whatweb target.com
nikto -h http://target.com

# 2. Быстрый скан на главные уязвимости
wapiti -u http://target.com -m "sql,xss" -f html -o wapiti-fast.html

# 3. Полный скан
wapiti -u http://target.com -m all -l 3 -f html -o wapiti-full.html --max-scan-time 3600

# 4. Анализ отчёта
# - Открыть HTML в браузере
# - Проверить каждую уязвимость вручную (Burp)
```

### Что искать в отчёте

| Категория | Приоритет |
|---|---|
| **SQLi** | 🔴 Критично |
| **Command Injection** | 🔴 Критично |
| **LFI/RFI** | 🔴 Критично |
| **XSS (stored)** | 🟠 Высокий |
| **XSS (reflected)** | 🟡 Средний |
| **Open Redirect** | 🟡 Средний |
| **Backup-файлы** | 🟡 Средний |
| **Утечки версий** | 🟢 Низкий |

> [!warning] Не верь слепо
> Wapiti часто даёт **ложные срабатывания**. Каждую находку нужно **подтвердить вручную** (Burp, curl).

---

## 🛡️ VIII. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1595.002** — Vulnerability Scanning
> - **T1190** — Exploit Public-Facing Application
> - **T1059** — Command and Scripting Interpreter
> - **T1505.003** — Web Shell

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Apache/Nginx access.log** | Payload'ы Wapiti (`' OR 1=1--`, `<script>alert`) |
| **Wazuh Rule 31101** | Multiple web server errors |
| **Wazuh Rule 31103** | Common web attack |
| **Wazuh Rule 31105** | SQL injection |
| **Wazuh Rule 31106** | XSS attack |
| **Wazuh Rule 31108** | Command injection |
| **Suricata** | Сигнатуры атак |
| **ModSecurity** | Срабатывания OWASP CRS |

### Признаки Wapiti в логах

```
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET /page?id=1%27%20OR%201%3D1-- HTTP/1.1" 500 1234 "-" "Wapiti/3.1.6"
192.168.56.107 - - [29/Sep/2026:10:23:46] "GET /search?q=<script>alert(1)</script> HTTP/1.1" 200 4523 "-" "Wapiti/3.1.6"
192.168.56.107 - - [29/Sep/2026:10:23:47] "GET /?file=../../../../etc/passwd HTTP/1.1" 403 198 "-" "Wapiti/3.1.6"
```

> [!danger] Красные флаги
> - **User-Agent** `Wapiti/*` — прямая подпись.
> - **Payload'ы в URL** — `' OR 1=1`, `<script>`, `../../etc/passwd`.
> - **Много 500-х** к одному endpoint.
> - **Аномальные параметры** — `id=1'`, `file=../../`.
> - **Регулярные интервалы** между запросами.

### Практика в лабе

1. С Kali:
   ```bash
   wapiti -u http://192.168.56.20 -m "sql,xss,file" -f html -o wapiti.html
   ```
2. В Wazuh → **Security events** → фильтр по `rule.id:31105` (SQLi) или `31106` (XSS).
3. Разбери алерт: URL, payload, rule, MITRE.

> [!info] Как защититься
> - **WAF** (ModSecurity + OWASP CRS) — блокирует payload'ы.
> - **Parameterized queries** — защита от SQLi.
> - **Output encoding** — защита от XSS.
> - **Rate limiting** — ограничение запросов с одного IP.
> - **Fail2ban** — бан по 500-м.
> - **User-Agent block** — резать `Wapiti`, `sqlmap`, `nikto`.

---

## 🩺 IX. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `wapiti: command not found` | `sudo apt install wapiti` или `pip install wapiti3` |
| Слишком медленно | Убери `-m all`, используй `-m "sql,xss"`, `-l 1` |
| Много ложных срабатываний | Проверяй каждую находку вручную (Burp) |
| Не работает авторизация | Проверь Cookie — они могли истечь |
| Прокси не работает | Проверь `--proxy http://127.0.0.1:8080` |
| Зависает на формах | Убери `--forms` |
| Слишком много запросов | `--max-links-per-page 50`, `--max-scan-time 1800` |
| Не сохраняет отчёт | Проверь права на папку: `-o /tmp/report.html` |

### Проверка установки

```bash
wapiti --version
wapiti --help
```

### Альтернативы

- [[nikto]] — быстрее, но поверхностнее.
- **OWASP ZAP** — GUI + автоматизация.
- **Burp Suite** — золотой стандарт (Pro).
- **Nuclei** — быстрый template-based сканер.
- **sqlmap** — специализирован на SQLi.

---

## 🔗 X. СВЯЗИ

- **Инструменты:** [[nikto]], [[ffuf]], [[curl]], [[sqlmap]], [[nmap]], [[whatweb]]
- **Техники:** [[T1595.002]], [[T1190]], [[T1059]], [[T1505.003]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[ModSecurity]], [[Suricata]]

---

## 📚 XI. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] **sqlmap** — специализация на SQLi.
- [ ] **OWASP ZAP** — аналог, но с GUI.
- [ ] **Nuclei** — быстрый, template-based.
- [ ] Написание кастомных payload'ов для Wapiti.
- [ ] Детект SQLi/XSS в ModSecurity.
- [ ] Ручное подтверждение уязвимостей в Burp.

---

## 📝 XII. ШПАРГАЛКА

```bash
# Быстрый тест
wapiti -u http://target.com -m "sql,xss" -f html -o report.html

# Полное сканирование
wapiti -u http://target.com -m all -l 3 -f html -o full.html

# С авторизацией
wapiti -u http://target.com -m all --cookie "PHPSESSID=abc" --forms

# Basic Auth
wapiti -u http://target.com --auth-type basic --auth-cred "admin:pass"

# Через прокси
wapiti -u http://target.com -m all --proxy http://127.0.0.1:8080

# Обход WAF
wapiti -u http://target.com --user-agent "Mozilla/5.0" --header "X-Forwarded-For: 8.8.8.8"

# Только текущая папка
wapiti -u http://target.com/admin --scope folder

# С ограничением времени
wapiti -u http://target.com -m all --max-scan-time 1800 -o report.html
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Wapiti — это гончая. Она бежит по следу, кусает каждую ссылку и возвращает добычу в виде отчёта. Но решение о ценности добычи — за тобой.*