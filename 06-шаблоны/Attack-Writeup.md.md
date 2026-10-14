---
type: writeup
target: <% tp.system.prompt("Цель (IP/имя):") %>
date: <% tp.date.now("YYYY-MM-DD") %>
severity: <% tp.system.suggester(["Low", "Medium", "High", "Critical"], ["Low", "Medium", "High", "Critical"]) %>
tags: [writeup, red-team, hydra]
---

> [!info] 🎯 ДОСЬЕ ОПЕРАЦИИ
> **Цель:** <% tp.frontmatter.target %>
> **Дата:** <% tp.frontmatter.date %>
> **Оперативник:** Cor Vex (CV-7)
> **Критичность:** <% tp.frontmatter.severity %>
> **Лаба:** [[IP-Table]]

---

## 📖 I. РАЗВЕДКА (RECON)

> [!quote] Девиз этапа
> *«Знай врага, прежде чем касаться его.»*

### Команды
```bash
# Разведка
```

### Что обнаружено
| Порт | Сервис | Версия | Комментарий |
|------|--------|--------|-------------|
|      |        |        |             |

**Выводы разведки:**
- 

---

## 🧠 II. ЭКСПЛУАТАЦИЯ (EXPLOITATION)

> [!warning] Уязвимость
> **CVE / Название:**
> **Тип:**
> **Вектор:**

### Шаги атаки
```bash
# Команды эксплуатации
```

### Результат
- 

---

## 🏁 III. ФЛАГИ (FLAGS)

> [!success] Захвачено
> - **User:** `flag{...}`
> - **Root:** `flag{...}`

---

## 🛡️ IV. ДЕТЕКТ В WAZUH

> [!danger] Что поймал защитник
> **Rule ID:**
> **Level:**
> **MITRE Technique:**
> **Source IP:**

### Лог из Wazuh
```json
// Вставить сырой лог
```

### Оценка детекта
- [ ] Wazuh поймал атаку
- [ ] Wazuh пропустил атаку
- [ ] Ложное срабатывание

**Что можно улучшить в правилах:**
- 

---

## 📚 V. ВЫВОДЫ

### Что сработало
- 

### Что не сработало
- 

### Что изучить дальше
- [ ] 
- [ ] 

---

## 🔗 VI. СВЯЗИ

- **Инструменты:** [[Nmap]], [[Metasploit]], 
- **Техники:** [[T1046]], [[T1190]]
- **Машина:** [[Metasploitable2.md]]
- **Предыдущий write-up:** [[]]
- **Следующий write-up:** [[]]

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Cor inquietum, hostis intus.*