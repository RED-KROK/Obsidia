---
date: 2026-10-01
week: 1
day: 2
tags: [journal, hydra, wazuh, ubuntu]
---

# 🐍 День 2 · Миссия «Ubuntu + Wazuh-агент»

**Дата:** 01.10.2026
**Оперативник:** Cor Vex (CV-7)
**Статус:** 🟡 Незавершено (продолжение после работы)

---

## 🎯 Цель дня
Подключить Ubuntu Server к Wazuh через агента и провести первую атаку с детектом.

---

## ✅ Что сделано

- [x] Запустил 3 VM: **Wazuh, Ubuntu, Kali**
- [x] Восстановил IP:
  - Kali — `192.168.56.107`
  - Ubuntu — `192.168.56.20`
  - Wazuh — `192.168.56.109`
- [x] Починил сеть Ubuntu (`ip link set enp0s3 up` + `ip addr add`)
- [x] Восстановил Host-only адаптер на Kali
- [x] Скачал `.deb` Wazuh-агента на Kali (`~/share/`)
- [x] Поднял HTTP-сервер на Kali (`python3 -m http.server 8000`)
- [x] Скачал `.deb` на Ubuntu через `wget` (13 MB, успешно)
- [x] Установил агент на Ubuntu (dpkg)

---

## ❌ Что не сработало

- Wazuh Manager **упал** — демоны `wazuh-modulesd`, `wazuh-analysisd`, `wazuh-db`, `wazuh-remoted`, `wazuh-execd` — stopped/failed.
- Панель пишет: `No API available to connect`.
- Гипотеза: модуль `vulnerability-detector` пытается скачать фиды из интернета, которого **нет** (изоляция). Падает → тянет за собой Manager.

---

## 📌 Что осталось на следующий раз

- [ ] `sudo /var/ossec/bin/wazuh-control status` — проверить, что именно упало
- [ ] Отключить `vulnerability-detector` в `/var/ossec/etc/ossec.conf`
- [ ] Перезапустить: `sudo /var/ossec/bin/wazuh-control restart`
- [ ] Проверить API: `curl -k -u wazuh-wui:wazuh-wui https://127.0.0.1:55000/`
- [ ] Проверить агента в панели (Endpoints → Active)
- [ ] **Атака:** `nmap -sV 192.168.56.20` с Kali
- [ ] **Детект:** Security events → `rule.id:5104/5105`
- [ ] Записать write-up #2: `02-wazuh-ubuntu-agent.md`

---

## ⚠️ Важно

- Файл `.deb` лежит в `/tmp` Ubuntu — **при перезагрузке исчезнет**.
  - Если будет `poweroff`: сначала `cp /tmp/wazuh-agent*.deb ~/`
- HTTP-сервер Kali работает только пока открыт терминал. При выключении — перезапускать.

---

## 💡 Что понял

- **`/tmp` очищается при перезагрузке.** Важные файлы — только в `~`.
- **Kali — bastion host.** Единственная VM с интернетом. Через неё качаем всё для изолированных VM.
- **Host-only адаптер легко отваливается** при правках VM. Всегда проверять `ip a` после перезапуска.
- **systemd timeout 90 сек** — мало для Wazuh. Нужно `TimeoutStartSec=600`.
- **Wazuh в изолированной сети** пытается тянуть файлы уязвимостей — модуль надо отключать.
## ✅ Финал дня
- [ ] Wazuh Manager починен (демоны Started)
- [ ] API Online
- [ ] Агент Ubuntu-server — **Active** (🟢 1 агент)
- [ ] Получены первые 316 алертов за сутки
---

## 🔗 Связи

- [[IP-Table]]
- [[Wazuh]]
- [[Ubuntu-Server]]
- [[01-first-scan]] ← первый write-up
- [[02-wazuh-ubuntu-agent]] ← следующий
