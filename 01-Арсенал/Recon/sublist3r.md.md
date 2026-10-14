---
type: tool
name: Sublist3r
category: Recon / OSINT / Subdomain Enumeration
platform: Linux, Windows, macOS
language: Python
status: studying
tags: [arsenal, recon, osint, subdomains, passive, red-team]
mitre:
  - T1590.001  # Gather Victim Network Information: Domain Properties
  - T1596.001  # Search Open Technical Databases: DNS/Passive DNS
  - T1596.005  # Search Open Technical Databases: Scan Databases
  - T1595.003  # Active Scanning: Wordlist Scanning
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Sublist3r
> **Категория:** Recon / OSINT / Subdomain Enumeration
> **Платформа:** Linux, Windows, macOS
> **Язык:** Python
> **Назначение:** Перечисление поддоменов через OSINT + брутфорс
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Sublist3r** — классический инструмент для **перечисления поддоменов** через **OSINT** + **bruteforce**. Работает на Python. Используется пентестерами, bug bounty-хантерами и OSINT-специалистами.

**Ключевое отличие от [[subfinder]]:**
- Sublist3r использует **поисковые системы** (Google, Bing, Yahoo, Baidu) — Subfinder — больше API-базы.
- Sublist3r имеет встроенный **брутфорс** через `subbrute`.
- Sublist3r — **старее**, но всё ещё актуален для комплексного подхода.

> [!tip] Когда использовать
> - **Subfinder** — быстрый пассивный сбор.
> - **Sublist3r** — когда нужны поисковые системы + брутфорс в одном инструменте.
> - **Amass** — если нужен максимально глубокий анализ (долго).

---

## 🎯 II. ИСТОЧНИКИ

### Поисковые системы
Google, Yahoo, Bing, Baidu, Ask, Yandex.

### Сервисы
- **VirusTotal** — данные по доменам.
- **Netcraft** — DNS-история.
- **ThreatCrowd** — OSINT по угрозам.
- **DNSdumpster** — DNS-разведка.
- **crt.sh** — SSL-сертификаты (золото для поддоменов).
- **Passive DNS** — история DNS.
- **Reverse DNS** — обратный DNS.

### Bruteforce (`subbrute`)
Перебор по словарю для поиска **неиндексированных** поддоменов.

---

## 🚀 III. УСТАНОВКА

### Kali Linux

```bash
sudo apt install sublist3r
```

### Из GitHub (если apt-версия старая)

```bash
git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r
sudo pip install -r requirements.txt
```

### Проверка

```bash
sublist3r -h
```

> [!warning] Python 2 vs 3
> Sublist3r изначально написан под **Python 2**. На Kali 2024+ обычно уже стоит адаптированная версия. Если ошибки — используй [[subfinder]] или [amass](https://github.com/owasp-amass/amass).

---

## 🎯 IV. ОСНОВНЫЕ ФЛАГИ

| Флаг | Что делает |
|---|---|
| `-d <домен>` | Целевой домен |
| `-e <источники>` | Конкретные источники (через запятую) |
| `-v` | Verbose (подробный вывод) |
| `-o <файл>` | Сохранить в файл |
| `-b` | Включить bruteforce (subbrute) |
| `-t <N>` | Потоков для bruteforce (по умолч. 10) |
| `-p <порты>` | Сканировать найденные поддомены на порты |
| `-n` | Не использовать поисковые системы (только DNS) |

---

## 🎯 V. ОСНОВНЫЕ СЦЕНАРИИ

### 1. Базовый поиск

```bash
sublist3r -d offsec.com
```

Найдёт поддомены из всех источников по умолчанию.

### 2. Конкретные источники

```bash
sublist3r -d offsec.com -e bing,yahoo,baidu,ask,netcraft,crtsh
```

Полезно, когда:
- Google блокирует запросы (CAPTCHA).
- Хочешь ускорить — исключить медленные источники.

### 3. Verbose + файл

```bash
sublist3r -d offsec.com -v -o subs.txt
```

`-v` — подробный вывод в реальном времени.
`-o` — сохранить результат.

### 4. С брутфорсом

```bash
sublist3r -d offsec.com -b -t 40
```

`-b` — включает `subbrute` (перебор по словарю).
`-t 40` — 40 потоков (ускорение).

> [!warning] Брутфорс — не пассивный
> В отличие от OSINT-фазы, брутфорс **касается** DNS-сервера цели. Это **видно** в логах DNS. Используй только на своей лабе или с разрешения.

### 5. Сканирование портов на найденных поддоменах

```bash
sublist3r -d offsec.com -p 80,443
```

Sublist3r автоматически проверит найденные поддомены на открытые порты.

### 6. Только DNS (без поисковиков)

```bash
sublist3r -d offsec.com -n
```

---

## 🔗 VI. КОНВЕЙЕР РАЗВЕДКИ

### Комбинированный сбор

```bash
# 1. Sublist3r
sublist3r -d target.com -o subs1.txt

# 2. Subfinder
subfinder -d target.com -silent -o subs2.txt

# 3. Assetfinder
assetfinder --subs-only target.com > subs3.txt

# 4. Объединяем + убираем дубли
cat subs1.txt subs2.txt subs3.txt | sort -u > all-subs.txt

# 5. Проверяем живые
httpx -silent -l all-subs.txt -o live.txt
```

> [!tip] Правило Легиона
> **Один инструмент = один угол зрения.** Собирай с нескольких источников. Пересечение = настоящие цели.

### Сравнение с [[subfinder]]

| | Sublist3r | Subfinder |
|---|---|---|
| Язык | Python | Go |
| Скорость | Медленнее | Быстрее |
| Поисковики | ✅ Google, Bing, Yahoo | ❌ |
| API-базы | Немного | 30+ источников |
| Bruteforce | ✅ (встроен) | ❌ (нужен dnsx) |
| Актуальность | Старее | Активнее |
| Лучше для | Полный OSINT-сбор | Быстрый пассив |

---

## 🛡️ VII. ДЕТЕКТ В WAZUH

> [!info] Особенность Sublist3r
> **Пассивная фаза (OSINT)** — не касается цели. Wazuh **не увидит**.
> **Активная фаза (`-b`, `-p`)** — касается цели. Wazuh **увидит** DNS-брутфорс и сканирование портов.

### Что видит защитник

| Источник | Что искать |
|---|---|
| **DNS-сервер** | Массовые NXDOMAIN-ответы (брутфорс) |
| **Firewall** | Много запросов к разным поддоменам |
| **Suricata** | DNS-аномалии, scan-сигнатуры |
| **Wazuh Rule 5104** | Port scan detected |
| **Wazuh Rule 5105** | Multiple ports scanned |
| **Apache/Nginx** | Обращения к root-путям разных поддоменов |
| **CDN / WAF** | Массовые запросы с одного IP |

### Признак брутфорса в DNS-логах

```
Sep 29 10:23:45 named[1234]: client 192.168.56.107#54321: query: admin.target.com IN A
Sep 29 10:23:45 named[1234]: client 192.168.56.107#54322: query: dev.target.com IN A
Sep 29 10:23:45 named[1234]: client 192.168.56.107#54323: query: test.target.com IN A
Sep 29 10:23:46 named[1234]: client 192.168.56.107#54324: query: backup.target.com IN A
```

> [!danger] Красные флаги
> - **Тысячи NXDOMAIN** за короткое время.
> - **Последовательные запросы** к поддоменам из словаря (`admin`, `dev`, `test`, `backup`).
> - **Один IP** — источник брутфорса.
> - **Регулярные интервалы** — признак автоматизации.

### Практика в лабе

1. С Kali (для примера на своём домене/лабе):
   ```bash
   sublist3r -d example.com -b -v -o subs.txt
   ```
2. Смотри DNS-логи (если у тебя есть DNS-сервер в лабе).
3. Wazuh → **Security events** → фильтр `rule.id:5104` (если порты сканировались).

> [!info] Как защититься
> - **Wildcard DNS** — все поддомены ведут на один сервер.
> - **Catch-all certificate** — единый SSL.
> - **DNS rate limiting** — ограничение запросов с одного IP.
> - **DNS-логи + алерты** — Suricata/Zeek отслеживают аномалии.
> - **Honeypot-поддомены** — ловушка для сканеров.

---

## 🩺 VIII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| `sublist3r: command not found` | `sudo apt install sublist3r` |
| Ошибки Python 2/3 | Установи свежую версию из GitHub |
| Google выдаёт CAPTCHA | Используй `-e` с другими источниками |
| Медленно | Убери `-b` (bruteforce), используй только OSINT |
| Мало результатов | Комбинируй с [[subfinder]] |
| Не сохраняет в файл | Проверь права: `-o /tmp/subs.txt` |
| Bruteforce зависает | Уменьши `-t` до 20 |

### Проверка установки

```bash
sublist3r -h
sublist3r -v
```

---

## 🔗 IX. СВЯЗИ

- **Инструменты:** [[subfinder]], [[assetfinder]], [[amass]], [[httpx]], [[dnsx]], [[ffuf]], [[nmap]]
- **Техники:** [[T1590.001]], [[T1596.001]], [[T1596.005]], [[T1595.003]]
- **Лаба:** [[IP-Table]]
- **Жертвы:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]], [[Suricata]]

---

## 📚 X. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] **Amass** — самый мощный для subdomain enum.
- [ ] **Assetfinder** — лёгкая альтернатива.
- [ ] **dnsx** — активная DNS-проверка.
- [ ] **github-subdomains** — поддомены из GitHub.
- [ ] **Wildcard detection** — определение catch-all DNS.
- [ ] **Certificate Transparency** — crt.sh API вручную.

---

## 📝 XI. ШПАРГАЛКА

```bash
# Базовый
sublist3r -d target.com

# Конкретные источники
sublist3r -d target.com -e bing,yahoo,crtsh

# Verbose + файл
sublist3r -d target.com -v -o subs.txt

# С брутфорсом
sublist3r -d target.com -b -t 40

# Сканирование портов
sublist3r -d target.com -p 80,443

# Без поисковиков
sublist3r -d target.com -n

# Комбинированный сбор
sublist3r -d target.com -o subs1.txt
subfinder -d target.com -silent -o subs2.txt
cat subs1.txt subs2.txt | sort -u > all-subs.txt
httpx -silent -l all-subs.txt
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Sublist3r — старая школа. Но иногда старый клинок проходит там, где новый ломается.*