---
title: 'Установка и конфигурация дистрибутива Debian'
date: 2024-03-22T21:37:04+03:00
tags: ["games", "blog", "linux", "debian"]
author: "Arthur Lokhov"
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: true
description: "Моя конфигурация дистрибутива Debian"
canonicalURL: ""
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
---

## О системе

- Корпус: AeroCool Aero One Mini Eclipse Black
- Мат.плата: MSI H510M-A PRO
- Процессор: Intel Core i5 - 11400 OEM
- Система охлождения: PCcooler GI-Paladin 400
- Оперативная память: 32Gb DDR4 3600MHz ADATA XPG Gammix D10 (AX4U360016G18I-DB10) (2x16Gb KIT)
- Хранилище: 1Tb ADATA XPG SX8100 (ASX8100NP-1TT-C)
- Видеокарта: NVIDIA GeForce RTX 3070 Gigabyte 8Gb LHR (GV-N3070EAGLE OC-8GD 2.0)
- Клавиатура: ErgoHaven K:02(отсылка, угадайте на кого) - на данный момент не выпускается, теперь K:03

## Почему в качестве основной машины я выбрал Debian?

- Самый стабильный по-моему мнению дистрибутив
- Apt-get используется в огромном количестве мануалов
- Почти каждый VPS поддерживает Debian
- Xanmod + последние драйвера Nvidia = крайне хорошая сборка для игр
- Ничего не ломается из-за "нестабильных" пакетов
- Если все же надо есть Debian Sid(не устанавливайте xanmod, он сломает систему из-за пакетов в Sid)

## Установка системы

Сначала ставим "голую" систему через "Expert Install".

Далее выполняем команды ниже по порядку.

1. Выдаем root-права созданному пользователю(если надо)
```sh
su -
usermod -aG sudo $user
reboot
```
2. Установка must-have пакетов
```sh
sudo apt install nala git timeshift curl build-essential
```
3. Обновление системы
```sh
sudo dpkg --add-architecture i386
sudo nala update && sudo nala upgrade
```
4. Установка xanmod ядра
```sh
# Создаем резерв в Timeshift и идем далее
wget -qO - https://dl.xanmod.org/archive.key | sudo gpg --dearmor -o /usr/share/keyrings/xanmod-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-release.list
sudo nala update
curl -LO https://dl.xanmod.org/check_x86-64_psabi.sh
sudo chmod +x check_x86-64_psabi.sh
./check_x86-64_psabi.sh # Получаем версию ядра от 1 до 4, используем в команде ниже вместо ?
sudo nala install linux-xanmod-x64v?
sudo reboot
```
5. Установка и удаление драйверов Nvidia

Инструкция [тут](https://i4ox.github.io/devops_blog/post/nvidia_debian/nvidia/)

6. Установка пакетных менеджеров

Cargo:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install alacritty # For example
```

Homebrew:

```sh
sudo nala install build-essential procps curl file git
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
brew install flameshot # For example
```

Deb-get:

```sh
sudo nala install curl lsb-release wget
curl -sL https://raw.githubusercontent.com/wimpysworld/deb-get/main/deb-get | sudo -E bash -s install deb-get
```

NixPkgs:

```sh
sh <(curl -L https://nixos.org/nix/install) --daemon
sudo reboot
```

Flatpak:

```sh
sudo nala install flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
sudo reboot
```

Snap(если вы мазохист):

```sh
sudo nala install snapd
sudo snap install core
```

По мере того как я буду находить что-то полезное про Debian это будет появляться тут
