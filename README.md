# Instalar o Arch Linux na mão

## PRÉ-INSTALAÇÃO

Configura o teclado do layout da ISO

```
# loadkeys br-abnt2
```

Verifica o status da internet na ISO

```
# ip link
```

Atualiza o relógio da ISO

```
# timedatectl
```

## PARTICIONAMENTO

Lista os discos

```
# fdisk -l
```

Gerenciar Partições

```
# cfdisk /dev/sda
```

Layout das partições

```
| PARTIÇÃO  | TAMANHO | FORMATO | PONTO DE MONTAGEM | BOOT | TIPO             |
|-----------|---------|---------|-------------------|------|------------------|
| /dev/sda1 | 512MiB  | FAT32   | /boot/efi         | Sim  | EFI System       |
| /dev/sda2 | 1G      | EXT4    | /boot             | Não  | Linux Filesystem |
| /dev/sda3 | 8G      | Swap    | -                 | Não  | Linux Swap       |
| /dev/sda4 | Maior % | EXT4    | /                 | Não  | Linux Filesystem |
| /dev/sda5 | Médio % | EXT4    | /home             | Não  | Linux Filesystem |
```

## FORMATANDO

Formatando a partição BOOT:

```
# mkfs.ext4 /dev/sda2
```

Formatando a partição UEFI:

```
# mkfs.fat -F 32 /dev/sda1
```

Formatando a partição SWAP:

```
# mkswap /dev/sda3
```

Formatando a partição ROOT:

```
# mkfs.ext4 /dev/sda4
```

Formatando a partição HOME:

```
# mkfs.ext4 /dev/sda5
```

## MONTANDO AS PARTIÇÕES

Montagem do ROOT:

```
# mount /dev/sda4 /mnt
```

Montagem do HOME:

```
# mount --mkdir /dev/sda5 /mnt/home
```

Montagem do BOOT:

```
# mount --mkdir /dev/sda2 /mnt/boot
```

Montagem do UEFI:

```
# mount --mkdir /dev/sda1 /mnt/boot/efi
```

Habilitando o SWAP:

```
# swapon /dev/sda3
```

## CONFIGURAÇÃO DOS ESPELHOS (SERVIDOR DE PACOTE)

Escrever as configurações do mirrorlist (Opção 1)

```
# echo "# Brazil
Server = http://br.mirror.archlinux-br.org/$repo/os/$arch
#Server = http://archlinux.c3sl.ufpr.br/$repo/os/$arch
#Server = http://www.caco.ic.unicamp.br/archlinux/$repo/os/$arch
#Server = https://www.caco.ic.unicamp.br/archlinux/$repo/os/$arch
#Server = http://linorg.usp.br/archlinux/$repo/os/$arch
#Server = http://archlinux.pop-es.rnp.br/$repo/os/$arch
#Server = http://mirror.ufam.edu.br/archlinux/$repo/os/$arch
#Server = http://mirror.ufscar.br/archlinux/$repo/os/$arch
#Server = https://mirror.ufscar.br/archlinux/$repo/os/$arch" > /etc/pacman.d/mirrorlist
```

Escrever as configurações do mirrorlist (Opção 2)

```
# curl -sL "https://archlinux.org/mirrorlist/?country=BR&protocol=http&protocol=https&ip_version=4&ip_version=6" > /etc/pacman.d/mirrorlist
```

Abrir o arquivo mirrorlist (Opção 2) (retirar apenas 1 hashtag de cada linha)

```
nano /etc/pacman.d/mirrorlist
```

Liberando o download paralelo (descomentar o ParallelDownloads = 5) (Colocar o número que desejar)

```
nano /etc/pacman.conf
```

Instalação do sistema base e pacotes adicionais

```
pacstrap -K /mnt base linux base-devel linux-firmware cinnamon nano grub efibootmgr lightdm lightdm-slick-greeter firefox gnome-terminal gnome-system-monitor gnome-calculator inkscape gimp gnome-disk-utility git sudo
```

## CONFIGURAÇÃO DO SISTEMA

Gerar tabela de partições

```
genfstab -U /mnt >> /mnt/etc/fstab
```

Entrar no sistema recém instalado

```
arch-chroot /mnt
```

Definir senha raiz

```
passwd
```

Configuração do rede

```
echo {hostname} > /etc/hostname
```

Habilita a rede do sistema

```
systemctl enable NetworkManager.service
```

Initramfs (Opcional)

```
# mkinitcpio -P
```

Instalando inicializador (GRUB)

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

Configurando o inicializador (GRUB)

```
grub-mkconfig -o /boot/grub/grub.cfg
```

Configurando a timezone

```
ln -sf /usr/share/zoneinfo/America/Bahia /etc/localtime
```

Configurando o relógio/horário

```
hwclock --systohc
```

Configurando idioma e teclado

```
echo "pt_BR.UTF-8 UTF-8" >> /etc/locale.gen
echo LANG="pt_BR.UTF-8" >> /etc/locale.conf
echo KEYMAP="br-abnt2" >> /etc/vconsole.conf
locale-gen
```

### Configuração do ambiente gráfico

Edite o arquivo `/etc/lightdm/lightdm.conf`

```
[Seat:*]
...
greeter-session=lightdm-slick-greeter
...
```

Habilitando LightDM

```
systemctl enable lightdm
```


## CRIANDO UM USUÁRIO

Administrador

```
groupadd sudo
useradd -m -s /bin/bash -G sudo {name_user}
passwd {name_user}
```

Edite o arquivo `/etc/sudoers`

```
#%sudo -> %sudo
```

## INSTAÇÃO DO YAY (AUR)

```
# su {name_user} -

$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
$ cd ..
$ rm -rf yay
```


## REBOOT

```
exit/CTRL+D
umount -R /mnt
reboot
```

