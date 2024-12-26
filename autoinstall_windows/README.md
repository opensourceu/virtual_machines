<!-- autoinstall_windows/README.md -->

# How to make a Windows autoinstaller

## 1. How does autoinstall work?

The main ingredient of a Windows autoinstaller is a file called the answer file. The answer file may be provided to the installer in different ways documented at https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-automation-overview#implicit-answer-file-search-order.

One way is to name it `autounattend.xml` and place it in the root directory of the installer ISO.

`autounattend.xml` an XML file with settings to automate a Windows installation. For example, the following XML snippet sets the hostname of the computer to `winauto1`. Its presence makes the installer not display Screen 28 below titled **`Let's name your device`**.

```xml
<settings pass="specialize">
  <component name="Microsoft-Windows-Shell-Setup">
    <ComputerName>winauto1</ComputerName>
  </component>
</settings>
```

Windows installation is fully automated by setting values for all installer screens in `autounattend.xml`

## 1. Windows installer screens

### Screen 1 - Select language settings

![01_setup_language.png](images/01_setup_language.png)

Screen 1 selects the language for the installer. The only language allowed is the one selected when the installer ISO was downloaded.
Screen 1 allows a choice of locales for time and currency format.

![01_setup_time_locale.png](images/01_setup_time_locale.png)

Screen 1 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-International-Core-WinPE">
    <UILanguage>en-US</UILanguage>
    <UILanguageFallback>en-US</UILanguageFallback>
    <UserLocale>en-US</UserLocale>
    <SystemLocale>en-US</SystemLocale>
    <SetupUILanguage>
      <UILanguage>en-US</UILanguage>
    </SetupUILanguage>
  </component>
</settings>
```

### Screen 2 - Select keyboard settings

![02_setup_keyboard.png](images/02_setup_keyboard.png)

Screen 2 selects the keyboard.

![02_setup_keyboard_choice.png](images/02_setup_keyboard_choice.png)

Screen 2 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-International-Core-WinPE">
    <InputLocale>en-US</InputLocale>
  </component>
</settings>
```

### Screen 3 - Select setup option

![03_install_repair.png](images/03_install_repair.png)

Screen 3 selects whether to install or repair Windows

There's an option to switch to the old installer shown below. There's no way to switch back to the new installer.

![03_old_installer.png](images/03_old_installer.png)

Screen 3 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
</settings>
```

### Screen 4 - Product key

![04_product_key.png](images/04_product_key.png)

Screen 4 sets the product key. There's an option to proceed without a product key.

Screen 4 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-Setup">
    <UserData>
      <ProductKey>
        <Key></Key>
      </ProductKey>
    </UserData>
  </component>
</settings>
```

### Screen 5 - We're getting a few things ready

![05_getting_ready_1_message.png](images/05_getting_ready_1_message.png)

Screen 5 is an informational message.

### Screen 6 - Select image

![06_os_image.png](images/06_os_image.png)

Screen 6 selects the version of Windows to install.

Screen 6 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-Setup">
    <ImageInstall>
      <OSImage>
        <InstallFrom>
          <MetaData wcm:action="add">
            <Key>/image/name</Key>
            <Value>Windows 11 Pro</Value>
          </MetaData>
        </InstallFrom>
      </OSImage>
    </ImageInstall>
  </component>
</settings>
```

### Screen 7 - Select setup option

![07_license.png](images/07_license.png)

Screen 7 selects whether to install or repair Windows.

Screen 7 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 8 - Please wait

![08_wait_message.png](images/08_wait_message.png)

Screen 8 is an informational message.

### Screen 9 - Searching for disks

![09_disk_search_message.png](images/09_disk_search_message.png)

Screen 9 is an informational message.

### Screen 10 - Select location to install Windows 11

An empty disk
![10_disk_partition.png](images/10_disk_partition.png)

A disk partitioned by a previous run of the installer
![10_partitioned_disk.png](images/10_partitioned_disk.png)

Screen 10 selects the disk partition to install Windows. There are options to manage disks and partitions.

Screen 10 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 11 - Ready to install

![11_ready_install.png](images/11_ready_install.png)

Screen 11 has a button to begin the install.

### Screen 12 - Installing Windows 11

![12_installing_message.png](images/12_installing_message.png)

Screen 12 is an informational message.

### Screen 13 - Installing progress

![13_install_progress_message.png](images/13_install_progress_message.png)

Screen 13 is an informational message.

### Screen 14 - Just a moment

![14_moment_1_message.png](images/14_moment_1_message.png)

Screen 14 is an informational message.

### Screen 15 - Windows logo

![15_winlogo_1_message.png](images/15_winlogo_1_message.png)

Screen 15 is an informational message.

### Screen 16 - Is this the right country or region?

![16_system_locale_1.png](images/16_system_locale_1.png)

Screen 16 selects the system locale.

Screen 16 marks the end of the WinPE setup stage of the installer and the beginning of the OOBE stage of the isntaller.

The option to create a local Windows account instead of using a Microsoft online account can be enabled at this point by pressing Shift-F10 to launch a `cmd` terminal window.

### Screen 17 - Terminal window

![17_cmd_terminal.png](images/17_cmd_terminal.png)

Screen 17 is a `cmd` terminal launched with Shift-F10

### Screen 18 - Run `oobe\bypassnro`

![18_bypassnro.png](images/18_bypassnro.png)

Screen 18 shows the `oobe\bypassnro` being run. Running the command makes the installer relaunch the first OOBE screen to set the system locale

### Screen 19 - Just a moment

![19_moment_2_message.png](images/19_moment_2_message.png)

Screen 19 is an informational message.

### Screen 20 - Windows logo

![20_winlogo_2_message.png](images/20_winlogo_2_message.png)

Screen 20 is an informational message.

### Screen 21 - Is this the right country or region?

![21_system_locale_2.png](images/21_system_locale_2.png)

Screen 21 selects the system locale.

Screen 21 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 22 - Is this the right keyboard layout or input method?

![22_system_keyboard.png](images/22_system_keyboard.png)

Screen 22 selects the keyboard.

Screen 22 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 23 - Want to add a second keyboard layout?

![23_second_keyboard.png](images/23_second_keyboard.png)

Screen 23 selects a second keyboard.

Screen 23 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 24 - Checking for updates

![24_check_update_1_message.png](images/24_check_update_1_message.png)

Screen 24 is an informational message.

### Screen 25 - Good things are coming your way

![25_good_things_message.png](images/25_good_things_message.png)

Screen 25 is an informational message.

### Screen 26 - your PC will restart before you continue

![25_restart_1_message.png](images/25_restart_1_message.png)

Screen 26 is an informational message.

### Screen 27 - Welcome

![27_welcome_message.png](images/27_welcome_message.png)

Screen 27 is an informational message.

### Screen 28 - Let's name your device

![28_hostname.png](images/28_hostname.png)

Screen 28 sets the system hostname.

Screen 28 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 29 - Just a moment

![29_moment_3_message.png](images/29_moment_3_message.png)

Screen 29 is an informational message.

### Screen 30 - How would you like to set up this device?

![30_personal_use.png](images/30_personal_use.png)

Screen 30 selects whether the system is personal or for work.

Screen 30 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 31 - Checking for Windows updates

![31_check_update_2_message.png](images/31_check_update_2_message.png)

Screen 31 is an informational message.

### Screen 32 - Windows update - Step 1 of 3: Downloading

![32_update_step_1_message.png](images/32_update_step_1_message.png)

Screen 32 is an informational message.

### Screen 33 - Windows update - Step 2 of 3: Installing

![33_update_step_2_message.png](images/33_update_step_2_message.png)

Screen 33 is an informational message.

### Screen 34 - Windows update - Step 3 of 3: Preparing to restart

![34_update_step_3_message.png](images/34_update_step_3_message.png)

Screen 34 is an informational message.

### Screen 35 - Restarting

![35_restart_2_message.png](images/35_restart_2_message.png)

Screen 35 is an informational message.

### Screen 36 - Updates are underway

![36_update_underway_message.png](images/36_update_underway_message.png)

Screen 36 is an informational message.

### Screen 37 - Updates progress

![37_update_progress_message.png](images/37_update_progress_message.png)

Screen 37 is an informational message.

### Screen 38 - Welcome

![38_welcome_message.png](images/38_welcome_message.png)

Screen 38 is an informational message.

### Screen 39 - Unlock your Microsoft experience

![39_microsoft_signin.png](images/39_microsoft_signin.png)

Screen 39 has a Sign in button.

Screen 39 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 40 - Who's going to use this device?

![40_username.png](images/40_username.png)

Screen 40 sets a local account username. This screen is enabled by running `oobe\bypassnro` in Screen 18.

Screen 40 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 41 - Create a super memorable password

![41_password.png](images/41_password.png)

Screen 41 sets the user password.

Screen 41 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 42 - Confirm your password

![42_confirm_password.png](images/42_confirm_password.png)

Screen 42 asks to reenter the password.

Screen 42 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 43 - Now add security questions

![43_security_question.png](images/43_security_question.png)

![43_security_question_dropdown.png](images/43_security_question_dropdown.png)

Screen 43 selects the first of 3 security questions

Screen 43 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 44 - Choose privacy settings for your device

![44_privacy_1.png](images/44_privacy_1.png)

![44_privacy_2.png](images/44_privacy_2.png)

![44_privacy_3.png.png](images/44_privacy_3.png.png)

Screen 44 selects privacy settings.

Screen 44 settings are set by the following section of `autounattend.xml`.

```xml
```

### Screen 45 - Just a moment

![45_moment_4_message.png](images/45_moment_4_message.png)

Screen 45 is an informational message.

### Screen 46 - Hi

![46_hi_message.png](images/46_hi_message.png)

Screen 46 is an informational message.

### Screen 47 - Getting things ready for you

![47_getting_ready_2_message.png](images/47_getting_ready_2_message.png)

Screen 47 is an informational message.

### Screen 48 - This might take a few minutes

![48_few_minutes_message.png](images/48_few_minutes_message.png)

Screen 48 is an informational message.

### Screen 49 - Login

![49_login.png](images/49_login.png)

Screen 49 is the Windows login screen marking the end of the installation.

