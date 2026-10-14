---
type: tool
name: Subfinder
category: Recon / OSINT / Subdomain Enumeration
platform: Linux, Windows, macOS
language: Go
status: studying
tags: [arsenal, recon, osint, subdomains, passive, red-team]
mitre:
  - T1590.001  # Gather Victim Network Information: Domain Properties
  - T1596.001  # Search Open Technical Databases: DNS/Passive DNS
  - T1596.002  # Search Open Technical Databases: WHOIS
  - T1596.005  # Search Open Technical Databases: Scan Databases
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Subfinder (ProjectDiscovery)
> **Категория:** Recon / OSINT / Subdomain Enumeration
> **Платформа:** Linux, Windows, macOS
> **Язык:** Go
> **Назначение:** **Пассивное** перечисление поддоменов из публичных источников
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Subfinder** — мощный инструмент для **пассивного** перечисления поддоменов. Он собирает валидные поддомены целевого домена из **публичных источников** — баз данных, сертификатов, поисковиков — **без прямых запросов к серверу жертвы**.

> [!success] Почему это важно
> **Пассивная разведка** = stealthy. Ты не касаешься цели. Цель не видит тебя в логах. Никаких запросов к её DNS/серверу. Только сбор публичной информации.

**Источники (30+):**
- **crt.sh** — сертификаты SSL (золото для поддоменов).
- **Censys, Shodan, BinaryEdge** — базы сканирований.
- **VirusTotal** — данные по доменам.
- **Chaos** — база ProjectDiscovery.
- **SecurityTrails** — пассивный DNS.
- **GitHub** — упоминания в репозиториях.
- **Wayback Machine** — история домена.

> [!tip] Разница с активным сканированием
> [[sublist3r]] — тоже пассивный, но менее точный.
> **Subfinder** — быстрее, качественнее, больше источников.
> **Активный DNS-брутфорс** — `dnsx`, `gobuster dns` — **заметен** в логах DNS-сервера.

---

## 🚀 II. УСТАНОВКА

### Вариант 1: Go

```bash
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
```

Бинарник попадёт в `~/go/bin/subfinder`.

### Вариант 2: apt (Kali)

```bash
sudo apt install subfinder
```

### Вариант 3: Docker

```bash
docker run -it --rm projectdiscovery/subfinder -d target.com
```

> [!info] API-ключи (опционально)
> Subfinder работает **без ключей**, но с ними — находит в разы больше.
> Конфиг: `~/.config/subfinder/provider-config.yaml`
> Заполни ключи: `virustotal`, `shodan`, `censys`, `chaos`, `github` и т.д.
> Все ключи бесплатны на соответствующих платформах.

---

## 🎯 III. БАЗОВЫЙ СИНТАКСИС

```bash
subfinder [опции] -d <домен>
```

### Основные флаги

| Флаг | Что делает |
|---|---|
| `-d <домен>` | Целевой домен |
| `-dL <файл>` | Список доменов из файла |
| `-all` | Использовать **все** источники (включая медленные) |
| `-silent` | Тихий режим (только поддомены, без баннера) |
| `-active` | Использовать активные источники тоже |
| `-o <файл>` | Сохранить результат в файл |
| `-oJ` | Вывод в JSON |
| `-es <источник>` | Исключить источник |
| `-s <источник>` | Использовать только указанные источники |
| `-t <N>` | Потоки |
| `-timeout <сек>` | Таймаут |
| `-v` | Подробный вывод |

---

## 🎯 IV. ОСНОВНЫЕ СЦЕНАРИИ

### 1. Базовый поиск

```bash
subfinder -d offsec.com
```

Выводит список поддоменов:
```
www.offsec.com
api.offsec.com
dev.offsec.com
mail.offsec.com
...
```

### 2. Тихий режим (только результат)

```bash
subfinder -d offsec.com -silent
```

Идеально для конвейера с другими инструментами.

### 3. Все источники (максимум)

```bash
subfinder -d offsec.com -all
```

Медленнее, но находит больше.

### 4. Из файла (много доменов)

```bash
subfinder -dL domains.txt -silent -o all-subs.txt
```

`domains.txt`:
```
target1.com
target2.com
target3.com
```

### 5. Активные источники

```bash
subfinder -d example.com -active
```

Использует **активные** методы тоже (DNS-проверки). Быстрее результат, но **немного заметнее**.

### 6. JSON-вывод

```bash
subfinder -d offsec.com -oJ
```

Для парсинга через `jq` или Python.

### 7. Исключить источник

```bash
subfinder -d offsec.com -es github
```

### 8. Только определённые источники

```bash
subfinder -d offsec.com -s crtsh,virustotal,shodan
```

### 9. Сохранить в файл

```bash
subfinder -d www.nacbharat.com -o subs.txt
```

---

## 🔗 V. КОНВЕЙЕР РАЗВЕДКИ

### Классическая цепочка

```bash
# 1. Пассивный сбор поддоменов
subfinder -d target.com -silent > subs.txt

# 2. Проверка живых (какие отвечают)
httpx -silent -l subs.txt -o live.txt

# 3. Технологии + заголовки
httpx -silent -l live.txt -td -title -sc -o enriched.txt

# 4. Сканер уязвимостей
nuclei -l live.txt -o vulns.txt
```

### С альтернативными инструментами

```bash
subfinder -d target.com -silent | \
  assetfinder --subs-only target.com | \
  sort -u > all-subs.txt
```

`sort -u` — объединяет результаты обоих инструментов и убирает дубли.

---

## 🛡️ VI. ДЕТЕКТ В WAZUH

> [!info] Особенность Subfinder
> Subfinder работает **пассивно** — он **не касается** цели. Поэтому Wazuh на стороне цели **НЕ УВИДИТ** активность Subfinder.
>
> Но есть косвенные признаки.

### Что можно поймать

| Источник | Что искать |
|---|---|
| **Твоя сеть** | Исходящие запросы к API crt.sh, VirusTotal, Shodan |
| **DNS-сервер цели** | Если источник использует активный DNS-брутфорс |
| **CDN / WAF** | Много запросов к разным поддоменам с одного IP |
| **Логи веб-сервера** | Массовое обращение к root-путям разных поддоменов |
| **Firewall (твой)** | Исходящие запросы на подозрительные домены |

> [!danger] MITRE
> - **T1590.001** — Domain Properties
> - **T1596.001** — DNS/Passive DNS
> - **T1596.005** — Scan Databases

### Как обнаружить на этапе активной фазы

После Subfinder ты **обычно** запускаешь:
```bash
httpx -l subs.txt -silent
```

**Вот тут** ты касаешься сервера. И это видно:

```
192.168.56.107 - - [29/Sep/2026:10:23:45] "GET / HTTP/1.1" 200 1234 "-" "httpx/1.3.7"
```

> [!info] Как защититься от активной фазы
> - **Wildcard DNS** — все поддомены ведут на один сервер.
> - **Catch-all certificate** — единый SSL на все поддомены.
> - **WAF** — блокировка по частоте запросов.
> - **Мониторинг DNS-запросов** — резкий рост запросов к разным поддоменам.
> - **Honeypot-поддомены** — `admin-backup.target.com` — ловушка.

### Практика в лабе

1. С Kali:
   ```bash
   subfinder -d example.com -silent
   ```
2. На своём DNS-сервере (если есть) или в Suricata:
   - Логи DNS-запросов.
   - Исходящие запросы к `crt.sh`, `dns.google`, `virustotal.com`.
3. Wazuh → **Security events** → фильтр по домену.

---

## 🩺 VII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `subfinder: command not found` | Добавь `~/go/bin` в `$PATH` |
| Мало результатов | Настрой API-ключи в `provider-config.yaml` |
| Медленно | Убери `-all`, используй только быстрые источники |
| Ошибки SSL | `-timeout 30` и убедись, что интернет работает |
| Не находит поддомены | Домен может не иметь публичных упоминаний — используй активный брутфорс (`dnsx`, `gobuster`) |
| Не работает GitHub-источник | Нужен GitHub токен в config |

### Проверка установки

```bash
subfinder -version
subfinder -h
```

### Настройка API-ключей

```bash
subfinder -ls                       # список источников
nano ~/.config/subfinder/provider-config.yaml
```

Пример содержимого:
```yaml
virustotal:
  - YOUR_API_KEY
shodan:
  - YOUR_API_KEY
censys:
  - YOUR_API_ID:YOUR_API_SECRET
github:
  - YOUR_TOKEN
```

---

## 🔗 VIII. СВЯЗИ

- **Инструменты:** [[sublist3r]], [[assetfinder]], [[amass]], [[httpx]], [[nmap]], [[ffuf]], [[nuclei]]
- **Техники:** [[T1590.001]], [[T1596.001]], [[T1596.005]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[Suricata]]

---

## 📚 IX. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] **Amass** — более мощный (но медленный) аналог.
- [ ] **Assetfinder** — лёгкая альтернатива.
- [ ] **dnsx** — активная DNS-проверка найденных поддоменов.
- [ ] **github-subdomains** — поиск поддоменов в GitHub.
- [ ] Настройка **API-ключей** во всех источниках.
- [ ] **Wildcard detection** — определение catch-all DNS.

---

## 📝 X. ШПАРГАЛКА

```bash
# Базовый поиск
subfinder -d target.com

# Тихий режим (для конвейера)
subfinder -d target.com -silent

# Все источники
subfinder -d target.com -all

# Из файла
subfinder -dL domains.txt -silent -o out.txt

# Активные источники
subfinder -d target.com -active

# JSON
subfinder -d target.com -oJ > subs.json

# Конкретные источники
subfinder -d target.com -s crtsh,virustotal

# Исключить источник
subfinder -d target.com -es github

# Конвейер
subfinder -d target.com -silent | httpx -silent
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Subfinder не стучится в двери. Он читает объявления. Тихий, незаметный, но именно с него начинается охота.*