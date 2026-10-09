# 5. Рабочая станция («ОС-инструмент»)

> **Версия:** v1 · **Дата:** 2026-10-08 · **Статус:** действующий

## 5.1. Операционная система

| Параметр | Значение |
|---|---|
| ОС | Windows 11 Pro (официальный ISO Microsoft) |
| Сборка | 10.0.26300 |
| Носитель | Внешний SSD (USB) |
| Имя компьютера | `DESKTOP-83O1RQP` |
| Учётная запись | Локальная, `Yarik` |
| Профиль | `C:\Users\Yarik` |
| Изоляция | Внутренний диск ноутбука системе не виден |
| Шифрование | ⏳ BitLocker — планируется (этап 8) |

> Предыдущая установка («lite»-сборка стороннего автора) удалена 2026-10-08 — см. [ADR-001](08-decisions.md).

## 5.2. ПО Windows

| ПО | Версия | Способ установки | Назначение |
|---|---|---|---|
| winget | 1.29.380 | В составе Windows | Менеджер пакетов |
| PowerShell | 7.6.6 | `winget` (`Microsoft.PowerShell`) | Оболочка, профиль по умолчанию в терминале |
| Git for Windows | 2.55.0 | `winget` (`Git.Git`) | Контроль версий |
| Visual Studio Code | 1.140.0 | `winget` (`Microsoft.VisualStudioCode`) | Редактор |
| 7-Zip | — | `winget` (`7zip.7zip`) | Архиватор |
| Windows Terminal | — | В составе Windows | Терминал |
| OpenSSH Client | — | В составе Windows | SSH-клиент, `ssh-agent` (автозапуск) |

## 5.3. WSL2

| Параметр | Значение |
|---|---|
| WSL | 3.0.1.0, ядро 6.18.40.1-1 |
| Дистрибутив | Ubuntu 24.04 LTS |
| Пользователь | `yarik` |
| Каталог проектов | `~/projects` |

**Установленные пакеты (apt):** git, curl, wget, unzip, zip, jq, tree, htop, tmux, bash-completion, dnsutils, net-tools, ca-certificates, gnupg, keychain, build-essential.

**Известные особенности:** при обновлениях появляется `systemd-binfmt.service failed` — штатная особенность WSL, на работу не влияет.

## 5.4. Конфигурация Git

| Параметр | Windows | WSL |
|---|---|---|
| `user.name` | `yvvdev` | `Yaroslav Ivoylov` |
| `user.email` | настроен | настроен |
| `init.defaultBranch` | `main` | `main` |
| `core.editor` | `code --wait` | `code --wait` |
| `core.autocrlf` | — | `input` |
| `core.sshCommand` | `C:/Windows/System32/OpenSSH/ssh.exe` | — |

## 5.5. Расширения VS Code

| Расширение | Где | Статус |
|---|---|---|
| Remote - WSL | Windows | ⏳ этап 4 |
| Remote - SSH | Windows | ⏳ этап 4 |
| YAML (Red Hat) | Windows + WSL | ⏳ этап 4 |
| GitLens | Windows + WSL | ⏳ этап 4 |
| Claude Code | по желанию | ⏳ |

## 5.6. Инструменты DevOps (план, этап 7)

| Инструмент | Где | Статус |
|---|---|---|
| Docker | WSL | ⬜ |
| Terraform / OpenTofu | WSL | ⬜ |
| Ansible | WSL | ⬜ |
| kubectl, helm, k9s | WSL | ⬜ |
