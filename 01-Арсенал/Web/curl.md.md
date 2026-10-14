---
type: tool
name: curl
category: Web / HTTP
platform: Linux, Windows, macOS
language: C
status: studying
tags: [arsenal, web, http, api, red-team, blue-team]
mitre:
  - T1071.001  # Application Layer Protocol: Web Protocols
  - T1190      # Exploit Public-Facing Application
  - T1595.002  # Active Scanning: Vulnerability Scanning
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** curl (Client URL)
> **Категория:** Web / HTTP-клиент
> **Платформа:** Linux, Windows, macOS
> **Назначение:** Отправка HTTP-запросов, скачивание файлов, тестирование API
> **Статус:** 🟢 Базовый уровень освоен
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**curl** — консольная утилита и библиотека для передачи данных по сети. Поддерживает **30+ протоколов**: HTTP, HTTPS, FTP, SFTP, SCP, SMTP, IMAP и другие.

**Применение в ИБ:**
- Тестирование API и веб-приложений.
- Ручная проверка уязвимостей (SQLi, XSS, IDOR).
- Отправка кастомных запросов с подменой заголовков.
- Скачивание полезной нагрузки (payload) на цель.
- Автоматизация в скриптах (bash, python).

---

## 🚀 II. БАЗОВЫЙ СИНТАКСИС

```bash
curl [опции] [URL]
```

### Основные флаги

| Флаг | Что делает |
|---|---|
| `-X METHOD` | Указать HTTP-метод (GET, POST, PUT, DELETE) |
| `-d "data"` | Отправить данные (POST по умолчанию) |
| `-H "Header: value"` | Добавить заголовок |
| `-u user:pass` | Basic-аутентификация |
| `-o file.txt` | Сохранить ответ в файл |
| `-O` | Сохранить с именем из URL |
| `-i` | Показать заголовки ответа |
| `-I` | Только заголовки (HEAD-запрос) |
| `-v` | Подробный вывод (verbose) |
| `-s` | Тихий режим (без прогресса) |
| `-L` | Следовать редиректам |
| `-k` | Игнорировать SSL-сертификат |
| `--compressed` | Запросить сжатие |
| `-x proxy:port` | Использовать прокси |

---

## 🎯 III. ОСНОВНЫЕ СЦЕНАРИИ

### 1. GET-запрос

```bash
# Простой GET
curl https://example.com

# GET с параметрами
curl "http://site.com/page?param=value&flag=true"

# GET с параметрами через -G
curl -G -d "param=value" -d "flag=true" http://site.com/page
```

### 2. POST-запрос

```bash
# Форма (x-www-form-urlencoded)
curl -X POST -d "username=admin&password=123" http://site.com/login

# JSON
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"key":"value"}' \
  http://site.com/api

# Из файла
curl -X POST -d "@data.json" \
  -H "Content-Type: application/json" \
  http://site.com/api
```

### 3. Работа с заголовками

```bash
# Один заголовок
curl -H "User-Agent: Mozilla/5.0" [URL]

# Несколько заголовков
curl -H "Content-Type: application/json" \
     -H "X-API-Key: mykey" \
     -H "Referer: http://google.com" \
     [URL]

# Только заголовки ответа
curl -I http://site.com

# Заголовки + тело
curl -i http://site.com
```

### 4. Аутентификация

```bash
# Basic Auth
curl -u username:password [URL]

# Bearer Token
curl -H "Authorization: Bearer TOKEN" [URL]

# Через куки
curl -b "name=value" [URL]              # отправить куки
curl -c cookies.txt [URL]                # сохранить куки в файл
curl -b cookies.txt [URL]                # использовать куки из файла
```

### 5. Работа с файлами

```bash
# Скачать файл
curl -o local_name.txt http://site.com/file.txt

# С исходным именем
curl -O http://site.com/file.txt

# Несколько файлов
curl -O http://site.com/file1.txt -O http://site.com/file2.txt

# Продолжить прерванную загрузку
curl -C - -O http://site.com/largefile.zip

# Загрузить файл на сервер (PUT)
curl -X PUT -T file.txt [URL]

# Multipart-форма
curl -F "file=@photo.jpg" -F "name=John" [URL]
```

---

## 🎯 IV. ПРОДВИНУТЫЕ СЦЕНАРИИ

### SSL / HTTPS

```bash
# Игнорировать проверку сертификата
curl -k https://self-signed.badssl.com

# Свой CA-сертификат
curl --cacert cert.pem [URL]

# Клиентский сертификат
curl --cert client.pem --key key.pem [URL]

# Указать TLS-версию
curl --tlsv1.2 [URL]
curl --tlsv1.3 [URL]
```

### Сетевые настройки

```bash
# Таймауты
curl --connect-timeout 10 [URL]
curl -m 30 [URL]                  # максимум 30 сек

# Прокси
curl -x http://proxy:8080 [URL]
curl --proxy-user user:pass -x http://proxy:8080 [URL]

# Только IPv4 / IPv6
curl -4 [URL]
curl -6 [URL]

# Ограничение скорости
curl --limit-rate 100K [URL]

# Следовать редиректам
curl -L [URL]
curl --max-redirs 5 [URL]
```

### Полный HTTP-запрос (комбинированный)

```bash
curl -v -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{"data":"value"}' \
  http://target.com/api/endpoint
```

---

## 🏴 V. ПРИМЕНЕНИЕ В CTF / ПЕНТЕСТЕ

### Быстрая проверка веб-приложения

```bash
# Проверка доступности
curl -I http://target.com

# Заголовок страницы
curl -s http://target.com | grep -i "<title>"

# Проверка robots.txt
curl http://target.com/robots.txt

# Проверка .git
curl -I http://target.com/.git/
```

### Ручной поиск уязвимостей

```bash
# SQLi-проверка (базовая)
curl "http://target.com/page?id=1'"
curl "http://target.com/page?id=1 OR 1=1--"

# XSS-проверка
curl -X POST -d "comment=<script>alert(1)</script>" http://target.com/comment

# IDOR
curl -H "Cookie: session=abc" http://target.com/user/1
curl -H "Cookie: session=abc" http://target.com/user/2

# Открытые админки
curl -u admin:admin http://target.com/admin
curl -u admin:password http://target.com/admin
```

### Брутфорс через curl (bash-цикл)

```bash
# Перебор паролей Basic Auth
for pass in $(cat passwords.txt); do
  result=$(curl -s -o /dev/null -w "%{http_code}" -u admin:$pass http://target.com)
  if [ "$result" == "200" ]; then
    echo "Пароль найден: $pass"
    break
  fi
done
```

> [!warning] Ограничение
> curl — не инструмент для брутфорса. Для этого используй [[hydra]] или [[ffuf]]. curl-циклы годятся только для быстрой проверки 10–50 паролей.

### Кастомные заголовки (CTF-задачи)

```bash
# Запрос с кастомным заголовком
curl -H "Magic: givemeaflag" http://target.com:8090/http2

# POST с JSON-телом
curl -c cookies.txt -X POST http://target.com:8090/http2 \
     -H "Content-Type: application/json" \
     -d '{"login":"admin", "pass":"P@ssw0rd"}'
```

---

## 🛡️ VI. ДЕТЕКТ В WAZUH

> [!info] Особенность curl
> curl — **легитимный инструмент**. Он используется в каждом втором скрипте автоматизации. Поэтому детект строится не на факте использования curl, а на **поведении**.

### Что искать в логах

| Источник | Что искать |
|---|---|
| **Apache/Nginx access.log** | Массовые 404, 401, 403; необычные User-Agent |
| **Wazuh Rule 31101** | Множественные ошибки веб-сервера |
| **Wazuh Rule 31103** | Сканирование директорий |
| **Wazuh Rule 31151** | SQL-инъекция в запросе |
| **Wazuh Rule 31161** | XSS-атака |
| **Suricata / Zeek** | Аномальный User-Agent `curl/*` в проде |

### Как выглядит атака curl в логах Apache

```
192.168.56.107 - - [29/Sep/2026:10:23:45 +0300] "GET /admin' OR 1=1-- HTTP/1.1" 500 1234 "-" "curl/7.88.1"
192.168.56.107 - - [29/Sep/2026:10:23:46 +0300] "GET /.git/config HTTP/1.1" 404 198 "-" "curl/7.88.1"
```

> [!danger] Красные флаги
> - User-Agent `curl/*` в production-логах.
> - Запросы с SQL-синтаксисом в параметрах.
> - Много запросов к `.git`, `.env`, `backup.zip`.
> - Множественные 401 за короткое время.

### Практика в лабе

1. С Kali: `curl -I http://192.168.56.20/.git/`
2. Смотри в Wazuh → **Security events** → фильтр `rule.id:31101`.
3. Разбери алерт: кто, откуда, что запросил.

---

## 🩺 VII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `curl: (60) SSL certificate problem` | Добавь `-k` или `--cacert` |
| `curl: (7) Failed to connect` | Проверь, что цель жива: `ping` |
| `curl: (28) Operation timed out` | Увеличь `--connect-timeout` или `-m` |
| Медленно работает | Добавь `-s` (без прогресса), `--compressed` |
| Кракозябры в выводе | `curl ... \| iconv -f utf-8` или `file --mime` |
| Нужно видеть реальный запрос | `curl -v [URL]` |

---

## 🔗 VIII. СВЯЗИ

- **Инструменты:** [[wget]], [[ffuf]], [[nikto]], [[burp-suite]]
- **Техники:** [[T1190]], [[T1595.002]], [[T1071.001]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]], [[Windows 10]]
- **Защита:** [[Wazuh]]

---

## 📚 IX. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] Разница между `-d` и `--data-binary` (важно для загрузки файлов).
- [ ] `curl --http2` и `--http3` (современные протоколы).
- [ ] `curl --resolve` (тестирование виртуальных хостов).
- [ ] Парсинг вывода через `jq` для JSON-API.
- [ ] Интеграция с [[ffuf]] (передача запросов).

---

## 📝 X. ШПАРГАЛКА ДЛЯ СКОРОГО ДОСТУПА

```bash
# Быстрые команды
curl -I URL                             # заголовки
curl -s URL | jq                        # JSON + парсинг
curl -o out.txt URL                     # сохранить
curl -X POST -d "a=1&b=2" URL           # форма
curl -X POST -H "Content-Type: application/json" -d '{"k":"v"}' URL
curl -u user:pass URL                   # Basic Auth
curl -b cookies.txt URL                 # с куками
curl -L URL                             # с редиректами
curl -k URL                             # игнор SSL
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *curl — это скальпель. Один запрос может вскрыть периметр. Или оставить след в логах.*