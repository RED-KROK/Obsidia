---
type: machine
name: <% tp.system.prompt("Имя машины:") %>
os: <% tp.system.prompt("ОС:") %>
role: <% tp.system.suggester(["Жертва Linux", "Жертва Windows", "Атакующий", "SIEM", "Уязвимая жертва"], ["Жертва Linux", "Жертва Windows", "Атакующий", "SIEM", "Уязвимая жертва"]) %>
ip: <% tp.system.prompt("IP:") %>
difficulty: <% tp.system.suggester(["Easy", "Medium", "Hard"], ["Easy", "Medium", "Hard"]) %>
tags: [lab, machine, target]
created: <% tp.date.now("YYYY-MM-DD") %>
---

# 🎯 <% tp.frontmatter.name %>

> [!info] ПАСПОРТ МАШИНЫ
> **ОС:** <% tp.frontmatter.os %>
> **Роль:** <% tp.frontmatter.role %>
> **IP:** <% tp.frontmatter.ip %>
> **Сложность:** <% tp.frontmatter.difficulty %>
> **Логин:** ?
> **Лаба:** [[IP-Table]]

---

## ⚙️ НАСТРОЙКИ VM

- **Адаптер:** Host-only
- **Интерфейс:** 
- **Особенности:** 

---

## 🔓 УЯЗВИМОСТИ / ОТКРЫТЫЕ ПОРТЫ

| Порт | Сервис | Версия | Риск |
|---|---|---|---|
|  |  |  |  |

---

## 📝 ЗАМЕТКИ

> [!tip] Что важно помнить

---

## 🔗 СВЯЗИ

- [[IP-Table]]
- [[Wazuh]]
- [[nmap]]
- [[msfconsole]]

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*