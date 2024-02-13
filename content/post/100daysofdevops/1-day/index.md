---
title: 'День 1'
date: 2024-02-11T15:22:00+03:00
tags: [""]
author: "Arthur Lokhov"
showToc: true
TocOpen: true
draft: true
hidemeta: false
comments: true
description: "Посвящен выбору и установке Linux дистрибутива"
canonicalURL: ""
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
---
<!-- markdownlint-disable MD013 -->
## Выбор дистрибутива

Вот и начался первый день #100daysofdevops челленджа. И начать свой долгий, но интересный поход я решил с установки Linux.
Прежде всего я вышел в интернет и задал великому Google вопрос: "Какой дистрибутив популярен у бизнеса?". Среди ответов было следующие:

- RHEL
- Rocky Linux(логичное продолжение ныне закрытого CentOS)
- Debian
- Ubuntu

Грубо говоря предстоял выбор между RPM и DEB дистрибутивами. Я сразу решил, что если брать RPM дистрибутив, чтобы он был максимально близкий к первым двум, то это будет Fedora. А если понадобится DEB, то сам Debian.

Однако меня посетила мысль, что раз мне так или иначе в будущем предстоит изучать виртуализацию и контейнеры, то и эти два дистрибутива я смогу рассмотреть позже.

Так что же я по итогу выбрал? Artix - Arch без systemd. Но на чем был основан такой выбор?

Тут несколько важных пунктов:

- он не systemd, то есть я смогу разобраться в чем различие между разными системами инициализации;
- он Arch подобный, то есть я смогу настроить из коробки все так как мне оно нужно;
- для разработчика, а особенно для Devops-инженера важна свежесть софта, так как надо знать все изменения тех инструментов, которые ты используешь и главное иметь возможность их протестировать;
- pacman является одним из быстрейших пакетных менеджеров, а AUR второ по велечине репозиторий(после nixpkgs).

## Установка Artix

Ряд вещей, которые стоит знать перед установкой:

- инструкция ниже справедлива только для UEFI систем;
- на некоторых системах изменения grub работают корректно, только после полной установки и перезапуска. Так что если вы не знакомы с выкрутасами своей системы, то стоит настраивать grub после перезапуска, то есть использовать дефолтный конфиг на момент установки;
- ваша система должна иметь минимум 16Гб и 8 потоков, иначе на сборке некоторых программ придется крайне долго ждать. И некоторые программы будут медленно работать;
- Инструкция была написана под runit версию Artix.

### Создаем загрузочную флешку

У Artix есть какие-то проблемы с тем, чтобы распоковать образ через balenaEtcher или через rufus, поэтому пользуемся неустариваемой классикой - командой dd.

```sh
sudo dd bs=4M conv=sync oflag=sync status=progress if=./artix.iso of=/dev/sdX
```

Незабываем заменить sdX на необходимое нам устройство.

Что же теперь перейдем к базовой установке Artix.

### Подключение к Wi-Fi

Этот пункт опционален, так как у вас может быть проводное подключение.

Для того, чтобы на Artix использовать wifi, его надо разблокировать. Для этого:

```sh
sudo rfkill unblock wifi
```

После разблокировки можно переходить к подключению.

```sh
# Из коробки доступен только connman, позже мы его заменим на networkmanager
conmmanctl
> scan wifi
> services
> agent on
> connect wifi_id
> exit
ping www.google.com -c 5
```

### Файловые системы и разметка диска

Перед тем, как начать базовую установку Artix, мы должны разметить файловую систему. Я буду использовать btrfs по ряду причин:

- Возможность делать снимки системы;
- Можно легко создать разделы для определенныз каталогов в системе;
- Крайне быстрая и оптимизированная файловая система.

Сначала переходим в инструмент разметки и создаем два раздела:

- root
- efi

Раздел подкачки нам не нужен. Почему он нам не нужен? Будет использоваться zram как инструмент для манипуляции разделом подкачки.

```sh
sudo cfdisk /dev/sdX
# Создаем 1Гб - Efi Partition
# Создаем +100%FREE - Linux filesystem
```

Теперь перейдем к созданию файловой системы.

```sh
su -
mkfs.fat -F 32 /dev/sdX1
mkfs.btrfs -L root -f /dev/sdX2
mount /dev/disk/by-label/root /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@opt
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@docker
btrfs subvolume create /mnt/@libvirt
umount /mnt

mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@ /dev/disk/by-label/root /mnt
mkdir -p /mnt/{home,opt,tmp,.snapshots,var/log,var/lib/docker,var/lib/libvirt,boot/efi}

mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@home /dev/disk/by-label/root /mnt/home
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@tmp /dev/disk/by-label/root /mnt/tmp
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@opt /dev/disk/by-label/root /mnt/opt
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@snapshots /dev/disk/by-label/root /mnt/.snapshots
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@log /dev/disk/by-label/root /mnt/var/log
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@docker /dev/disk/by-label/root /mnt/var/lib/docker
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@libvirt /dev/disk/by-label/root /mnt/var/lib/libvirt

mount /dev/sdX1 /mnt/boot/efi

lsblk # Проверяем все ли в порядке
```
