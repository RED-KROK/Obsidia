---
type: loot
tags: [ctf, loot, collection]
created: 2025-09-29
---

> [!warning] OPSEC
> Это **учебные** флаги с CTF-платформ. Реальные пароли, ключи, токены **никогда** не храни здесь. Если случайно попал реальный кред — удали и смени пароль.

# 🏆 LOOT: Коллекция флагов

> [!quote] Девиз
> *«Флаг — это не трофей. Это доказательство, что ты понял.»*

---

## 📋 Сводная таблица

| # | Платформа | Челлендж | Флаг / Кред | Дата | Write-up |
|---|---|---|---|---|---|
| 1 | rootme | First Blood | `nZ^&@q5&sjJHev0` | 2025-09-29 | [[first-blood]] |
| 2 | kznctf | HTTP (первый) | `kznctf{my_f1r$t_fl4g}` | 2025-09-29 | [[http1]] |
| 3 | kznctf | UDP socket | `kznctf{UdP_s0ck3t_c0nn3ct1on}` | 2025-09-29 | [[http2]] |
| 4 | kznctf | POST data | `kznctf{p0st_d4t4}` | 2025-09-29 | [[http3]] |
| 5 | kznctf | Dynamic HTTP | `kznctf{dyN4m1c_http_r3q43st}` | 2025-09-29 | [[http4]] |
| 6 | kznctf | POST with body | `kznctf{p0st_r3q43st_w1TH_b0dy}` | 2025-09-29 | [[http5]] |
| 7 | kznctf | JS object | `kznctf{js0bj3ct_N0T4BL3}` | 2025-09-29 | [[http6]] |
| 8 | kznctf | Magic headers | `kznctf{m4g1c_04ts1d3_H0gw4rts}` | 2025-09-29 | [[http7]] |

---

## 🧪 Коллекция кредов (для учебных лаб)

> [!info] Что это
> Учётные данные для доступа к **тренировочным** машинам. Используются только в изолированной лабе.

| Логин | Пароль | Где |
|---|---|---|
| `rtk16021602` | `uLbPjig24e` | ? |
| ? | `U48ekiYPAy` | ? |

> [!warning] Заполни сам
> В твоих записях не указано, **куда** эти логины/пароли. Это плохо. Всегда фиксируй: **платформа, машина, порт, сервис**. Иначе через месяц это мусор.

---

## 📊 Статистика

```dataview
TABLE WITHOUT ID
  count(rows) AS "Всего флагов"
FROM "03-Отчеты по Атакам/CTF"
WHERE type = "writeup"
```

---

> [!abstract] 🐍 HYDRA DOMINATUS
> *Vigilo. Vexo. Vinco.*
> *Флаг без write-up'а — просто строка. Write-up без флага — просто рассказ.*