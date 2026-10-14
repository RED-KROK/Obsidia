---
type: tool
name: ffuf
category: Web / Fuzzing
platform: Linux, Windows, macOS
language: Go
status: studying
tags: [arsenal, web, fuzzing, bruteforce, red-team]
mitre:
  - T1595.003  # Active Scanning: Wordlist Scanning
  - T1190      # Exploit Public-Facing Application
  - T1110      # Brute Force
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** ffuf (Fuzz Faster U Fool)
> **Категория:** Web / Fuzzing
> **Платформа:** Linux, Windows, macOS
> **Язык:** Go (быстрый, многопоточный)
> **Назначение:** Перебор директорий, файлов, поддоменов, параметров
> **Статус:** 🟢 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**ffuf** — быстрый инструмент для **фаззинга веб-приложений**. Написан на Go, работает в разы быстрее, чем `dirb` или `gobuster`. Основное применение:
- Перебор директорий и файлов (`/admin`, `/backup.zip`).
- Перебор поддоменов (`dev.target.com`, `api.target.com`).
- Перебор параметров (`?id=FUZZ`, `?user=FUZZ`).
- Брутфорс логинов (`user=admin&pass=FUZZ`).

**Ключевая идея:** ты подставляешь слово из словаря в место `FUZZ` в URL, заголовке, теле запроса — и смотришь, что отвечает сервер.

---

## 🚀 II. БАЗОВЫЙ СИНТАКСИС

```bash
ffuf [опции] -u URL -w словарь:местозаполнитель
```

| Параметр | Что делает |
|---|---|
| `-u` | Целевой URL с местозаполнителем `FUZZ` |
| `-w` | Словарь (файл, stdin, несколько) |
| `-H` | Добавить заголовок |
| `-X` | HTTP-метод (GET, POST, PUT...) |
| `-d` | Данные для POST-запроса |
| `-c` | Цветной вывод |
| `-t` | Количество потоков (по умолч. 40) |
| `-p` | Задержка между запросами (0.1s, 1s) |
| `-e` | Расширения файлов (.php,.html,.txt) |
| `-ac` | Автокалибровка фильтров |
| `-recursion` | Рекурсивное сканирование |
| `-proxy` | Прокси (напр. `http://127.0.0.1:8080`) |

### Фильтры (что исключить)

| Флаг | Исключает по... |
|---|---|
| `-fc` | Коду ответа (404, 403) |
| `-fw` | Количеству слов |
| `-fl` | Количеству строк |
| `-fs` | Размеру ответа |

### Фильтры (что показать)

| Флаг | Показывает только... |
|---|---|
| `-mc` | Указанные коды ответа (200, 301) |

### Вывод

| Флаг | Что делает |
|---|---|
| `-o results.json` | Сохранить в файл |
| `-of json` | Формат: json, csv, html, ecsv |
| `-v` | Подробный вывод |
| `-s` | Тихий режим (только найденное) |

---

## 🎯 III. ОСНОВНЫЕ СЦЕНАРИИ

### 1. Перебор директорий

```bash
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

**Что найдёт:** `/admin`, `/login`, `/backup`, `/uploads` — всё, что есть в словаре.

### 2. Перебор файлов с расширениями

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -e .php,.html,.txt
```

Флаг `-e` добавляет расширения к каждому слову: `admin.php`, `admin.html`, `admin.txt`.

### 3. POST-запрос с перебором параметра

```bash
ffuf -X POST -u http://target.com/login \
     -d "user=admin&pass=FUZZ" \
     -w passlist.txt \
     -fc 401
```

**Что делает:** перебирает пароли для логина `admin`. `-fc 401` исключает неудачные попытки, оставляя только успех (200) или редирект (302).

### 4. Фильтрация по коду ответа

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -fc 404
```

Исключает всё, что отвечает 404. Показывает только «живые» страницы.

### 5. Фильтрация по размеру

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -fs 1234
```

Исключает ответы размером ровно 1234 байта. Полезно, когда сервер отвечает одинаковой страницей 404, но с кодом 200.

### 6. С заголовками и куками

```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -H "User-Agent: ffuf" \
     -H "Cookie: session=abc123"
```

> [!tip] Совет
> Если приложение требует авторизации — **обязательно** добавляй `-H "Cookie: ..."`. Иначе будешь фаззить страницу логина, а не целевую область.

### 7. Перебор поддоменов (виртуальные хосты)

```bash
ffuf -u http://FUZZ.target.com -w subdomains.txt -H "Host: FUZZ.target.com"
```

Флаг `Host` заставляет сервер отвечать на разные виртуальные хосты. Находит `dev.target.com`, `admin.target.com` и т.д., даже если они не в DNS.

### 8. Несколько словарей одновременно

```bash
ffuf -u http://target.com/FUZZ1/FUZZ2 \
     -w wordlist1:FUZZ1 \
     -w wordlist2:FUZZ2
```

Заполнители `FUZZ1` и `FUZZ2` берут слова из **разных** словарей. Полезно для перебора пар «директория + файл».

### 9. Рекурсивное сканирование

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -recursion -recursion-depth 3
```

Если найден `/admin/` — ffuf зайдёт внутрь и просканирует `/admin/FUZZ`. Глубина — 3 уровня.

### 10. Автокалибровка фильтров

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -ac
```

> [!info] Зачем это нужно
> Сервер может отвечать **одинаково** на все запросы: 200 OK, размер 1234 байта. Без фильтра ты утонешь в тысяче ложных результатов. `-ac` (autocalibration) сам определит «шум» и отфильтрует его.

### 11. Через прокси (Burp)

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -proxy http://127.0.0.1:8080
```

Пропускает весь трафик через Burp Suite. Ты видишь запросы, можешь их модифицировать.

### 12. Сохранение в JSON

```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.json -of json
```

Для автоматизации и последующего анализа через `jq` или Python.

---

## 🏴 IV. ПРИМЕНЕНИЕ В ПЕНТЕСТЕ / CTF

### Быстрая разведка веб-приложения

```bash
# 1. Стандартная проверка директорий
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404 -c

# 2. Найденные поддомены
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.target.com"

# 3. Параметры в GET
ffuf -u "http://target.com/page?FUZZ=test" -w params.txt -fs 0

# 4. Логины через форму
ffuf -u http://target.com/login -X POST -d "user=FUZZ&pass=admin" -w users.txt -fc 401
```

### Где брать словари

| Путь | Что внутри |
|---|---|
| `/usr/share/wordlists/dirb/common.txt` | Базовые директории |
| `/usr/share/seclists/Discovery/Web-Content/` | Директории, файлы (SecLists) |
| `/usr/share/seclists/Discovery/DNS/` | Поддомены |
| `/usr/share/seclists/Passwords/` | Пароли |
| `/usr/share/wordlists/rockyou.txt` | Классика (14 млн паролей) |

> [!tip] Совет
> SecLists (`/usr/share/seclists/`) — золотой стандарт. Если нет:
> ```bash
> sudo apt install seclists
> ```

---

## 🛡️ V. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1595.003** — Wordlist Scanning
> - **T1190** — Exploit Public-Facing Application
> - **T1110** — Brute Force

### Что видно защитнику

| Источник | Что искать |
|---|---|
| **Apache/Nginx access.log** | Множественные 404 от одного IP |
| **Wazuh Rule 31101** | Множественные ошибки веб-сервера |
| **Wazuh Rule 31103** | Common web attack (SQLi, XSS) |
| **Wazuh Rule 31151** | SQL-инъекция |
| **Suricata** | Аномальное число HTTP-запросов |
| **Zeek** | HTTP-аномалии, сканирование |

### Признаки фаззинга в логах

```
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET /admin HTTP/1.1" 404 198 "-" "Fuzz Faster U Fool v2.1.0"
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET /backup HTTP/1.1" 404 198 "-" "Fuzz Faster U Fool v2.1.0"
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET /config.php HTTP/1.1" 200 4523 "-" "Fuzz Faster U Fool v2.1.0"
```

> [!danger] Красные флаги
> - Дефолтный **User-Agent** `Fuzz Faster U Fool`.
> - **50–100+ запросов в секунду** с одного IP.
> - **Много 404 подряд**, потом 200.
> - Обращения к `.git/`, `.env`, `backup.zip`, `wp-admin`.

### Практика в лабе

1. С Kali:
   ```bash
   ffuf -u http://192.168.56.20/FUZZ -w /usr/share/wordlists/dirb/common.txt -c
   ```
2. В Wazuh → **Security events** → фильтр по `rule.id:31101`.
3. Проверь: сколько запросов, какой User-Agent, какие URL.

> [!info] Как защититься
> - **Rate limiting** на веб-сервере (nginx: `limit_req`).
> - **WAF** (ModSecurity, Cloudflare).
> - **Fail2ban** — бан IP после N запросов с 404.
> - **Скрытие версий** и удаление стандартных путей.

---

## 🩺 VI. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| Слишком много ложных результатов | Добавь `-ac` или `-fs <размер>` |
| Медленно работает | Увеличь `-t 100`, но не выше 200 (риск блокировки) |
| Сервер блокирует | Уменьши `-t`, добавь `-p 0.5` (задержка) |
| Не вижу результат | Убери `-fc 404` и посмотри сырой вывод |
| Нужно остановить — `Ctrl+C`, потом `ffuf -resume` | Продолжить с места остановки |

> [!warning] Осторожно
> **Не ставь `-t 500+`** на реальных целях — сервер упадёт или тебя заблокируют. Начинай с `-t 40` (по умолчанию).

---

## 🔗 VII. СВЯЗИ

- **Инструменты:** [[curl]], [[nikto]], [[wapiti]], [[httpx]], [[gobuster]]
- **Техники:** [[T1595.003]], [[T1190]], [[T1110]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[Suricata]]

---

## 📚 VIII. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] Сравнение с `gobuster` и `feroxbuster`.
- [ ] Использование SecLists для разных задач.
- [ ] Фаззинг JSON-параметров (`-d '{"key":"FUZZ"}'`).
- [ ] Фаззинг заголовков (`-H "X-Forwarded-For: FUZZ"`).
- [ ] Автоматизация с `jq` для парсинга JSON-вывода.

---

## 📝 IX. ШПАРГАЛКА

```bash
# Директории
ffuf -u http://target/FUZZ -w wordlist.txt -fc 404 -c

# Файлы с расширениями
ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.html,.bak

# Поддомены
ffuf -u http://FUZZ.target -w subdomains.txt -H "Host: FUZZ.target"

# POST-брутфорс
ffuf -u http://target/login -X POST -d "user=admin&pass=FUZZ" -w pass.txt -fc 401

# С автокалибровкой
ffuf -u http://target/FUZZ -w wordlist.txt -ac

# В JSON
ffuf -u http://target/FUZZ -w wordlist.txt -o results.json -of json
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *ffuf — это молот. Стучишь по URL тысячей слов — находишь дверь. Но помни: молот слышно в логах.*