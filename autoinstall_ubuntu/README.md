<!-- autoinstall_ubuntu/README.md -->

# The simplest Ubuntu autoinstaller

## 1. Intro

## 1. How does autoinstall work?

Automated installation of Ubuntu is driven by the autoinstall config documented at https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html.

The autoinstall config can be provided to the installer in different ways described at https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/providing-autoinstall.html.

One way is via a file named `autoinstall.yaml` placed in the root directory of the installer ISO.

`autoinstall.yaml` is a file in YAML format. It contains settings for the installer to use in lieu of user input from interactive screens. The installation is fully automated if `autoinstall.yaml` has all settings the installer needs. Alternatively `autoinstall.yaml` may be used to only populate some installer screens for a partly automated install in which some screens are skipped while others will require user input.

For example, the following YAML section sets the system locale to `fr_FR.UTF-8`

```yaml
autoinstall:
  version: 1

  locale: fr_FR.UTF-8
```


If an `interactive-sections` object containing `locale` is added to `autoinstall.yaml`, the installer will no longer be fully automated. The `Welcome language` UI screen will be shown though in French. If the user doesn't change the language, the installer will continue in French.

```yaml
autoinstall:
  version: 1

  interactive-sections:
    - locale

  locale: fr_FR.UTF-8
```

![french_welcome_language.png](images/french_welcome_language.png)

## 2. Ubuntu installer screens

These screens are from the Ubuntu 24.04 server installer ISO at https://ubuntu.com/download/server. The installer was run interactively to install Ubuntu 24.04.1.

## Screen 1 - Try or Install Ubuntu Server

![01_try_install.png](images/01_try_install.png)

## Screen 2 - Installer init

![02_installer_init_message.png](images/02_installer_init_message.png)

## Screen 3 - Welcome language

![03_welcome_language.png](images/03_welcome_language.png)

## Screen 4 - Keyboard configuration

![04_keyboard.png](images/04_keyboard.png)

## Screen 5 - Choose the type of installation

![05_install_type.png](images/05_install_type.png)

## Screen 6 - Network configuration

![06_network.png](images/06_network.png)

## Screen 7 - Proxy configuration

![07_proxy.png](images/07_proxy.png)

## Screen 8 - Ubuntu archive mirror configuration test

![8_apt_config.png](images/8_apt_config.png)

## Screen 9 - Ubuntu archive mirror configuration passed

![9_apt_mirror_success.png](images/9_apt_mirror_success.png)

## Screen 10 - Guided storage configuration

![10_disk_config.png](images/10_disk_config.png)

## Screen 11 - Storage configuration

![11_disk_summary.png](images/11_disk_summary.png)

## Screen 12 - Confirm destructive action

![12_start_install.png](images/12_start_install.png)

## Screen 13 - Profile configuration

![13_user_account.png](images/13_user_account.png)

## Screen 14 - Installing system

![14_installing_-1_message.png](images/14_installing_1_message.png)

## Screen 15 - Upgrade to Ubuntu Pro

![15_ubuntu_pro.png](images/15_ubuntu_pro.png)

## Screen 16 - SSH configuration

![16_ssh.png](images/16_ssh.png)

## Screen 17 - Featured server snaps

![17_snap.png](images/17_snap.png)

## Screen 18 - installing system

![18_installing_2_message.png](images/18_installing_2_message.png)

## Screen 19 - Updating system

![19_updating_system_message.png](images/19_updating_system_message.png)

## Screen 20 - Installation complete

![20_install_complete.png](images/20_install_complete.png)

## Screen 21 - Failed unmounting cdrom.mount

![21_unmount_cdrom.png](images/21_unmount_cdrom.png)

## Screen 22 - Reboot

![22_reboot_message.png](images/22_reboot_message.png)

## Screen 23 - Login

![23_login.png](images/23_login.png)


