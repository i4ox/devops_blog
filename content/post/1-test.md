---
title: 'Настройка дистрибутива Artix Linux'
date: 2024-02-08T10:04:36+03:00
url: /artix_setup/
tags:
- Arch
- Artix
draft: true
---
<!-- markdownlint-disable MD013 -->
![Intro](/images/intro_linux.png)

## Компоненты

|                           | Artix(with Wayland) | Artix(with Xorg) |
|---------------------------|---------------------|------------------|
| Window Manager            | [Hyprland](https://github.com/hyprwm/Hyprland) | [i3](https://github.com/i3/i3) |
| Terminal Emulator         | [Alacritty](https://github.com/alacritty/alacritty) + [Tmux](https://github.com/tmux/tmux) |  [Alacritty](https://github.com/alacritty/alacritty) + [Tmux](https://github.com/tmux/tmux) |
| Bar                       | [Waybar](https://github.com/Alexays/Waybar) | [Polybar](https://github.com/polybar/polybar) |
| Application Launcher      | [Anyrun](https://github.com/Kirottu/anyrun) | [Rofi](https://github.com/davatorium/rofi) |
| Notification Daemon       | [Mako](https://github.com/emersion/mako) | [Dunst](https://github.com/dunst-project/dunst) |
| Display Manager           | [GDM](https://wiki.archlinux.org/title/GDM) | [GDM](https://wiki.archlinux.org/title/GDM) |
| Colorscheme               | [Nord](https://www.nordtheme.com/) | [Nord](https://www.nordtheme.com/) |
| Network management tool   | [NetworkManager](https://wiki.gnome.org/Projects/NetworkManager) | [NetworkManager](https://wiki.gnome.org/Projects/NetworkManager) |
| Input method framework    | [Fcitx5](https://github.com/fcitx/fcitx5) | [Fcitx5](https://github.com/fcitx/fcitx5) |
| System resource monitor   | [Btop++](https://github.com/aristocratos/btop) | [Btop++](https://github.com/aristocratos/btop) |
| File Manager              | [Nemo](https://github.com/linuxmint/nemo) + [Yazi](https://github.com/sxyazi/yazi) | [Nemo](https://github.com/linuxmint/nemo) + [Yazi](https://github.com/sxyazi/yazi) |
| Shell                     | [Zsh](https://ohmyz.sh/) + [Starship](https://github.com/starship/starship) | [Zsh](https://ohmyz.sh/) + [Starship](https://github.com/starship/starship) |
| Music Player              | ? | ? |
| Media Player              | [mpv](https://github.com/mpv-player/mpv) | [mpv](https://github.com/mpv-player/mpv) |
| Text Editor               | [Neovim](https://github.com/neovim/neovim) + [Obsidian](https://obsidian.md/) | [Neovim](https://github.com/neovim/neovim) + [Obsidian](https://obsidian.md/) |
| Fonts                     | [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts) | [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts) |
| Image Viewer              | [imv](https://sr.ht/~exec64/imv/) | [imv](https://sr.ht/~exec64/imv/) |
| Screenshot Software       | [Flameshot](https://github.com/flameshot-org/flameshot) + [Grim](https://sr.ht/~emersion/grim/) | [Flameshot](https://github.com/flameshot-org/flameshot) |
| Screen recording          | [OBS](https://obsproject.com/) | [OBS](https://obsproject.com/) |
| Filesystem & Encryption   | [Btrfs](https://btrfs.readthedocs.io/en/latest/) with [btrbk](https://github.com/digint/btrbk) for snapshots | [Btrfs](https://btrfs.readthedocs.io/en/latest/) with [btrbk](https://github.com/digint/btrbk) for snapshots |
| Secure Boot               | [Grub](https://wiki.archlinux.org/title/GRUB) | [Grub](https://wiki.archlinux.org/title/GRUB) |
| Init system               | [Runit](https://wiki.archlinux.org/title/runit) | [Runit](https://wiki.archlinux.org/title/runit) |
| Initramfs build tool      | [Booster](https://github.com/anatol/booster) | [Booster](https://github.com/anatol/booster) |

Другие приложения, которые я использую:

- OnlyOffice
- Virt-Manager
- Zathura
- Kdenlive
- Bitwarden

## Процесс настройки

Прежде чем мы начнем настраивать Artix Linux, его надо скачать:

```sh
# Версия ISO файла может отличаться от вашей
xd --download https://download.artixlinux.org/iso/artix-base-runit-20230814-x86_64.iso
# Замените /dev/sdb на свое внешнее накопительное устройство
sudo bs=4M conv=fsync status=progress oflag=sync if=artix-base-runit-20230814-x86_64.iso of=/dev/sdb
```

### Подключение к Wi-Fi

```sh
rfkill unblock wifi
connmanctl
> scan wifi
> services
> agent on
> connect wifi_id
> exit
ping www.google.com -c 5
```

### Форматирование разделов

```sh
cfdisk /dev/sda # Нужно создать два раздела EFI Partition и root-раздел.
mkfs.fat -F 32 /dev/sda1
mkfs.btrfs -L root -f /dev/sda2
mount /dev/disk/by-label/artix /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@opt
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@docker
btrfs subvolume create /mnt/@libvirt
umount /mnt

mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@ /dev/disk/by-label/artix /mnt
mkdir -p /mnt/{home,opt,tmp,.snapshots,var/log,var/lib/docker,var/lib/libvirt,boot/efi}
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@home /dev/disk/by-label/artix /mnt/home
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@snapshots /dev/disk/by-label/artix /mnt/.snapshots
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@log /dev/disk/by-label/artix /mnt/var/log
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@opt /dev/disk/by-label/artix /mnt/opt
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@tmp /dev/disk/by-label/artix /mnt/tmp
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@docker /dev/disk/by-label/artix /mnt/var/lib/docker
mount -o rw,ssd,noatime,compress=zstd,space_cache=v2,commit=120,subvol=@libvirt /dev/disk/by-label/artix /mnt/var/lib/libvirt

mount /dev/sda1 /mnt/boot/efi

lsblk # Проверьте, что вы все сделали правильно
```

### Базовая установка

```sh
basestrap /mnt base base-devel runit elogind-runit
basestrap /mnt booster linux-zen linux-zen-headers linux-firmware
basestrap /mnt btrfs-progs intel-ucode iucode-tool # amd-ucode можно использовать вместо intel-ucode
basestrap /mnt dhcpcd dhclient networkmanager networkmanager-runit
basestrap /mnt grub efibootmgr mtools dosfstools os-prober ntfs-3g
basestrap /mnt zsh curl wget git zip unzip fzf ripgrep neovim

fstabgen -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab # Проверка на то, что все в порядке

artix-chroot /mnt
bash
```

### Настройка системы

```sh
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc

nvim /etc/locale.gen
curl -L "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/locale.conf" > /etc/locale.conf
locale-gen

echo 'username' > /etc/hostname
nvim /etc/hosts
# 127.0.0.1       localhost
# ::1             localhost
# 127.0.0.1       username.localdomain       username

curl -LO "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/install_arch_repos.sh"
curl -LO "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/pacman_without_arch.conf"
curl -LO "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/pacman_with_arch.conf"

chmod +x ./install_arch_repos.sh
./install_arch_repos.sh

rm -rf install_arch_repos.sh pacman_with_arch.conf pacman_without_arch.conf
```

### Настройка загрузчика

```sh
curl -LO "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/booster.yaml"
mv booster.yaml /etc/booster.yaml
cat /etc/booster.yaml # Не забудьте добавить специфичные для вашего сетапа модули
/usr/lib/booster/regenerate_images
mkdir /etc/pacman.d/hooks
curl -LO "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/booster-update.hook"
mv booster-update.hook /etc/pacman.d/hooks/booster-update.hook

nvim /etc/default/grub # Uncomment GRUB_DISABLE_OS_PROBER=false
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ARTIX --recheck
grub-mkconfig -o /boot/grub/grub.cfg
curl -LO "https://raw.githubusercontent.com/artphnix/dotfiles/main/artix/10_linux"
mv 10_linux /etc/grub.d/10_linux
grub-mkconfig -o /boot/grub/grub.cfg

git clone https://github.com/vinceliuice/grub2-themes.git
cd grub2-themes
./install.sh -b -t vimix -i white -s 1080p
cd .. && rm -rf grub2-themes

echo "LIBSEAT_BACKEND=logind" >> /etc/environment
```

### Создание пользователя и перезагрузка

```sh
passwd

useradd -m -g users -G wheel,storage,power -s /bin/zsh username
passwd username

EDITOR=nvim visudo # Add lines below
# %wheel ALL=(ALL:ALL) ALL
# %wheel ALL=(ALL:ALL) NOPASSWD: /usr/bin/reboot
# %wheel ALL=(ALL:ALL) NOPASSWD: /usr/bin/poweroff
# %wheel ALL=(ALL:ALL) NOPASSWD: /usr/bin/shutdown
# %wheel ALL=(ALL:ALL) NOPASSWD: /usr/bin/libinput

exit
exit
umount -R /mnt
reboot

ln -s /etc/runit/sv/NetworkManager /run/runit/service
ln -s /etc/runit/sv/elogind /run/runit/service
```

### Настройка pacman

```sh
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# pacman-static будет собираться ~5-10 минут
yay -S pacman-contrib pacman-static
sudo mkdir -p /etc/pacman.d/hooks

git https://github.com/artphnix/dotfiles.git

sudo mv dotfiles/artix/mirrorlist-ranking.hook /etc/pacman.d/hooks/
sudo mv dotfiles/artix/cache-cleanup.hook /etc/pacman.d/hooks/

sudo mkdir $HOME/scripts
mv dotfiles/artix/ranking-mirrors.sh $HOME/scripts/ranking-mirrors.sh
sudo chmod +x $HOME/scripts/ranking-mirrors.sh
```

### Настройка btrbk

```sh
# Создаем файлы, в которых будет храниться информация от btrbk
mkdir /.secret
touch /.secret/btrbk.log
touch /.secret/btrbk.lock

yay -S btrbk grub-btrfs

cp dotfiles/artix/btrbk-pre.hook /etc/pacman.d/hooks/btrbk-pre.hook
cp dotfiles/artix/grub-update.hook /etc/pacman.d/hooks/grub-update.hook
cp dotfiles/artix/btrbk.conf /etc/btrbk/btrbk.conf
```

### Настройка шрифтов

```sh
yay -S ttf-dejavu noto-fonts noto-fonts-cjk noto-fonts-emoji
curl -LO "https://github.com/ryanoasis/nerd-fonts/releases/download/v3.1.1/CascadiaCode.zip"
sudo mkdir -p /usr/share/fonts/CascadiaCode/
unzip CascadiaCode.zip -d cc
sudo mv cc/*.ttf /usr/share/fonts/CascadiaCode/

sudo fc-cache --force
```

### Настройка терминала и связанных приложений

```sh
yay -S alacritty doas tmux btop fd sd eza dust dog xh duf jq jqp bat
mkdir ~/.config/{alacritty,tmux}
mv dotfiles/terminal/alacritty.toml ~/.config/alacritty/alacritty.toml
mv dotfiles/terminal/tmux.conf ~/.config/tmux/tmux.conf
mv dotfiles/terminal/themes/* ~/.config/btop/themes/
mv dotfiles/artix/doas.conf /etc/doas.conf

curl -sS https://starship.rs/install.sh | sh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
mv dotfiles/terminal/.zshrc ~/.zshrc
mv dotfiles/terminal/starship.toml ~/.config/starship.toml
```

## Установка драйверов

### Аудио

```sh
yay -Rs pipewire-session-manager
yay -S pipewire-alsa pipewire pipewire-jack pipewire-pulse pipewire-media-session pamixer pulsemixer
```

### Bluetooth

```sh
yay -S bluez bluez-utils bluez-runit bluetuith
sudo ln -s /etc/runit/sv/bluetoothd /run/runit/service/
sudo sv restart bluetoothd
```

### Управление энергосбережением

```sh
yay -S tlp tlp-runit tlpui
sudo ln -s /etc/runit/sv/tlp /run/runit/service
```

### Поддержка принтеров

```sh
yay -S avahi cups cups-pdf cups-runit
sudo ln -s /etc/runit/sv/cupsd /run/runit/service
```

### Nvidia

```sh
# Новые драйвера
yay -S nvidia-dkms nvidia-utils nvidia-settings opencl-nvidia vulkan-icd-loader
yay -S lib32-opencl-nvidia lib32-vulkan-icd-loader lib32-nvidia-utils

# Старын драйвера ниже 390
yay -S nvidia-390xx nvidia-390xx-settings nvidia-390xx-utils opencl-nvidia-390xx libvdpau libxnvctrl-390xx vulkan-icd-loader
yay -S lib32-nvidia-390xx-utils lib32-opencl-nvidia-390xx lib32-vulkan-icd-loader

sudo nvim /etc/booster.yaml # Add "nvidia,nvidia_modeset,nvidia_uvm,nvidia_drm"
sudo nvim /etc/default/grub # Add "nvidia_drm.modeset=1" into GRUB_CMDLINE_LINUX
```

### Настройка драйверов для использования Qemu+KVM

```sh
yay -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat dmidecode
yay -S ebtables iptables-nft
yay -S libguestfs
yay -S libvirt-runit
sudo ln -s /etc/runit/sv/libvirtd /run/runit/services

sudo nvim /etc/libvirt/libvirtd.conf # Добавьте линии ниже
# unix_sock_group = "libvirt"
# unix_sock_rw_perms = "0770"

sudo usermod -aG libvirt $(whoami)
newgrp libvirt
sudo sv restart libvirtd

### Intel Processor ###
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1

### AMD Processor ###
sudo modprobe -r kvm_amd
sudo modprobe kvm_amd nested=1

echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```

### Настройка раздела подкачки

```sh
yay -S zramen zramen-runit
sudo ln -s /etc/runit/sv/zramen /run/runit/service
```

### Включаем поддержку NTFS устройств

```sh
sudo nvim /etc/modprobe.d/disable-ntfs3.conf # Add 'blacklist ntfs3'
```

### Настройка системных папок

```sh
xdg-mime default nemo.desktop inode/directory application/x-gnome-saved-search
gsettings set org.cinnamon.desktop.default-applications.terminal exec alacritty
yay -S xdg-utils xdg-user-dirs
mkdir ~/Share ~/Download ~/Documents ~/Media ~/Templates
xdg-user-dirs-update --set DESKTOP ~/Media
xdg-user-dirs-update --set DOCUMENTS ~/Documents
xdg-user-dirs-update --set DOWNLOAD ~/Download
xdg-user-dirs-update --set MUSIC ~/Media
xdg-user-dirs-update --set PICTURES ~/Media
xdg-user-dirs-update --set PUBLICSHARE ~/Share
xdg-user-dirs-update --set TEMPLATES ~/Templates
xdg-user-dirs-update --set VIDEOS ~/Media
mkdir ~/Repos
mkdir ~/Applications
```

## Настрока Artix Linux(Xorg)

```sh
yay -S xorg xorg-xinit xclip gdm gdm-runit # Установка Xorg и gdm
mkdir /etc/X11/xorg.conf.d/
sudo cp dotfiles/xorg/xorg.conf.d/* /etc/X11/xorg.conf.d/ # Чтобы работала переферия
sudo cp dotfiles/artix/gdm.conf /etc/gdm/custom.conf # Автологин после запуска системы
yay -S i3-wm polybar rofi picom-ftlabs-git dunst feh nemo yazi mpv imv flameshot betterlockscreen

# Установка конфигов для программ
mv dotfiles/xorg/i3/scripts/* ~/scripts/
sudo chmod +x ~/scripts/*
mkdir ~/.config/{i3,picom,polybar,rofi,dunst}
mv dotfiles/xorg/i3/config ~/.config/i3/config
mv dotfiles/xorg/picom/picom.conf ~/.config/picom/picom.conf
mv dotfiles/xorg/polybar/* ~/.config/polybar/
mv dotfiles/xorg/rofi/* ~/.config/rofi/
mv dotfiles/xorg/dunst/dunstrc ~/.config/dunst/dunstc

# Обои и запуск gdm
mv dotfiles/wallpaper.png ~/wallpaper.png
betterlockscreen -u wallpaper.png
sudo ln -s /etc/runit/sv/gdm /run/runit/service
```

## Настрока Artix Linux(Wayland)

Скоро будет ...

## Установка темы

```sh
# Установка иконок
yay -S numix-circle-icon-theme-git numix-cursor-theme-git
yay -S lxappearance qt5ct kvantum
echo "QT_QPA_PLATFORMTHEME=qt5ct" >> /etc/environment
nvim ~/.config/gtk-3.0/settings.ini # Установите тему курсора
nvim ~/.icons/default/index.theme # Установите тему курсора

yay -S nordic-theme-git kvantum-theme-nordic-git
# Далее используя lxappearance qt5ct и kvantum просто установите установленные курсор, иконки и темы
```

## Установка приложений

```sh
yay -S obsidian obs-studio ark nekoray

yay -S flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install flathub org.onlyoffice.desktopeditors
flatpak install flathub com.bitwarden.desktop
```
