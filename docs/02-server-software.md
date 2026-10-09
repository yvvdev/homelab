# 2. Программное обеспечение сервера

> **Версия:** v3 · **Дата:** 2026-10-09 · **Статус:** действующий (после подготовки сервера 2026-10-09)

## 2.1. Гипервизор

| Параметр | Значение |
|---|---|
| ПО | Proxmox Virtual Environment |
| Версия | **9.2.21** (`pve-manager 9.2.21`) |
| Базовая ОС | Debian 13 «trixie» |
| Ядро | `7.0.14-23-pve` (ядра 6.14 оставлены как резервные в меню GRUB) |
| Подписка | Нет, репозиторий `pve-no-subscription` |
| Веб-интерфейс | `https://192.168.1.100:8006` (локально), `https://proxmox.<домен>` (через туннель и Cloudflare Access) |
| Дата установки | ⏳ уточнить |

## 2.2. Разметка диска и хранилища

Диск `/dev/sda` (1,8 ТиБ):

| Раздел / том | Размер | Тип | Назначение |
|---|---|---|---|
| `sda1` | 1 МБ | — | BIOS boot |
| `sda2` | 1 ГБ | vfat | EFI / загрузчик |
| `sda3` | 1,8 ТБ | LVM PV (группа `pve`) | Всё остальное |
| └ `pve/swap` | 8 ГБ | swap | Подкачка |
| └ `pve/root` | 96 ГБ | ext4, `/` | Система Proxmox |
| └ `pve/data` | 1,7 ТБ | LVM-thin pool | Диски VM и контейнеров |

Хранилища Proxmox (2026-10-09, после подготовки):

| ID | Тип | Объём | Занято | Содержимое |
|---|---|---|---|---|
| `local` | dir (`/var/lib/vz`) | 94 ГиБ | 12,6 % | ISO-образы, шаблоны, бэкапы |
| `local-lvm` | lvmthin | 1,67 ТиБ | 2,7 % | Диски VM / LXC |

ISO-образы в `local`: `Win10_22H2_Russian_x64v1.iso`, `virtio-win-0.1.285.iso`.

## 2.3. Службы на хосте

| Служба | Назначение | Конфигурация | Статус |
|---|---|---|---|
| `cloudflared` | Cloudflare Tunnel `proxmox-tunnel` | Бинарный файл `/usr/local/bin/cloudflared` (версия 2025.9.0, автообновление отключено); unit `/etc/systemd/system/cloudflared.service`; дополнение `/etc/systemd/system/cloudflared.service.d/override.conf` — запуск после `network-online.target` | active, enabled |
| `pveproxy` | Веб-интерфейс Proxmox | В составе Proxmox | active |
| `ssh` (`sshd`) | SSH-доступ к хосту | `/etc/ssh/sshd_config.d/10-hardening.conf` — вход только по ключам | active |

Конфигурация туннеля: `/etc/cloudflared/config.yml` (см. [04](04-network.md)).
Учётные данные туннеля: `/root/.cloudflared/<tunnel-id>.json` — **секрет**, не копируется и не публикуется.

### 2.3.1. Изменённые конфигурационные файлы

| Файл | Изменение | Резервная копия |
|---|---|---|
| `/etc/network/interfaces` | Исправлена опечатка `vmdr0` → `vmbr0`, IP `.101` → `.100`, убраны лишние строки | `/etc/network/interfaces.bak-2026-10-09` |
| `/etc/ssh/sshd_config.d/10-hardening.conf` | Создан: `PasswordAuthentication no`, `KbdInteractiveAuthentication no`, `PermitRootLogin prohibit-password` | — (новый файл) |
| `/etc/systemd/system/cloudflared.service.d/override.conf` | Создан: `Wants=`/`After=network-online.target` | — (новый файл) |
| `/etc/apt/sources.list.d/*.sources` | Отключены enterprise-репозитории, добавлены no-subscription (через веб-интерфейс) | — |

## 2.4. Репозитории пакетов

| Репозиторий | Компонент | Статус |
|---|---|---|
| `deb.debian.org` trixie, trixie-updates | main contrib non-free-firmware | ✅ Включён |
| `security.debian.org` trixie-security | main contrib non-free-firmware | ✅ Включён |
| `download.proxmox.com/debian/pve` | pve-no-subscription | ✅ Включён |
| `download.proxmox.com/debian/ceph-squid` | no-subscription | ✅ Включён (клиентские пакеты Ceph) |
| `enterprise.proxmox.com/debian/pve` | pve-enterprise | ⛔ Отключён (`Enabled: false`) |
| `enterprise.proxmox.com/debian/ceph-squid` | enterprise | ⛔ Отключён (`Enabled: false`) |

Процедура обновления: `apt update && apt full-upgrade` (только `full-upgrade`, не `upgrade`), затем перезагрузка при обновлении ядра. Выполнять с физической консоли или по SSH, не через Shell веб-интерфейса.

## 2.5. Пользователи хоста

| Пользователь | Realm | Назначение | Вход |
|---|---|---|---|
| `root` | pam | Администратор | Веб-интерфейс: пароль + TOTP. SSH: только ключ. Консоль: пароль |

## 2.6. Журнал обновлений

| Дата | Версия до | Версия после | Ядро | Примечание |
|---|---|---|---|---|
| 2026-10-09 | 9.0.3 | 9.2.21 | 6.14.8-2 → 7.0.14-23 | Первое обновление после смены репозиториев, ~270 пакетов, в т.ч. обновления безопасности и микрокод CPU |

## 2.7. Реестр проблем

| № | Проблема | Критичность | Статус |
|---|---|---|---|
| P-01 | Опечатка `auto vmdr0` в `/etc/network/interfaces`, IP в файле не совпадал с рабочим — сеть не поднималась после перезагрузки | 🔴 Высокая | ✅ Закрыта 2026-10-09 |
| P-02 | Подключены только enterprise-репозитории — обновления не устанавливались | 🟠 Средняя | ✅ Закрыта 2026-10-09 |
| P-03 | Веб-интерфейс Proxmox доступен из интернета, защищён только паролем `root` | 🟠 Средняя | ✅ Закрыта 2026-10-09 (Cloudflare Access + TOTP) |
| P-04 | У VM 100–102 избыточно задан CPU (12 vCPU) и память | 🟡 Низкая | ✅ Закрыта 2026-10-09 (VM 101, 102 удалены; VM 100 перенастроена) |

## 2.8. Команды для аудита

Сбор данных одним запуском с рабочей станции (результат — в файл на ноутбуке):

```bash
ssh pve 'bash -s' > ~/server-audit-$(date +%F).txt <<'EOF'
pveversion; uname -r; free -h
smartctl -H -A /dev/sda | grep -E "overall|Reallocated_Sector|Current_Pending|Power_On_Hours"
pvesm status; qm list; pct list
ip -br a; ip route | grep default
grep -hE "^(URIs|Components|Enabled)" /etc/apt/sources.list.d/*.sources
systemctl is-active cloudflared ssh pveproxy
sshd -T | grep -E "^passwordauthentication|^permitrootlogin"
pveum user tfa list root@pam
EOF
```
