<p align="center">
  <a href="#ru"><strong>Русский</strong></a> ·
  <a href="#en"><strong>English</strong></a>
</p>

<details id="ru" open>
<summary><strong>Русский</strong></summary>

# arch-courbet

Эксперимент по запуску **Arch Linux ARM** на Xiaomi Mi 11 Lite 4G (`courbet`, SM7150 / Snapdragon 732G). Конечная цель — получить на самом телефоне настоящий Arch userspace, графическую сессию и терминал, в котором можно выполнить `fastfetch`.

> Это ранний bring-up, а не готовая прошивка. На текущем этапе рабочий экран — это консоль ядра; рабочий стол ещё не запускался.

## Текущее состояние

### Что проверено

- На `courbet` загружается mainline Linux 6.17 с отдельным DTB устройства.
- Временная загрузка через `fastboot boot` показывает консоль на дисплее и даёт serial-shell через USB ACM.
- Пользователь подтвердил с телефона `uname -a`: `aarch64`, ядро `6.17.0-courbet+`.
- Пользовательская проверка `/etc/os-release` в текущем shell показала **Alpine Linux 3.21.9**. В этой среде нет `pacman`, `phoc`, `weston` или `fastfetch`.
- Ранее подготовленный `boot-arch-final.img` содержал Arch release metadata, но не полноценный Arch userspace. Этот файл не является Arch-системой и не должен использоваться как desktop-образ.
- Устройство не было проверено с Wayland-композитором. Работа GPU acceleration и сенсорного ввода в графической сессии не подтверждена.

Иными словами, пока работает **ядро и bring-up userspace**, а не Arch на телефоне. Один `os-release` или баннер в initramfs не считается доказательством установленного дистрибутива.

### Отдельный kernel bring-up

Основное дерево ядра, display-патчи и проверенные временные boot-образы находятся в [qualixxe/mainline-courbet](https://github.com/qualixxe/mainline-courbet). В том проекте на устройстве проверялись экранная консоль и serial-shell; его Alpine-based pmOS образ — отдельный вариант, не Arch.

## Цель: Arch с рабочим столом

Планируемая последовательность:

1. Подготовить настоящий generic Arch Linux ARM AArch64 rootfs и проверить его содержимое (`/usr/bin/pacman`, `/usr/bin/bash`, `/etc/arch-release`, `/etc/os-release`).
2. Размещать rootfs отдельно от boot image: архив занимает около 829 MB, распакованная система — несколько гигабайт, тогда как boot-раздел устройства имеет размер 128 MiB.
3. Настроить initramfs для ожидания UFS, обнаружения целевой файловой системы, монтирования rootfs и корректного `switch_root`.
4. Сначала загрузить систему временно и проверить на телефоне `cat /etc/os-release`, `pacman --version`, `uname -m`, serial login и вывод терминала на экран.
5. Проверить DRM/KMS nodes и input devices. Начать с Weston и терминала `foot`; затем оценить сенсорное управление и, если драйверы позволяют, мобильную оболочку Phosh/Phoc.
6. Проверить сеть, установку пакетов и повторяемость холодной загрузки. Только после этого решать вопрос постоянной установки.

Ни один из этих пунктов не следует считать выполненным лишь потому, что соответствующие пакеты доступны для AArch64: конкретная совместимость с `courbet` должна быть проверена на устройстве.

## Данные и безопасная загрузка

По выводу fastboot раздел `userdata` имеет тип F2FS и размер `0x1AC4BFB000` (около 107 GiB). Это пользовательские данные телефона, а не свободный раздел для экспериментов. Форматирование, разметка или запись rootfs туда уничтожит существующее содержимое.

- Не форматировать и не изменять `userdata` без проверенной резервной копии и отдельного согласованного плана.
- Для ранних испытаний предпочитать `fastboot boot`, который загружает образ временно, вместо `fastboot flash`.
- Не делать вывод, что UFS отсутствует, только по неудачному пути `/dev/block/by-name/userdata`: имена устройств зависят от Linux DT и udev/initramfs.
- Не выдавать временный initramfs за persistent installation.

Для первого desktop-прототипа предпочтителен обратимый путь: внешний носитель или иной отдельный rootfs, который можно удалить без изменения Android-раздела.

## Устройство и известные ограничения

| Компонент | Состояние |
| --- | --- |
| Устройство | Xiaomi Mi 11 Lite 4G (`courbet`, M2101K9AG) |
| SoC | Qualcomm SM7150 / Snapdragon 732G |
| Панель | K9A DSI |
| Kernel baseline | Linux 6.17 bring-up из связанного проекта |
| Экранная консоль | Проверена |
| USB serial shell | Проверена ранее через USB ACM / COM8 |
| Текущий userspace на телефоне | Alpine Linux 3.21.9, не Arch |
| Arch rootfs на устройстве | Не загружен |
| `pacman` и `fastfetch` на устройстве | Не проверены как работающие; текущая Alpine-среда их не содержит |
| Wayland / Weston / Phosh | Не запускались |
| Сенсор в desktop-сеансе | Не проверен |
| Persistent install | Не выполнялась |

Kernel config в связанном дереве включает UFS/QCOM, F2FS, ext4, DRM MSM и framebuffer console; это конфигурация сборки, а не доказательство, что каждый из этих компонентов успешно поднялся в Arch desktop boot.

## Связанные проекты и источники

- [Mainline kernel / display bring-up для courbet](https://github.com/qualixxe/mainline-courbet)
- [Arch Linux ARM](https://archlinuxarm.org/)
- [Arch Linux ARM rootfs downloads](https://archlinuxarm.org/about/downloads)
- [postmarketOS device notes: Xiaomi Mi 11 Lite 4G](https://wiki.postmarketos.org/wiki/Xiaomi_Mi_11_Lite_4G_(xiaomi-courbet))

</details>

<details id="en">
<summary><strong>English</strong></summary>

# arch-courbet

An experiment to bring **Arch Linux ARM** to the Xiaomi Mi 11 Lite 4G (`courbet`, SM7150 / Snapdragon 732G). The goal is a real Arch userspace on the phone, followed by a graphical session and a terminal where `fastfetch` can run.

> This is early bring-up work, not a ready-to-install image. The working display currently shows the kernel console; a desktop has not been started.

## Current status

### Verified

- Mainline Linux 6.17 boots on `courbet` with a device DTB.
- A temporary `fastboot boot` image reaches the on-device console and exposes a USB ACM serial shell.
- The user confirmed `uname -a` on the phone: `aarch64`, kernel `6.17.0-courbet+`.
- The user's `/etc/os-release` output from the running shell reported **Alpine Linux 3.21.9**. That environment has no `pacman`, `phoc`, `weston`, or `fastfetch`.
- The earlier `boot-arch-final.img` had Arch release metadata but did not contain a complete Arch userspace. It is not an Arch system or a desktop image.
- No Wayland compositor has been tested on the phone. GPU acceleration and touchscreen input in a graphical session are unverified.

The current result is **kernel/device bring-up**, not Arch running on the phone. An `os-release` file or a boot banner in an initramfs is not proof of an installed distribution.

### Related kernel work

The kernel tree, display patches, and tested temporary boot images live in [qualixxe/mainline-courbet](https://github.com/qualixxe/mainline-courbet). That project verified the display console and serial shell. Its Alpine-based pmOS image is a separate variant, not Arch.

## Desktop roadmap

1. Prepare a genuine generic Arch Linux ARM AArch64 rootfs and verify its files (`/usr/bin/pacman`, `/usr/bin/bash`, `/etc/arch-release`, `/etc/os-release`).
2. Keep the rootfs outside the boot image: the archive is about 829 MB and expands to several gigabytes, while the device boot partition is 128 MiB.
3. Extend initramfs to wait for UFS, discover and mount the target filesystem, and perform a correct `switch_root`.
4. Boot temporarily first. On-device checks should include `cat /etc/os-release`, `pacman --version`, `uname -m`, serial login, and a terminal rendered on the phone display.
5. Check DRM/KMS nodes and input devices. Start with Weston and the `foot` terminal; then test touch and consider Phosh/Phoc if the drivers support it.
6. Verify networking, package installation, and repeatable cold boots before considering a persistent installation.

A package being available for AArch64 does not establish that it works on `courbet`; each component still needs on-device testing.

## Data and boot safety

Fastboot reports `userdata` as F2FS with size `0x1AC4BFB000` (about 107 GiB). This is the phone's user-data partition, not spare space. Formatting or writing a rootfs there would destroy existing contents.

- Do not format or modify `userdata` without a verified backup and a separately agreed installation plan.
- Prefer `fastboot boot` for early tests instead of `fastboot flash`.
- A missing `/dev/block/by-name/userdata` path alone does not prove UFS is unavailable; device names depend on the Linux DT and initramfs/udev setup.
- Do not describe a temporary initramfs as a persistent installation.

An external device or another separate root filesystem is preferable for the first desktop prototype because it can be removed without altering Android partitions.

## Device and known limitations

| Component | Status |
| --- | --- |
| Device | Xiaomi Mi 11 Lite 4G (`courbet`, M2101K9AG) |
| SoC | Qualcomm SM7150 / Snapdragon 732G |
| Panel | K9A DSI |
| Kernel baseline | Linux 6.17 bring-up from the related project |
| Display console | Verified |
| USB serial shell | Previously verified via USB ACM / COM8 |
| Current phone userspace | Alpine Linux 3.21.9, not Arch |
| Arch rootfs on device | Not booted |
| `pacman` and `fastfetch` on device | Not verified working; absent from the current Alpine environment |
| Wayland / Weston / Phosh | Not started |
| Touch in desktop session | Not tested |
| Persistent install | Not performed |

The related kernel config enables UFS/QCOM, F2FS, ext4, DRM MSM, and framebuffer console. That describes the build configuration; it does not prove each component is operational in an Arch desktop boot.

## Related projects and sources

- [Mainline kernel / display bring-up for courbet](https://github.com/qualixxe/mainline-courbet)
- [Arch Linux ARM](https://archlinuxarm.org/)
- [Arch Linux ARM rootfs downloads](https://archlinuxarm.org/about/downloads)
- [postmarketOS device notes: Xiaomi Mi 11 Lite 4G](https://wiki.postmarketos.org/wiki/Xiaomi_Mi_11_Lite_4G_(xiaomi-courbet))

</details>
