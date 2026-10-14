---
type: tool
name: httpx
category: Recon / HTTP Probing
platform: Linux, Windows, macOS
language: Go
status: studying
tags: [arsenal, recon, web, http, red-team]
mitre:
  - T1595.002  # Active Scanning: Vulnerability Scanning
  - T1595.003  # Active Scanning: Wordlist Scanning
  - T1592.002  # Gather Victim Host Information: Software
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** httpx (ProjectDiscovery)
> **Категория:** Recon / HTTP Probing
> **Платформа:** Linux, Windows, macOS
> **Язык:** Go (быстрый, многопоточный)
> **Назначение:** Быстрая проверка списка хостов — какие живы, что за сервис, какая технология
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**httpx** — это «пробник» для HTTP-хостов. Когда у тебя после разведки есть **1000 имён и IP**, ты не будешь открывать их браузером вручную. httpx пройдёт по всем, определит:
- Кто **жив** (отвечает ли на HTTP/HTTPS).
- Какой **код ответа** (200, 301, 403).
- Какой **заголовок** `<title>`.
- Какие **технологии** используются (nginx, WordPress, Cloudflare).
- Куда **редиректит**.

**Идеальный компаньон:** [[subfinder]] → [[httpx]] → [[nuclei]].

> [!tip] Зачем нужен
> Ты нашёл 500 поддоменов через [[subfinder]]. Из них живых — 50. httpx отсеет мёртвые и покажет только реальные цели за **10 секунд**.

---

## 🚀 II. УСТАНОВКА

```bash
sudo apt update
sudo apt install golang-go
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

**Бинарник попадёт в:**
```
~/go/bin/httpx
```

> [!info] Алиас для удобства
> Добавь в `~/.bashrc` или `~/.zshrc`:
> ```bash
> export PATH="$PATH:$HOME/go/bin"
> ```
> Потом: `source ~/.bashrc`. Теперь можно писать просто `httpx`, а не `~/go/bin/httpx`.

---

## 🎯 III. ОСНОВНЫЕ ФЛАГИ

| Флаг | Сокращение | Что делает |
|---|---|---|
| `-u` | — | Один конкретный URL |
| `-l` | `-list` | Файл со списком целей (главный сценарий) |
| `-silent` | — | Тихий режим — только результат |
| `-sc` | `-status-code` | Показать код ответа (200, 301) |
| `-title` | — | Извлечь `<title>` страницы |
| `-td` | `-tech-detect` | Определить технологии (Wappalyzer) |
| `-cl` | — | Content-Length |
| `-server` | — | Заголовок `Server` |
| `-json` | — | Вывод в JSON |
| `-mc` | `-match-code` | Показать только указанные коды |
| `-fc` | `-filter-code` | Исключить коды |
| `-fr` | `-follow-redirects` | Следовать редиректам |
| `-t` | `-threads` | Потоки (200–500 обычно) |
| `-x` | — | Полноценный HTTP-метод (`-x GET`) |

---

## 🎯 IV. БАЗОВЫЕ СЦЕНАРИИ

### 1. Проверка одного URL

```bash
~/go/bin/httpx -silent -u https://example.com
```

**Что делает:**
- `httpx` — запускает инструмент.
- `-silent` — тихий режим, убирает баннер и статистику.
- `-u` — говорит «будет один конкретный URL».
- `https://example.com` — цель.

**Результат:** одна строка — `https://example.com [200 OK]`.

### 2. Проверка списка хостов (главный сценарий)

```bash
httpx -silent -l hosts.txt
```

`-l` (или `-list`) — файл со списком целей, по одной на строку.

### 3. С кодом ответа и заголовком

```bash
httpx -silent -l hosts.txt -sc -title
```

**Результат:**
```
https://example.com [200] [Example Domain]
https://admin.example.com [403] [Admin Panel]
https://api.example.com [200] [API Docs]
```

### 4. Определение технологий

```bash
httpx -silent -l hosts.txt -td -server
```

**Результат:**
```
https://example.com [nginx] [PHP, jQuery]
https://admin.example.com [cloudflare] [WordPress 6.2]
```

### 5. Только живые (фильтр кодов)

```bash
httpx -silent -l hosts.txt -mc 200,301,302,403
```

Показать только эти коды. Все остальные (404, 500) — скрыть.

### 6. Исключить коды

```bash
httpx -silent -l hosts.txt -fc 404,500
```

Убрать 404 и 500.

### 7. С редиректами

```bash
httpx -silent -l hosts.txt -fr -sc
```

Следовать за редиректами и показать финальный код.

### 8. Список целей + JSON для автоматизации

```bash
httpx -silent -l hosts.txt -json -o results.json
```

Пригодится для последующего парсинга через `jq` или Python.

### 9. Много потоков (быстро)

```bash
httpx -silent -l hosts.txt -t 200
```

200 потоков. Быстрее, но осторожнее — не превышай разумные пределы на реальных целях.

### 10. Полноценный GET-запрос

```bash
httpx -silent -l hosts.txt -x GET
```

По умолчанию httpx делает лёгкий запрос. `-x GET` — полноценный GET с телом ответа.

---

## 🔗 V. КОНВЕЙЕР РАЗВЕДКИ

Классическая цепочка:

```bash
# 1. Найти поддомены
subfinder -d target.com -silent > subs.txt

# 2. Отсеять мёртвые, оставить живые
httpx -silent -l subs.txt -o live.txt

# 3. Собрать технологии и заголовки
httpx -silent -l live.txt -td -title -sc -o enriched.txt

# 4. Прогнать через сканер уязвимостей
nuclei -l live.txt -o vulns.txt
```

> [!tip] Правило Легиона
> Не работай с **одним** инструментом. Работай с **цепочкой**. Один инструмент находит вход, второй подтверждает, третий — атакует.

---

## 🛡️ VI. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1595.002** — Vulnerability Scanning
> - **T1592.002** — Gather Victim Host Information

### Что видно защитнику

| Источник | Что искать |
|---|---|
| **Apache/Nginx access.log** | Массовые короткие запросы к `/` |
| **Wazuh Rule 31101** | Множественные ошибки веб-сервера |
| **Wazuh Rule 31103** | Web scan |
| **Suricata** | Аномальное число HTTP-соединений с одного IP |
| **CDN / WAF** | Параллельные запросы к разным поддоменам |

### Признак httpx в логах

```
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET / HTTP/1.1" 200 1234 "-" "httpx/1.3.7"
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET / HTTP/1.1" 403 198 "-" "httpx/1.3.7"
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET / HTTP/1.1" 301 0 "-" "httpx/1.3.7"
```

> [!danger] Красные флаги
> - **User-Agent** `httpx/*` в проде (нормальные браузеры так не пишут).
> - **Одинаковые запросы** к корню разных поддоменов.
> - **Много запросов** с одного IP за секунды.
> - **Ровные интервалы** между запросами.

### Практика в лабе

1. С Kali:
   ```bash
   echo "http://192.168.56.20" > targets.txt
   echo "http://192.168.56.30" >> targets.txt
   httpx -silent -l targets.txt -sc -title -td
   ```
2. В Wazuh → **Security events** → фильтр `rule.id:31101`.
3. Проверь: User-Agent, частота, целевые URL.

> [!info] Как защититься
> - **WAF** (ModSecurity / Cloudflare) — блокировка по User-Agent.
> - **Rate limiting** — ограничение запросов с одного IP.
> - **Fail2ban** — бан по количеству 4xx-кодов за минуту.
> - **Обфускация** реальных сервисов (не выдавать заголовок `Server`).

---

## 🩺 VII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `httpx: command not found` | Добавь `~/go/bin` в `$PATH` |
| Медленно работает | `-t 200`, но не выше — рискуешь баном |
| Ошибка `no such file` | Проверь путь к списку (`-l hosts.txt`) |
| Ложные результаты | Используй `-mc` вместо `-fc` |
| Не видит HTTPS | Убедись, что цель слушает 443 |
| Много мусора в выводе | Добавь `-silent` и `-mc 200,301,302` |

### Проверка установки

```bash
httpx -version
httpx -h
```

---

## 🔗 VIII. СВЯЗИ

- **Инструменты:** [[subfinder]], [[sublist3r]], [[nmap]], [[nuclei]], [[ffuf]], [[whatweb]]
- **Техники:** [[T1595.002]], [[T1595.003]], [[T1592.002]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[Suricata]]

---

## 📚 IX. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] `-x POST` — отправка POST-запросов.
- [ ] `-H` — кастомные заголовки.
- [ ] `-path` — проверка конкретного пути (`.git/config`, `.env`).
- [ ] `-favicon` — хэш иконки для fingerprinting.
- [ ] Связка с [[nuclei]] для авто-поиска уязвимостей.
- [ ] Флаги `-asn`, `-cdn` — определение хостинга.

---

## 📝 X. ШПАРГАЛКА

```bash
# Один URL
httpx -silent -u https://example.com

# Список хостов
httpx -silent -l hosts.txt

# С кодом и заголовком
httpx -silent -l hosts.txt -sc -title

# С технологиями
httpx -silent -l hosts.txt -td -server

# Только живые
httpx -silent -l hosts.txt -mc 200,301,302

# Исключить 404
httpx -silent -l hosts.txt -fc 404

# В JSON
httpx -silent -l hosts.txt -json -o results.json

# С редиректами
httpx -silent -l hosts.txt -fr
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *httpx — это сито. Тысяча имён входит, десяток живых выходит. Дальше — работа других инструментов.*