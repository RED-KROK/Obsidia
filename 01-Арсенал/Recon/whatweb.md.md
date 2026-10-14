---
type: tool
name: WhatWeb
category: Recon / Web Fingerprinting
platform: Linux, Windows, macOS
language: Ruby
status: studying
tags: [arsenal, recon, web, fingerprinting, cms, red-team]
mitre:
  - T1592.002  # Gather Victim Host Information: Software
  - T1592.004  # Gather Victim Host Information: Client Configurations
  - T1595.002  # Active Scanning: Vulnerability Scanning
  - T1518.001  # Software Discovery: Security Software Discovery
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** WhatWeb
> **Категория:** Recon / Web Fingerprinting
> **Платформа:** Linux, Windows, macOS
> **Язык:** Ruby
> **Назначение:** Определение технологий веб-сайта (CMS, сервер, фреймворк, версии)
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**WhatWeb** — инструмент для **fingerprinting** веб-приложений. Определяет:
- **CMS** (WordPress, Joomla, Drupal).
- **Веб-сервер** (Apache, nginx, IIS) + версию.
- **Язык** (PHP, Python, Ruby, ASP.NET).
- **JavaScript-библиотеки** (jQuery, React, Angular).
- **Аналитику** (Google Analytics, Yandex.Metrika).
- **Заголовки безопасности** (X-Frame-Options, CSP).
- **Панели управления** (`/wp-admin`, `/administrator`).

**Что даёт:**
1. **Подготовка к тестированию.** Узнал CMS → знаешь, где искать уязвимости.
2. **Подтверждение версии.** Если известна CVE в nginx 1.18.0 — WhatWeb скажет, есть ли она.
3. **Карта атакуемой поверхности.** Понимаешь, какие векторы доступны.

> [!success] Тихий режим
> В обычном режиме WhatWeb ведёт себя **как обычный браузер** — отправляет всего несколько запросов. Это делает его **малозаметным** в логах.

---

## 🚀 II. УСТАНОВКА

### Kali Linux (уже предустановлен)

```bash
whatweb --version
```

### Если нет

```bash
sudo apt install whatweb
```

### Из GitHub

```bash
git clone https://github.com/urbanadventurer/WhatWeb.git
cd WhatWeb
bundle install
```

---

## 🎯 III. БАЗОВЫЙ СИНТАКСИС

```bash
whatweb [опции] <URL>
```

Пример:
```bash
whatweb example.com
```

**Вывод:**
```
http://example.com [200 OK] Apache[2.4.41], Country[UNITED STATES][US],
HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41], IP[93.184.216.34],
Title[Example Domain]
```

---

## 🎯 IV. ОСНОВНЫЕ ФЛАГИ

| Флаг | Что делает |
|---|---|
| `-a <1-4>` | Уровень агрессивности (1 — тихо, 4 — максимум) |
| `-v` | Подробный вывод (какая сигнатура сработала и как) |
| `-i <файл>` | Список целей из файла |
| `--url-prefix <path>` | Добавить префикс к URL (для виртуальных хостов) |
| `--follow-redirect` | Следовать редиректам |
| `--log-json=<файл>` | Экспорт в JSON |
| `--log-brief=<файл>` | Краткий вывод в файл |
| `--log-verbose=<файл>` | Подробный вывод в файл |
| `-l` | Список всех плагинов |
| `-I <плагин>` | Подробная информация о конкретном плагине |
| `--user-agent <UA>` | Свой User-Agent |
| `--proxy <proxy>` | Прокси |
| `--no-errors` | Не показывать ошибки |

### Уровни агрессивности (`-a`)

| Уровень | Режим | Что делает |
|---|---|---|
| `1` | Stealthy | 1 запрос на цель |
| `2` | (по умолч.) | Несколько запросов |
| `3` | Aggressive | Активные проверки, максимум деталей |
| `4` | Heavy | Экстремально агрессивно (может упасть) |

> [!tip] Какой использовать
> - **Разведка на реальной цели** → `-a 1` или `-a 3`.
> - **Своя лаба** → `-a 3` или `-a 4`.
> - **Обход WAF** → `-a 1` с кастомным User-Agent.

---

## 🎯 V. ОСНОВНЫЕ СЦЕНАРИИ

### 1. Базовое сканирование (быстро)

```bash
whatweb example.com
```

### 2. Агрессивный режим (максимум информации)

```bash
whatweb -a 3 example.com
```

Определит всё, что можно: CMS, версии, плагины, пути.

### 3. Verbose + редиректы

```bash
whatweb -v --follow-redirect example.com
```

Показывает, **какая сигнатура** сработала и **как** был определён каждый элемент.

### 4. Список целей из файла

```bash
whatweb -i targets.txt
```

`targets.txt`:
```
example.com
test.com
192.168.56.20
192.168.56.30
```

### 5. Экспорт в JSON

```bash
whatweb --log-json=result.json example.com
```

Для автоматизации и парсинга через `jq` или Python.

### 6. Информация о плагине

```bash
whatweb -I wordpress
```

Показывает, **как** WhatWeb определяет WordPress.

### 7. Виртуальные хосты (с префиксом)

```bash
whatweb --url-prefix /blog example.com
```

Полезно, когда сайт отвечает по-разному в зависимости от пути.

### 8. Свой User-Agent (обход WAF)

```bash
whatweb --user-agent "Mozilla/5.0 (Windows NT 10.0)" example.com
```

---

## 🏴 VI. ПРИМЕНЕНИЕ В ПЕНТЕСТЕ

### Классическая цепочка

```bash
# 1. Разведка портов
nmap -sV -p 80,443 target.com

# 2. Fingerprinting технологий
whatweb -a 3 target.com

# 3. Проверка конкретных технологий
# Если WordPress → wp-scan, /wp-admin, /wp-login.php
# Если nginx → проверить версии, CVE
# Если PHP → проверить, есть ли уязвимые модули

# 4. Глубокий скан уязвимостей
nikto -h target.com
wapiti -u http://target.com -m all
```

### Что искать в выводе

| Строка | Что значит |
|---|---|
| `WordPress[6.2]` | Устаревшая версия → известные CVE |
| `Apache[2.4.41]` | Версия сервера → CVE, мисконфиги |
| `PHP[7.4]` | Устаревшая версия → EOL |
| `X-Powered-By[PHP/5.6]` | Утечка версии |
| `jQuery[1.7.2]` | Устаревшая библиотека → XSS |
| `Google-Analytics[UA-XXX]` | Идентификация компании |
| `Country[UNITED STATES][US]` | Геолокация сервера |

### Связка с другими инструментами

```bash
# WhatWeb → httpx → nmap
whatweb -a 3 example.com -v
httpx -u example.com -td -title -sc
nmap -sV -sC example.com
```

---

## 🛡️ VII. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1592.002** — Gather Victim Host Information: Software
> - **T1595.002** — Vulnerability Scanning
> - **T1518.001** — Security Software Discovery

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Apache/Nginx access.log** | Запросы к `/wp-admin/`, `/administrator/`, `/phpmyadmin/` |
| **Wazuh Rule 31101** | Multiple web server errors |
| **Wazuh Rule 31103** | Common web attack |
| **Suricata** | Сигнатуры сканирования |
| **ModSecurity** | Срабатывания на подозрительные пути |
| **CDN / WAF** | Аномальные User-Agent (`WhatWeb/*`) |

### Признаки WhatWeb в логах

```
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET / HTTP/1.1" 200 1234 "-" "WhatWeb/0.5.5"
192.168.56.107 - - [29/Sep/2026:10:23:46] "GET /wp-login.php HTTP/1.1" 200 4523 "-" "WhatWeb/0.5.5"
192.168.56.107 - - [29/Sep/2026:10:23:47] "GET /administrator/ HTTP/1.1" 404 198 "-" "WhatWeb/0.5.5"
192.168.56.107 - - [29/Sep/2026:10:23:48] "GET /phpmyadmin/ HTTP/1.1" 404 198 "-" "WhatWeb/0.5.5"
```

> [!danger] Красные флаги
> - **User-Agent** `WhatWeb/*` — прямая подпись.
> - **Запросы к админкам** (`/wp-admin/`, `/administrator/`, `/phpmyadmin/`).
> - **Множественные 404** к стандартным путям CMS.
> - **Обращения к `robots.txt`** с последующим сканированием.
> - **Много запросов к `/`** с разными User-Agent.

### Практика в лабе

1. С Kali:
   ```bash
   whatweb -a 3 http://192.168.56.20
   ```
2. В Wazuh → **Security events** → фильтр по `rule.id:31101` или по User-Agent `WhatWeb`.
3. Проверь: сколько запросов, какие URL, откуда.

> [!info] Как защититься
> - **User-Agent block** — резать `WhatWeb`, `WPScan`, `Nikto`.
> - **WAF** (ModSecurity) — блок по сигнатурам сканеров.
> - **Убрать отпечатки** из заголовков:
>   - `Server: nginx` (без версии).
>   - Убрать `X-Powered-By`.
>   - Скрыть `X-AspNet-Version`.
> - **Rate limiting** — ограничение частоты запросов.
> - **robots.txt** — не давать подсказки.

---

## 🩺 VIII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| Медленно | Уменьши уровень `-a 1` |
| Много ложных срабатываний | `-v` для проверки, какая сигнатура сработала |
| Не определяет CMS | Используй `-a 3` (агрессивный режим) |
| `SSL error` | Добавь `--no-check-certificate` или используй `http://` |
| WAF блокирует | Смени User-Agent, снизь уровень, используй прокси |
| Нет вывода JSON | Проверь права: `--log-json=/tmp/result.json` |
| Плагин не работает | `whatweb -I <плагин>` для проверки |

### Полезные проверки

```bash
whatweb --version
whatweb --help
whatweb -l                    # список плагинов
```

### Обновление

```bash
sudo apt update && sudo apt upgrade whatweb
```

---

## 🔗 IX. СВЯЗИ

- **Инструменты:** [[nmap]], [[httpx]], [[nikto]], [[wapiti]], [[curl]], [[wpscan]]
- **Техники:** [[T1592.002]], [[T1592.004]], [[T1595.002]], [[T1518.001]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[ModSecurity]], [[Suricata]]

---

## 📚 X. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] **WPScan** — специализация на WordPress.
- [ ] **JoomScan** — для Joomla.
- [ ] **Droopescan** — для Drupal.
- [ ] **WhatWeb-плагины** — написание своих сигнатур.
- [ ] **Retire.js** — поиск уязвимых JS-библиотек.
- [ ] **Nuclei** — tech detection + vuln scanning в одном.

---

## 📝 XI. ШПАРГАЛКА

```bash
# Базовое сканирование
whatweb example.com

# Агрессивный режим
whatweb -a 3 example.com

# Verbose + редиректы
whatweb -v --follow-redirect example.com

# Список целей из файла
whatweb -i targets.txt

# JSON-вывод
whatweb --log-json=result.json example.com

# Инфо о плагине
whatweb -I wordpress

# Виртуальный хост
whatweb --url-prefix /blog example.com

# Свой User-Agent (обход)
whatweb --user-agent "Mozilla/5.0" example.com

# С прокси (Burp)
whatweb --proxy 127.0.0.1:8080 example.com
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *WhatWeb — это опознаватель. Ты смотришь на сайт — и сразу видишь: WordPress, nginx, PHP 7.4. Дальше — только вопрос, какую CVE применить.*