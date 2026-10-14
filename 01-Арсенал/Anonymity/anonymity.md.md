---
type: tool
name: Anonymity (Tor, macchanger)
category: Anonymity / OPSEC
platform: Linux
status: studying
tags: [arsenal, anonymity, opsec, tor, mac, red-team]
mitre:
  - T1090.003  # Proxy: Multi-hop Proxy
  - T1036      # Masquerading
  - T1036.005  # Match Legitimate Name or Location
created: 2025-09-29
---

> [!info] 🛠️ ДОСЬЕ ИНСТРУМЕНТА
> **Название:** Tor Browser, macchanger
> **Категория:** Anonymity / OPSEC
> **Платформа:** Linux
> **Назначение:** Скрытие IP-адреса (Tor) и MAC-адреса (macchanger)
> **Статус:** 🟡 Базовый уровень
> **Лаба:** [[IP-Table]]

---

## 📖 I. ЗАЧЕМ ЭТО

**Анонимность** в ИБ — это не про «скрыться от ФБР». Это про **OPSEC**:
- Не светить **свой реальный IP** при разведке публичных ресурсов.
- Не палить **свой MAC** в чужой сети.
- Понимать, как работают механизмы сокрытия — чтобы **детектить** их (синяя сторона).

> [!warning] Юридически
> Tor **легален** в большинстве стран. macchanger — тоже. Но их использование для **незаконных** действий — статья. В лабе — только для обучения.

---

## 🧅 II. TOR BROWSER

### Что это
**Tor** — сеть из тысяч ретрансляторов, которая **шифрует и перенаправляет** твой трафик через 3+ узла. Каждый узел знает только предыдущий и следующий. Выходной узел видит конечный трафик (если нет HTTPS).

**Что даёт:**
- Скрытие **реального IP**.
- Доступ к **.onion** сайтам (скрытые сервисы).
- Обход **гео-блокировок**.

**Что НЕ даёт:**
- Не защищает от вирусов/фишинга.
- Не скрывает твоё **поведение**.
- Медленно.

### Установка на Linux

```bash
# Скачать Tor Browser Bundle
wget https://www.torproject.org/dist/torbrowser/15.0.3/tor-browser-linux-x86_64-15.0.3.tar.xz

# Распаковать
tar -xf tor-browser-linux-x86_64-15.0.3.tar.xz

# Перейти в папку
cd tor-browser
```

### Запуск

```bash
./start-tor-browser.desktop
```

> [!tip] Если не запускается
> Сделай исполняемым:
> ```bash
> chmod +x start-tor-browser.desktop
> ```

### Проверка работы

Открой в Tor Browser:
- `https://check.torproject.org/` — покажет, что ты через Tor.
- `https://whatismyipaddress.com/` — покажет IP выходного узла (не твой).

### Продвинутый режим

```bash
# Tor как прокси (для curl, nmap и т.д.)
sudo systemctl start tor
curl --socks5 127.0.0.1:9050 https://check.torproject.org/
```

> [!warning] Tor + Nmap
> **Не работает** для сканирования. Nmap через Tor — медленно и бесполезно. Tor — для HTTP-запросов, не для сканов.

---

## 🎭 III. MACCHANGER

### Что это
**macchanger** — утилита для смены **MAC-адреса** сетевого интерфейса. MAC — это «серийный номер» сетевой карты. Меняя его, ты не светишь реальный MAC в сети.

**Зачем:**
- Не палить свой MAC в чужом Wi-Fi.
- Обходить **MAC-фильтрацию** на роутерах.
- Обходить **DHCP-ограничения**.

### Установка

```bash
sudo apt install macchanger
```

> [!tip] Отключи автосмену MAC
> При установке Debian/Kali спросит: «Should macchanger run at startup?». Выбери **No** — иначе MAC будет меняться при каждой загрузке.

### Просмотр текущего MAC

```bash
macchanger -s eth0
```

### Смена MAC

| Команда | Что делает |
|---|---|
| `sudo macchanger -r eth0` | Случайный MAC |
| `sudo macchanger -A eth0` | MAC известного производителя (любой) |
| `sudo macchanger -m XX:XX:XX:XX:XX:XX eth0` | Свой MAC |
| `sudo macchanger -p eth0` | Вернуть оригинальный MAC |

### Твоя команда

```bash
sudo macchanger eth0 --random
```

> [!warning] Интерфейс должен быть выключен
> Если ошибка — сначала:
> ```bash
> sudo ip link set eth0 down
> sudo macchanger -r eth0
> sudo ip link set eth0 up
> ```

### Смена MAC на Wi-Fi

```bash
sudo ip link set wlan0 down
sudo macchanger -r wlan0
sudo ip link set wlan0 up
```

---

## 🛡️ IV. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1090.003** — Multi-hop Proxy (Tor)
> - **T1036** — Masquerading (macchanger)

### Tor — что видит защитник

| Источник | Что искать |
|---|---|
| **Firewall / Proxy** | Подключения к **Tor entry nodes** (списки в открытом доступе) |
| **Suricata** | Сигнатуры Tor-трафика (SSL/TLS fingerprint) |
| **Zeek** | Tor handshake (специфический паттерн) |
| **DNS** | Запросы к `*.onion`, `torproject.org` |
| **Wazuh Agent** | Запуск `tor` / `tor-browser` на хосте |
| **NetFlow** | Трафик на порты 9001, 9030, 9040, 9050 |

### macchanger — что видит защитник

| Источник | Что искать |
|---|---|
| **Switch / Router** | MAC-flapping (один порт, много MAC) |
| **DHCP-сервер** | Частые смены MAC с одного порта |
| **WIDS / WIPS** | Аномальные MAC (OUI не совпадает с вендором) |
| **Wazuh Agent** | Запуск `macchanger` (`auditd`) |
| **Suricata** | ARP-аномалии |

> [!danger] Красные флаги
> - **OUI локально-администрируемый** (`02:00:00`, `06:00:00`, `0A:00:00`, `0E:00:00`) — явный признак рандома.
> - **Постоянная смена MAC** с одного порта.
> - **Подключения к Tor entry nodes** — по спискам `check.torproject.org` / Tor Metrics.
> - **DNS-запросы к .onion**.

### Практика в лабе

1. Проверь текущий MAC:
   ```bash
   macchanger -s eth0
   ```
2. Смени на случайный:
   ```bash
   sudo ip link set eth0 down
   sudo macchanger -r eth0
   sudo ip link set eth0 up
   ```
3. Смотри в Wazuh → **Security events** → фильтр по MAC-адресу.

> [!info] Как защититься
> - **802.1X** — авторизация устройств в сети (MAC-спуфинг бесполезен).
> - **Port Security** на коммутаторах — один MAC на порт.
> - **DHCP snooping** — привязка MAC к порту.
> - **Блокировка Tor** — firewall по списку exit nodes.
> - **Deep Packet Inspection** — детект Tor handshake.

---

## 🩺 V. ТРОУБЛШУТИНГ

| Проблема | Решение |
|---|---|
| Tor Browser не запускается | `chmod +x start-tor-browser.desktop` |
| Tor медленно | Норма — много узлов. Смени выходной узел. |
| Tor не подключается | Проверь firewall, разреши 9050/tcp |
| macchanger: «interface not found» | Проверь имя: `ip a` |
| macchanger не меняет MAC | Выключи интерфейс: `ip link set eth0 down` |
| MAC сбрасывается после перезагрузки | Настрой `macchanger` в автозапуске или скрипт |
| NetworkManager сбрасывает MAC | Отключи `NetworkManager` для интерфейса |

---

## 🔗 VI. СВЯЗИ

- **Инструменты:** [[bettercap]], [[nmap]], [[curl]], [[proxychains]], [[wireshark]]
- **Техники:** [[T1090.003]], [[T1036]], [[T1036.005]]
- **Лаба:** [[IP-Table]]
- **Защита:** [[Wazuh]], [[Suricata]], [[802.1X]]

---

## 📚 VII. ЧТО ИЗУЧИТЬ ДАЛЬШЕ

- [ ] **proxychains** — цепочка прокси для любых команд.
- [ ] **Tor bridges** — обход блокировок Tor.
- [ ] **Tails OS** — live-система с Tor по умолчанию.
- [ ] **Whonix** — VM с изоляцией Tor.
- [ ] **802.1X** — настройка в лабе (синяя сторона).
- [ ] **Tor exit node lists** — как блокировать на firewall.

---

## 📝 VIII. ШПАРГАЛКА

```bash
# Tor
cd tor-browser
./start-tor-browser.desktop

# Tor как прокси
sudo systemctl start tor
curl --socks5 127.0.0.1:9050 https://check.torproject.org/

# MAC — просмотр
macchanger -s eth0

# MAC — случайный
sudo ip link set eth0 down
sudo macchanger -r eth0
sudo ip link set eth0 up

# MAC — вернуть оригинал
sudo macchanger -p eth0

# MAC — свой
sudo macchanger -m XX:XX:XX:XX:XX:XX eth0
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Tor скрывает, откуда ты идёшь. Macchanger — кто ты. Вместе — призрак в сети. Но помни: даже призраки оставляют следы.*