---
type: writeup
subtype: ctf
platform: <% tp.system.prompt("Платформа (rootme/THM/HTB/kznctf):") %>
challenge: <% tp.system.prompt("Название челленджа:") %>
category: <% tp.system.suggester(["Web", "Crypto", "Forensics", "Reverse", "Pwn", "Misc", "Network"], ["Web", "Crypto", "Forensics", "Reverse", "Pwn", "Misc", "Network"]) %>
difficulty: <% tp.system.suggester(["Easy", "Medium", "Hard", "Insane"], ["Easy", "Medium", "Hard", "Insane"]) %>
flag: <% tp.system.prompt("Флаг:") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: [ctf, writeup, hydra]
---

# 🏴 <% tp.frontmatter.platform %> — <% tp.frontmatter.challenge %>

> [!info] ДОСЬЕ ЧЕЛЛЕНДЖА
> **Платформа:** <% tp.frontmatter.platform %>
> **Категория:** <% tp.frontmatter.category %>
> **Сложность:** <% tp.frontmatter.difficulty %>
> **Дата:** <% tp.frontmatter.date %>
> **Флаг:** `<% tp.frontmatter.flag %>`

---

## 📖 I. УСЛОВИЕ

> Вставь сюда описание челленджа с платформы.

---

## 🔍 II. РАЗВЕДКА

```bash
# Команды разведки
```

**Что обнаружил:**
- 

---

## 🧠 III. РЕШЕНИЕ

### Гипотеза
- 

### Шаги
1. 
2. 
3. 

### Команды
```bash
# Рабочие команды
```

### Скриншот / лог
![[screenshot.png]]

---

## 🏁 IV. ФЛАГ

> [!success] Захвачено
> ```
> <% tp.frontmatter.flag %>
> ```

---

## 🛡️ V. ДЕТЕКТ (если применимо)

> [!danger] Что должен поймать защитник
> **MITRE:**
> **Wazuh Rule:**
> **Как выглядит в логах:**

---

## 📚 VI. ВЫВОДЫ

### Что понял
- 

### Что изучить дальше
- [ ] 
- [ ] 

---

## 🔗 VII. СВЯЗИ

- **Инструменты:** 
- **Техники:** 
- **Похожие челленджи:** [[]]

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*