---
type: knowledge
topic: OS / Linux
platform: Linux (Kali/Debian/Ubuntu)
status: mastering
tags: [knowledge, os, linux, admin, fundamentals, apt, blue-team, red-team]
mitre:
  - T1082      # System Information Discovery
  - T1083      # File and Directory Discovery
  - T1552.001  # Unsecured Credentials: Credentials In Files
  - T1070.003  # Clear Command History
  - T1548      # Abuse Elevation Control Mechanism
created: 2025-09-29
---

> [!info] 📚 БАЗА ЗНАНИЙ
> **Тема:** Linux (файловая система, команды, администрирование)
> **Платформа:** Kali, Debian, Ubuntu
> **Для чего:** Полный справочник по работе в Linux — от навигации до apt и VirtualBox
> **Уровень:** 🟢 → 🟡

---

# ЧАСТЬ I. ФУНДАМЕНТ

## 📖 I.1. СТРУКТУРА ФАЙЛОВОЙ СИСТЕМЫ

| Каталог | Что там | Примеры |
|---|---|---|
| `/` | Главный каталог | всё |
| `/bin` | Базовые команды | `ls`, `cp`, `bash` |
| `/sbin` | Утилиты для админа | `fdisk`, `iptables`, `init` |
| `/lib` | Библиотеки для `/bin`, `/sbin` | `libc.so` |
| `/usr` | Пользовательские программы | `/usr/bin`, `/usr/lib` |
| `/etc` | **Конфиги** | `/etc/passwd`, `/etc/network/interfaces` |
| `/var` | Переменные данные | `/var/log`, `/var/www` |
| `/tmp` | Временные файлы | очищаются |
| `/boot` | Загрузчик | `vmlinuz`, GRUB |
| `/dev` | Файлы устройств | `/dev/sda`, `/dev/tty1` |
| `/proc` | Виртуальная ФС процессов | `/proc/cpuinfo` |
| `/sys` | Информация о ядре | — |
| `/home` | Домашние каталоги | `/home/user` |
| `/root` | Дом. каталог root | не путать с `/` |
| `/mnt` | Временные монтирования | флешки |
| `/media` | Автомонтирование | `/media/usb` |
| `/opt` | Доп. ПО | `/opt/google/chrome` |

> [!tip] Что важно для SOC
> - **`/var/log`** — логи (главное для Blue Team).
> - **`/etc/passwd`** — пользователи.
> - **`/etc/shadow`** — хэши паролей (только root).
> - **`/usr/bin`** — исполняемые файлы.
> - **`/etc`** — конфиги.
> - **`/usr/share/doc`** — документация.

### Windows vs Linux — соответствие

| Linux | Windows | Назначение |
|---|---|---|
| `/home/user` | `C:\Users\user` | Домашние каталоги |
| `/etc` | `C:\ProgramData` + Реестр | Конфиги |
| `/bin`, `/usr/bin` | `C:\Windows\System32` | Исполняемые файлы |
| `/dev/sda1` | `\\.\PhysicalDrive0` | Устройства |

---

## 🎯 I.2. ОСНОВНЫЕ КОМАНДЫ

### Навигация

| Команда | Что делает |
|---|---|
| `pwd` | Показать текущий каталог |
| `cd <path>` | Перейти в каталог |
| `cd ..` | На уровень выше |
| `ls` | Список файлов |
| `ls -la` | Все файлы + права + скрытые |

### Работа с файлами

| Команда | Что делает |
|---|---|
| `cat <file>` | Показать содержимое |
| `touch <file>` | Создать файл |
| `vi` / `nano <file>` | Создать + редактировать |
| `cp <src> <dst>` | Копировать |
| `mv <src> <dst>` | Переместить / переименовать |
| `mkdir <dir>` | Создать папку |
| `rm <file>` | Удалить файл |
| `rmdir <dir>` | Удалить пустую папку |
| `rm -rf <dir>` | **ОПАСНО!** Удалить рекурсивно |
| `head <file>` | Первые строки |
| `tail <file>` | Последние строки |
| `less <file>` | Просмотр постранично |
| `echo "text"` | Вывести текст |
| `echo "text" >> file` | Дописать в файл |
| `clear` | Очистить терминал |

### Поиск

| Команда | Что делает |
|---|---|
| `grep "text" file` | Поиск по файлу |
| `grep -r "text" dir/` | Рекурсивный поиск |
| `grep -i "text"` | Игнорировать регистр |
| `grep -E "regex"` | Regex-поиск |
| `find /path -type f` | Найти файлы |
| `find /etc -type f -exec grep -l "Flag" {} \;` | Найти файл с текстом |
| `find /home -type f 2>/dev/null` | Игнорируя ошибки |

### CTF-приёмы

```bash
# Поиск флагов
grep -r "CTF{.*}" /path/
grep -r "flag{.*}" /path/
grep -ri "flag" /path/

# Поиск в конфигах
find /etc -type f -exec grep -l "Flag" {} \;

# Поиск в домашних директориях
find /home -type f -exec grep -l "Flag" {} \; 2>/dev/null
```

### Сортировка и обработка

| Команда | Что делает |
|---|---|
| `sort <file>` | Сортировка |
| `uniq` | Уникальные строки |
| `seq 0 1000 > 1.txt` | Создать файл с числами 0–1000 |
| `chmod <mode> <file>` | Изменить права |
| `man <command>` | Справка |
| `<command> --help` | Краткая справка |

### Опасные команды

> [!danger] НИКОГДА
> ```bash
> rm -rf /           # мгновенно удалит всю систему
> ```

### Права root

```bash
sudo su                       # переключиться на root
sudo <command>                # выполнить команду от root
```

### Удаление программ

```bash
sudo apt remove <программа>   # удалить (конфиги остаются)
sudo apt purge <программа>    # удалить + конфиги
```

### История команд

```bash
history                       # показать историю
history -c                    # очистить текущую сессию
nano .zsh_history             # очистить историю zsh
```

> [!warning] Почему это важно
> `history` и `.zsh_history` — **следы атакующего**. В пентесте их чистят (`T1070.003`). В SOC — ищут.

---

## 📝 I.3. NANO — ГОРЯЧИЕ КЛАВИШИ

| Клавиша | Действие |
|---|---|
| `Ctrl + O` | Сохранить файл (Write Out) |
| `Ctrl + X` | Выйти |
| `Ctrl + K` | Вырезать строку |
| `Ctrl + U` | Вставить |
| `Ctrl + W` | Поиск |
| `Ctrl + \` | Заменить текст |
| `Ctrl + G` | Справка |
| `Ctrl + C` | Показать позицию курсора |
| `Ctrl + _` | Перейти на строку |

---

# ЧАСТЬ II. АДМИНИСТРИРОВАНИЕ

## 📦 II.1. APT — ПАКЕТНЫЙ МЕНЕДЖЕР

### Основные функции

| Функция | Команда |
|---|---|
| Поиск пакетов | `apt search <запрос>` |
| Установка | `apt install <пакет>` |
| Обновление системы | `apt upgrade` / `apt full-upgrade` |
| Удаление | `apt remove <пакет>` |

### Полезные команды

```bash
# Обновить списки пакетов
sudo apt update

# Обновить установленные пакеты
sudo apt upgrade -y

# Полное обновление (включая ядро)
sudo apt full-upgrade -y

# Очистка
sudo apt-get autoremove -y     # удалить ненужные зависимости
sudo apt-get autoclean -y      # очистить кэш устаревших
sudo apt-get clean -y          # очистить весь кэш

# Починить сломанные зависимости
sudo apt-get install -f -y
```

---

## 🐉 II.2. KALI LINUX — СПЕЦИФИКА

### Обновление Kali

```bash
sudo apt update
sudo apt full-upgrade -y
```

### Источники обновлений

**По умолчанию** (`/etc/apt/sources.list`):
```
deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware
```

**Через HTTPS:**
```bash
echo -e "deb https://http.kali.org/kali kali-rolling main non-free contrib" > /etc/apt/sources.list
```

### Смена языка

```bash
sudo dpkg-reconfigure locales
sudo reboot
```

### Bluetooth автозапуск

```bash
sudo systemctl enable bluetooth
```

---

## 🎭 II.3. СМЕНА MAC-АДРЕСА

### Способ 1: macchanger (быстро)

```bash
sudo ip link set eth0 down
sudo macchanger -r eth0
sudo ip link set eth0 up
```

Подробнее: [[anonymity]]

### Способ 2: через NetworkManager (постоянно)

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

Добавь:
```ini
[connection]
wifi.cloned-mac-address=random
ethernet.cloned-mac-address=random
```

Перезапусти:
```bash
sudo service NetworkManager restart
```

---

## 🖥️ II.4. VIRTUALBOX — ТРОУБЛШУТИНГ

### Проблема: `Kernel driver not installed`

**Быстрое решение:**
```bash
sudo modprobe vboxdrv
```

**Полное решение:**
```bash
sudo apt update
sudo apt install --reinstall linux-headers-$(uname -r) virtualbox-dkms dkms
sudo reboot
sudo modprobe vboxdrv
```

### Вариант 2: источники + переустановка

```bash
sudo nano /etc/apt/sources.list
# Добавить в начало:
# deb http://http.kali.org/kali kali-rolling main non-free contrib
# deb-src http://http.kali.org/kali kali-rolling main non-free contrib

sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
sudo reboot
sudo apt-get install linux-headers-$(uname -r)
```

---

## 💻 II.5. УСТАНОВКА VSCODIUM

### Debian

```bash
sudo apt update && sudo apt install extrepo -y
sudo extrepo enable vscodium
extrepo search vscodium
sudo apt update && sudo apt install codium -y
codium --version
```

### Kali

```bash
wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg \
    | gpg --dearmor \
    | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg

echo 'deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg] https://download.vscodium.com/debs vscodium main' \
    | sudo tee /etc/apt/sources.list.d/vscodium.list

sudo apt update && sudo apt install codium
```

> [!info] Что такое VSCodium
> Свободная сборка VS Code без телеметрии Microsoft.

---

## 💾 II.6. USB-ФЛЕШКА НЕ ПОДКЛЮЧАЕТСЯ

### Ручное монтирование

```bash
# Создать точку монтирования
sudo mkdir /media/my_flash

# Примонтировать
sudo mount -t ntfs-3g /dev/sdb1 /media/my_flash   # NTFS
sudo mount -t vfat /dev/sdb1 /media/my_flash      # FAT32

# Посмотреть устройства
lsblk
sudo fdisk -l

# Размонтировать
sudo umount /media/my_flash
```

> [!warning] Не выдёргивай флешку без umount
> Это повредит файловую систему.

---

# ЧАСТЬ III. БЕЗОПАСНОСТЬ

## 🛡️ III.1. ДЕТЕКТ В WAZUH

> [!danger] MITRE
> - **T1082** — System Information Discovery
> - **T1083** — File and Directory Discovery
> - **T1552.001** — Credentials In Files
> - **T1070.003** — Clear Command History
> - **T1548** — Abuse Elevation Control Mechanism

### Что видит защитник

| Источник | Что искать |
|---|---|
| **Auditd** | Запуск `find`, `grep -r`, `ls -la /etc/` |
| **Wazuh FIM** | Изменения в `/etc/`, `/var/log/`, `/etc/apt/sources.list` |
| **Wazuh Rule 550** | Integrity checksum changed |
| **Bash history** | `rm .bash_history`, `history -c` |
| **Sysmon (Linux)** | Необычные команды от сервисных юзеров |

### Красные флаги

- **Рекурсивный `find`** по `/etc/`, `/home/` — разведка.
- **`grep -r` по всей системе** — поиск кред/флагов.
- **`rm .bash_history`** — очистка следов.
- **`chmod 777`** на критичных файлах.
- **Изменения источников apt** — supply-chain attack.
- **`sudo su`** на сервере — эскалация.

### Практика в лабе

1. На Ubuntu запусти:
   ```bash
   grep -r "flag" /etc/ 2>/dev/null
   find /home -type f 2>/dev/null
   ```
2. В Wazuh → **Security events** → фильтр по `command`.
3. Разбери: кто, что, когда.

---

## 🔗 IV. СВЯЗИ

- **Инструменты:** [[network-basics]], [[anonymity]], [[nmap]], [[msfconsole]]
- **Техники:** [[T1082]], [[T1083]], [[T1552.001]], [[T1070.003]], [[T1548]]
- **Лаба:** [[IP-Table]]
- **Машины:** [[Ubuntu Server]], [[Metasploitable2.md]]
- **Защита:** [[Wazuh]]

---

## 📝 V. ШПАРГАЛКА

```bash
# === НАВИГАЦИЯ ===
pwd
ls -la
cd /path

# === ФАЙЛЫ ===
touch file
cat file
nano file
cp src dst
mv src dst
mkdir dir
rm file
rm -rf dir  # ОПАСНО

# === ПОИСК ===
grep "text" file
grep -r "text" dir/
grep -ri "flag" /path/
find /path -type f

# === ПРАВА ===
chmod 755 file
sudo su
sudo <cmd>

# === ОБРАБОТКА ===
sort file
uniq
head -10 file
tail -10 file
less file

# === APT ===
sudo apt update && sudo apt full-upgrade -y
sudo apt install <пакет>
sudo apt remove <пакет>
sudo apt purge <пакет>
sudo apt-get autoremove -y && sudo apt-get clean -y
sudo apt-get install -f -y

# === KALI ===
sudo dpkg-reconfigure locales
sudo systemctl enable bluetooth

# === VIRTUALBOX FIX ===
sudo modprobe vboxdrv
sudo apt install --reinstall linux-headers-$(uname -r) virtualbox-dkms dkms

# === USB MOUNT ===
sudo mkdir /media/flash
sudo mount -t ntfs-3g /dev/sdb1 /media/flash
sudo umount /media/flash

# === MAC CHANGE ===
sudo ip link set eth0 down
sudo macchanger -r eth0
sudo ip link set eth0 up

# === ОПАСНОЕ ===
rm -rf /  # НИКОГДА
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Linux — это не система. Это язык. Кто его знает — говорит с машиной напрямую. Кто нет — тыкается в GUI.*