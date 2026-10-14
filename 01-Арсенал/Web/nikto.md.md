---
type: tool
name: Nikto
category: Web / Vulnerability Scanner
platform: Linux, Windows, macOS
language: Perl
status: studying
tags: [arsenal, web, scanner, vulnerabilities, red-team]
mitre:
  - T1595.002  # Active Scanning: Vulnerability Scanning
  - T1595.003  # Active Scanning: Wordlist Scanning
  - T1190      # Exploit Public-Facing Application
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Nikto
> **Категория:** Web Vulnerability Scanner
> **Платформа:** Linux, Windows, macOS
> **Язык:** Perl
> **Назначение:** Поиск опасных файлов, устаревших версий ПО, мисконфигов, известных уязвимостей веб-сервера
> **Статус:** 🟢 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Nikto** — классический сканер веб-серверов с открытым исходным кодом. Проверяет:
- **Опасные файлы** (`/admin`, `/backup.zip`, `.git/`, `.env`).
- **Устаревшие версии ПО** (Apache, nginx, PHP, CMS).
- **Мисконфиги** (открытые директории, default-страницы).
- **Известные уязвимости** (по базе сигнатур).
- **Опасные заголовки** (`Server`, `X-Powered-By`).

**Что НЕ делает:**
- Не эксплуатирует уязвимости.
- Не заменяет ручной анализ (Burp, ffuf).
- Не работает по авторизованным зонам без cookies.

> [!warning] Юридически
> Только для своей лабы или с письменного разрешения. Сканирование чужих сайтов — статья.

---

## 🚀 II. БАЗОВЫЙ СИНТАКСИС

```bash
nikto -h <целевой_хост>
```

Пример:
```bash
nikto -h http://192.168.56.20
```

**Что происходит:**
- Nikto отправляет **6700+ запросов** к серверу.
- Ищет файлы, мисконфиги, устаревшие версии.
- Выводит отчёт в консоль.

> [!warning] Медленно
> Один хост — от 2 до 30 минут. Используй `-Tuning` или `-maxtime` для ускорения.

---

## 🎯 III. ОСНОВНЫЕ ОПЦИИ

| Опция | Что делает |
|---|---|
| `-h <host>` | Целевой хост или IP |
| `-p <порты>` | Порты (по умолчанию 80, 443) |
| `-ssl` | Принудительно HTTPS |
| `-Tuning <набор>` | Выбрать конкретные тесты (см. таблицу ниже) |
| `-o <файл>` | Сохранить результат (`.html`, `.txt`, `.csv`) |
| `-C all` | Игнорировать проверку SSL-сертификата |
| `-maxtime <мин>` | Максимальное время сканирования |
| `-nolookup` | Не запрашивать DNS-имена для IP |
| `-no404` | Отключить проверку «файл не найден» |
| `-timeout <сек>` | Таймаут запроса |
| `-Pause <сек>` | Задержка между тестами |
| `-dbcheck` | Проверить целостность базы сигнатур |
| `-update` | Обновить базу Nikto |

---

## 🎯 IV. ОСНОВНЫЕ СЦЕНАРИИ

### 1. Базовое сканирование

```bash
nikto -h http://192.168.56.20
```

### 2. HTTPS-сайт

```bash
nikto -h https://example.com -ssl
```

### 3. Нестандартный порт

```bash
nikto -h example.com -p 8080
nikto -h example.com -ssl --port 443
```

### 4. С автопропуском SSL-сертификата (для self-signed)

```bash
nikto -h https://192.168.56.20 -ssl -C all
```

Полезно для тестовой лабы с самоподписанными сертификатами.

### 5. Ограничение по времени

```bash
nikto -h https://example.com -ssl -maxtime 1
```

Сканирование не дольше 1 минуты.

### 6. С задержкой между тестами (тише)

```bash
nikto -h example.com -Pause 10
```

10 секунд между запросами. Медленно, но не попадёшь под fail2ban.

### 7. Без DNS-запросов (быстрее)

```bash
nikto -h 192.168.56.20 -nolookup
```

### 8. Отчёт в HTML

```bash
nikto -h http://192.168.56.20 -o report.html
```

Открой отчёт в браузере — увидишь красивую таблицу.

### 9. Отчёт в CSV (для автоматизации)

```bash
nikto -h http://192.168.56.20 -o report.csv -Format csv
```

### 10. Проверка базы сигнатур

```bash
nikto -dbcheck
```

Проверяет, не повреждена ли база. Если ошибки — переустанови:

```bash
sudo apt install --reinstall nikto
```

---

## 🎛️ V. TUNING — УПРАВЛЕНИЕ ТЕСТАМИ

Флаг `-Tuning` позволяет выбрать **типы тестов**. По умолчанию — все. Если нужны конкретные — используй коды.

| Код | Что тестируется |
|---|---|
| `0` | Загрузка файла (File Upload) |
| `1` | Интересные файлы / видны в логах |
| `2` | Мисконфигурации / default-файлы |
| `3` | Раскрытие информации |
| `4` | XSS / Script / HTML injection |
| `5` | Remote file retrieval (внутри web root) |
| `6` | Отказ в обслуживании (DoS) |
| `7` | Remote file retrieval (на всём сервере) |
| `8` | Выполнение команд / Remote shell |
| `9` | SQL-инъекция |
| `a` | Обход аутентификации |
| `b` | Идентификация ПО |
| `c` | Remote source inclusion |
| `x` | Обратные опции |

**Примеры:**

```bash
# Только SQL-инъекции + XSS
nikto -h http://target.com -Tuning 49

# Только идентификация ПО (быстро)
nikto -h http://target.com -Tuning b

# Проверка конфигурации + устаревшие версии
nikto -h http://target.com -Tuning 2b
```

> [!tip] Совет
> Комбинируй коды: `-Tuning 123` = тесты 1, 2 и 3. По умолчанию — все.

---

## 🏴 VI. ПРИМЕНЕНИЕ В ПЕНТЕСТЕ

### Классический сценарий

```bash
# 1. Разведка порта 80
nmap -sV -p 80,443 192.168.56.20

# 2. Скан Nikto
nikto -h http://192.168.56.20 -o nikto-report.html

# 3. Анализ результатов
# Смотрим:
# - Устаревшие версии (Apache 2.2.8, PHP 5.2)
# - Опасные файлы (/phpMyAdmin, /backup)
# - Заголовки (Server: Apache/2.2.8)
```

### Что искать в отчёте

| Строка | Что значит |
|---|---|
| `+ Server: Apache/2.2.8` | Устаревшая версия → известные CVE |
| `+ /phpMyAdmin/: phpMyAdmin directory found` | Панель управления БД доступна |
| `+ OSVDB-XXXX: /admin/: Admin login page` | Админка открыта |
| `+ /backup.zip: Backup file found` | Утечка данных |
| `+ Apache/2.2.8 appears to be outdated` | Прямое указание на версию |

### Где брать словари для ffuf (если Nikto нашёл мало)

После Nikto всегда стоит пройтись по **расширенному** словарю через [[ffuf]]:

```bash
ffuf -u http://192.168.56.20/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
     -fc 404 -c
```

Nikto — быстрый, но ограниченный. [[ffuf]] — медленный, но глубокий.

---

## 🛡️ VII. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1595.002** — Vulnerability Scanning
> - **T1595.003** — Wordlist Scanning
> - **T1190** — Exploit Public-Facing Application

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Apache/Nginx access.log** | Множественные 404 от одного IP |
| **Wazuh Rule 31101** | Multiple web server errors |
| **Wazuh Rule 31103** | Common web attack |
| **Wazuh Rule 31105** | SQL injection |
| **Suricata** | Сигнатуры Nikto |
| **ModSecurity** | Правила OWASP CRS срабатывают на `nikto` UA |

### Признаки Nikto в логах

```
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET /admin HTTP/1.1" 404 198 "-" "Mozilla/5.00 (Nikto/2.5.0)"
192.168.56.107 - - [29/Sep/2026:10:23:46] "GET /.git/config HTTP/1.1" 404 198 "-" "Mozilla/5.00 (Nikto/2.5.0)"
192.168.56.107 - - [29/Sep/2026:10:23:47] "GET /backup.zip HTTP/1.1" 404 198 "-" "Mozilla/5.00 (Nikto/2.5.0)"
192.168.56.107 - - [29/Sep/2026:10:23:48] "GET /phpMyAdmin/ HTTP/1.1" 404 198 "-" "Mozilla/5.00 (Nikto/2.5.0)"
```

> [!danger] Красные флаги
> - **User-Agent** `Nikto` или `Mozilla/5.00 (Nikto/2.5.0)` — прямая подпись.
> - **Много запросов** к несуществующим путям (`/.git/`, `/backup.zip`).
> - **Обращения к админкам** (`/admin/`, `/phpMyAdmin/`, `/wp-admin/`).
> - **Попытки обхода** (`/../`, `%2e%2e/`, `//admin`).
> - **Скорость** — 50–100 запросов в секунду.

### Практика в лабе

1. С Kali:
   ```bash
   nikto -h http://192.168.56.20 -o nikto.html
   ```
2. В Wazuh → **Security events** → фильтр по `rule.id:31101` или по User-Agent `Nikto`.
3. Проверь: сколько запросов, какие URL, откуда.

> [!info] Как защититься
> - **WAF** (ModSecurity + OWASP CRS) — блокирует по сигнатурам.
> - **User-Agent block** — режет `Nikto`, `sqlmap` и подобных.
> - **Rate limiting** — ограничение запросов с одного IP.
> - **Fail2ban** — автобан после N 404.
> - **Скрытие заголовков** `Server`, `X-Powered-By`.
> - **Убрать `.git/`, `.env`, backups из корня.**

---

## 🩺 VIII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| Слишком медленно | `-Tuning b` (только ID ПО) или `-maxtime 5` |
| Ложные срабатывания | Отключи `-no404`, добавь `-no404` (да, оба) |
| SSL-ошибки | Добавь `-ssl -C all` |
| Не сканирует HTTPS | Явно укажи `https://` в `-h` |
| `dbcheck` ругается | `sudo apt install --reinstall nikto` |
| Устаревшая база | `nikto -update` |
| Ничего не находит | Сервер закрыт WAF — меняй UA через `-useragent` |
| Слишком тихо | Убери `-Pause` |

### Полезные комбинации

```bash
# Быстро (только ПО и конфиг)
nikto -h http://target.com -Tuning 2b -maxtime 3

# Тихо (не попадёшь под fail2ban)
nikto -h http://target.com -Pause 5 -timeout 10

# Смена User-Agent
nikto -h http://target.com -useragent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

> [!tip] Обход WAF
> Если Nikto блокируется — **всегда** меняй User-Agent:
> ```bash
> nikto -h http://target.com -useragent "Mozilla/5.0 (compatible; Googlebot/2.1)"
> ```
> Но помни: продвинутые WAF (Cloudflare) вычисляют бота по поведению, а не только UA.

---

## 🔗 IX. СВЯЗИ

- **Инструменты:** [[nmap]], [[ffuf]], [[whatweb]], [[wapiti]], [[curl]], [[httpx]]
- **Техники:** [[T1595.002]], [[T1595.003]], [[T1190]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[ModSecurity]], [[Fail2ban]]

---

## 📚 X. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] Сравнение с [[wapiti]] (более глубокий сканер).
- [ ] Написание своих правил Nikto (`db_tests`).
- [ ] Интеграция с Burp Suite (через прокси).
- [ ] Автоматизация: `nikto + ffuf + nuclei` в одном скрипте.
- [ ] Детектив Nikto в ModSecurity.
- [ ] Ручной анализ результатов Nikto.

---

## 📝 XI. ШПАРГАЛКА

```bash
# Базовая проверка
nikto -h http://target.com

# HTTPS
nikto -h https://target.com -ssl -C all

# Быстро (только ПО)
nikto -h http://target.com -Tuning b -maxtime 3

# Тихо
nikto -h http://target.com -Pause 5 -timeout 10

# Смена User-Agent (обход WAF)
nikto -h http://target.com -useragent "Mozilla/5.0 (Windows NT 10.0)"

# Отчёт HTML
nikto -h http://target.com -o report.html

# Только SQL-инъекции
nikto -h http://target.com -Tuning 9

# Только XSS
nikto -h http://target.com -Tuning 4

# Проверка базы
nikto -dbcheck
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Nikto стучит в каждую дверь, дёргает каждую ручку. Быстро, шумно, но находит то, что пропустит ленивый пентестер.*