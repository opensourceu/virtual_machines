<!-- autoinstall_windows/README.md -->

# How to make a Windows autoinstaller

## 1. How does autoinstall work?

The main ingredient of a Windows autoinstaller is the answer file. The answer file may be provided to the installer in different ways documented at https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-automation-overview#implicit-answer-file-search-order.

One way is to name it `autounattend.xml` and place it in the root directory of the installer ISO.

`autounattend.xml` is an XML file with settings to automate a Windows installation. For example, the following XML section sets the hostname of the computer to `winauto1`. It makes the installer not display Screen 28 below titled **`Let's name your device`**.

```xml
<settings pass="specialize">
  <component name="Microsoft-Windows-Shell-Setup">
    <ComputerName>winauto1</ComputerName>
  </component>
</settings>
```

Windows installation is fully automated by setting values in `autounattend.xml` for all installer screens and removing the need for interactive user input.

Some installer screens only display informational or progress messages and don't require user input. These screens still appear in an unattended install.

## 2. A complete minimal answer file for an unattended install

The file [autounattend.xml](autounattend.xml) is a nearly minimal answer file to automatically install Windows 11 Pro 24H2 version 10.0.26100

## 2. What's needed to create a Windows autoinstaller

1. An official Windows installer ISO
   - Download from https://www.microsoft.com/en-us/software-download/windows11
2. Windows Assessment and Deployment Kit (ADK)
   - Download from https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install

## 3. High-level steps to creating a Windows autoinstaller

1. Mount the official Windows ISO. Copy its contents to a directory for making modifications
2. Extract `install.wim` from the ISO directory
3. Create an answer file for `install.wim` in Windows System Image Manager from the ADK
4. Copy answer file to the ISO directory
5. Create a new autoinstaller ISO file from the ISO directory

## 4. Step 1 - Mount and copy Windows ISO

Mount the ISO and note the drive

```powershell
mount-diskimage Win11_24H2_English_x64.iso | get-volume
```

```
DriveLetter FriendlyName            FileSystemType DriveType HealthStatus OperationalStatus SizeRemaining
----------- ------------            -------------- --------- ------------ ----------------- -----
K           CCCOMA_X64FRE_EN-US_DV9 Unknown        CD-ROM    Healthy      OK                  0 B

```
Copy ISO contents to a directory for modification. With the ISO mounted on `k:` drive, copy its contents to a directory e.g. `win11mod`

```powershell
robocopy k: win11mod -copy:dt -dcopy:t -e -r:0
```

Unmount the ISO. It won't be needed again.

```powershell
dismount-diskimage Win11_24H2_English_x64.iso
```

## 5. Step 2 - Extract `install.wim`

Copy Windows image `install.wim` to make it writable.

```powershell
cp win11mod/sources/install.wim .
```

## 6. Step 3 - Create answer file in Windows System Image Manager

Launch Windows System Image Manager from ADK. Open `install.wim`

![wsim_open_image.png](images/wsim_open_image.png)

Create an answer file. Add settings from the Windows Image panel to the answer file

![wsim_answer_file.png](images/wsim_answer_file.png)

## 7. Step 4 - Add answer file to ISO contents

Copy the answer file to the ISO directory `win11mod`

```powershell
cp autounattend.xml win11mod
```

## 8. Step 5 - Create ISO file from modified ISO contents

Use `oscdimg` from the ADK to create an ISO file from the ISO directory contents. Its usage is documented at https://learn.microsoft.com/en-us/troubleshoot/windows-server/setup-upgrade-and-drivers/create-iso-image-for-uefi-platforms and https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/oscdimg-command-line-options

```powershell
oscdimg -bootdata:2#p0,e,bwin11mod/boot/etfsboot.com#pEF,e,bwin11mod/efi/microsoft/boot/efisys.bin -u1 -udfver102 win11mod win11.auto.iso
```

## 9. Troubleshooting and debugging

Generally, it helps to make tiny incremental changes to easily isolate and fix mistakes. It also helps to first test new commands and files in small standalone toy examples in order to understand their usage and impact.

The System Image Manager application provides validation of an answer file with warnings and errors in its Messages pane

All powershell commands used here print messages on error

Errors during a run of the autoinstaller terminate the unattended install. The installer will stop at one of the interactive screens described below or display an error screen. The displayed screen may have enough information to figure out what is wrong in the answer file.

More detailed information can be obtained by launching a `cmd` terminal with `Shift-F10` when autoinstall is interrupted. Installer logs can be examined and other commands can be run in the terminal. The location of installer logs is documented at https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-log-files-and-event-logs

The preinstall environment available at the beginning of the installer is described at https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-intro

## 10. Windows installer screens

These screens are from the Windows 11 installer ISO at https://www.microsoft.com/en-us/software-download/windows11. The installer was run interactively to install Windows 11 Pro 24H2 version 10.0.26100.

## Screen 1 - Select language settings

Screen 1 selects the language for the installer. The only language allowed is the one selected when the installer ISO was downloaded.

Screen 1 allows a choice of locales for time and currency format.

![01_setup_language.png](images/01_setup_language.png)

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

## Screen 2 - Select keyboard settings

Screen 2 selects the keyboard.

![02_setup_keyboard.png](images/02_setup_keyboard.png)

![02_setup_keyboard_choice.png](images/02_setup_keyboard_choice.png)

Screen 2 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-International-Core-WinPE">
    <InputLocale>en-US</InputLocale>
  </component>
</settings>
```

## Screen 3 - Select setup option

Screen 3 selects whether to install or repair Windows

There's an option to switch to the old installer shown below. There's no way to switch back to the new installer.

![03_install_repair.png](images/03_install_repair.png)

Old installer

![03_old_installer.png](images/03_old_installer.png)

## Screen 4 - Product key

Screen 4 sets the product key. There's an option to proceed without a product key.

![04_product_key.png](images/04_product_key.png)

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

## Screen 5 - We're getting a few things ready

Screen 5 is an informational message.

![05_getting_ready_1_message.png](images/05_getting_ready_1_message.png)

## Screen 6 - Select image

Screen 6 selects the version of Windows to install.

![06_os_image.png](images/06_os_image.png)

Screen 6 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-Setup">
    <ImageInstall>
      <OSImage>
        <InstallFrom>
          <MetaData wcm:action="add">
            <Key>/IMAGE/NAME</Key>
            <Value>Windows 11 Pro</Value>
          </MetaData>
        </InstallFrom>
      </OSImage>
    </ImageInstall>
  </component>
</settings>
```

`<Key>` can be `/IMAGE/NAME`, `/IMAGE/INDEX` or `/IMAGE/DESCRIPTION` as documented at https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-setup-imageinstall-dataimage-installfrom-metadata-key

The corresponding `<Value>` is obtained from `install.wim` using `dism` from ADK. Its usage is documented at https://learn.microsoft.com/en-us/windows/deployment/windows-adk-scenarios-for-it-pros and https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/deployment-image-servicing-and-management--dism--command-line-options

```powershell
dism -get-imageinfo -imagefile:install.wim
```

```
Deployment Image Servicing and Management tool
Version: 10.0.26100.2454

Details for image : install.wim

Index : 1
Name : Windows 11 Home
Description : Windows 11 Home
Size : 18,727,965,088 bytes

Index : 2
Name : Windows 11 Home N
Description : Windows 11 Home N
Size : 18,190,503,625 bytes

Index : 3
Name : Windows 11 Home Single Language
Description : Windows 11 Home Single Language
Size : 18,725,453,549 bytes

Index : 4
Name : Windows 11 Education
Description : Windows 11 Education
Size : 19,230,378,207 bytes

Index : 5
Name : Windows 11 Education N
Description : Windows 11 Education N
Size : 18,698,289,981 bytes

Index : 6
Name : Windows 11 Pro
Description : Windows 11 Pro
Size : 19,250,929,144 bytes

Index : 7
Name : Windows 11 Pro N
Description : Windows 11 Pro N
Size : 18,700,496,532 bytes

Index : 8
Name : Windows 11 Pro Education
Description : Windows 11 Pro Education
Size : 19,230,428,845 bytes

Index : 9
Name : Windows 11 Pro Education N
Description : Windows 11 Pro Education N
Size : 18,698,315,750 bytes

Index : 10
Name : Windows 11 Pro for Workstations
Description : Windows 11 Pro for Workstations
Size : 19,230,479,483 bytes

Index : 11
Name : Windows 11 Pro N for Workstations
Description : Windows 11 Pro N for Workstations
Size : 18,698,341,519 bytes
```

## Screen 7 - Select setup option

Screen 7 shows the EULA to accept.

![07_license.png](images/07_license.png)

Screen 7 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-Setup">
    <UserData>
      <AcceptEula>true</AcceptEula>
    </UserData>
  </component>
</settings>
```

## Screen 8 - Please wait

Screen 8 is an informational message.

![08_wait_message.png](images/08_wait_message.png)

## Screen 9 - Searching for disks

Screen 9 is an informational message.

![09_disk_search_message.png](images/09_disk_search_message.png)

## Screen 10 - Select location to install Windows 11

Screen 10 selects the disk partition to install Windows. There are options to manage disks and partitions.

An empty disk

![10_disk_partition.png](images/10_disk_partition.png)

A disk with existing partitions created by a previous run of the installer

![10_partitioned_disk.png](images/10_partitioned_disk.png)

Screen 10 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="windowsPE">
  <component name="Microsoft-Windows-Setup">
    <DiskConfiguration>
      <Disk wcm:action="add">
        <WillWipeDisk>true</WillWipeDisk>
        <DiskID>0</DiskID>
        <CreatePartitions>
          <CreatePartition wcm:action="add">
            <Order>1</Order>
            <Size>200</Size>
            <Type>EFI</Type>
          </CreatePartition>
          <CreatePartition wcm:action="add">
            <Order>2</Order>
            <Size>16</Size>
            <Type>MSR</Type>
          </CreatePartition>
          <CreatePartition wcm:action="add">
            <Order>3</Order>
            <Size>129188</Size>
            <Type>Primary</Type>
          </CreatePartition>
          <CreatePartition wcm:action="add">
            <Order>4</Order>
            <Size>642</Size>
            <Type>Primary</Type>
          </CreatePartition>
        </CreatePartitions>
        <ModifyPartitions>
          <ModifyPartition wcm:action="add">
            <Order>1</Order>
            <PartitionID>1</PartitionID>
            <Format>FAT32</Format>
            <Label>System</Label>
          </ModifyPartition>
          <ModifyPartition wcm:action="add">
            <Order>2</Order>
            <PartitionID>3</PartitionID>
            <Letter>C</Letter>
            <Format>NTFS</Format>
            <Label>Windows</Label>
            <TypeID>ebd0a0a2-b9e5-4433-87c0-68b6b72699c7</TypeID>
          </ModifyPartition>
          <ModifyPartition wcm:action="add">
            <Order>3</Order>
            <PartitionID>4</PartitionID>
            <TypeID>de94bba4-06d1-4d40-a16a-bfd50179d6ac</TypeID>
            <Label>Recovery</Label>
            <Format>NTFS</Format>
          </ModifyPartition>
        </ModifyPartitions>
      </Disk>
    </DiskConfiguration>
  </component>
</settings>
```

The partitioning above follows Microsoft recommendations at https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions.

Here four partitions are created for a 127 GB disk.

Partition 1 is the system partition of type EFI. Its size is 200 MB and format is FAT32.

Partition 2 is the MSR. Its size is 16 MB.

Partition 3 is the Windows partition of type Primary. Its 126.2 GB size is determined by subtracting the sizes of Partitions 1, 2 and 4 from the disk size. Its format is NTFS.

Partition 4 is the Recovery partition. Its size of 642 MB is taken from the default partitioning done by an interactive run of the Windows installer. It's in keeping with Microsoft recommendations. It's bigger than the `winre.wim` Windows recovery tools image inside `install.wim` that it needs to accomodate but not overly wasteful. The size of `winre.wim` is obtained by using `dism` from the ADK to mount the Windows 11 Pro image inside `install.wim` and examining its contents.

```powershell
mkdir tmpmnt
dism -mount-wim -wimfile:install.wim -index:6 -mountdir:tmpmnt -readonly
ls tmpmnt/windows/system32/recovery/winre.wim
```

```
Mode             LastWriteTime      Length Name
----             -------------      ------ ----
-a----     9/5/2024  10:53 PM    532748021 winre.wim
```

```powershell
dism -unmount-wim -mountdir:tmpmnt -discard
```

The Recovery partition `<TypeID>` is `de94bba4-06d1-4d40-a16a-bfd50179d6ac` as documented at https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-setup-diskconfiguration-disk-modifypartitions-modifypartition-typeid

## Screen 11 - Ready to install

Screen 11 has a button to begin the install.

![11_ready_install.png](images/11_ready_install.png)

## Screen 12 - Installing Windows 11

Screen 12 is an informational message.

![12_installing_message.png](images/12_installing_message.png)

## Screen 13 - Installing progress

Screen 13 is an informational message.

![13_install_progress_message.png](images/13_install_progress_message.png)

## Screen 14 - Just a moment

Screen 14 is an informational message.

![14_moment_1_message.png](images/14_moment_1_message.png)

## Screen 15 - Windows logo

Screen 15 is an informational message.

![15_winlogo_1_message.png](images/15_winlogo_1_message.png)

## Screen 16 - Is this the right country or region?

Screen 16 selects the system locale.

Screen 16 marks the end of the WinPE setup stage of the installer and the beginning of the OOBE stage of the installer.

The option to create a local Windows account instead of using a Microsoft online account can be enabled at this point by pressing `Shift-F10` to launch a `cmd` terminal window.

![16_system_locale_1.png](images/16_system_locale_1.png)

## Screen 17 - Terminal window

Screen 17 shows a `cmd` terminal launched with `Shift-F10`

![17_cmd_terminal.png](images/17_cmd_terminal.png)

## Screen 18 - Run `oobe\bypassnro`

Screen 18 shows the `oobe\bypassnro` command being run. Running the command makes the installer relaunch the first OOBE screen to set the system locale

![18_bypassnro.png](images/18_bypassnro.png)

## Screen 19 - Just a moment

Screen 19 is an informational message.

![19_moment_2_message.png](images/19_moment_2_message.png)

## Screen 20 - Windows logo

Screen 20 is an informational message.

![20_winlogo_2_message.png](images/20_winlogo_2_message.png)

## Screen 21 - Is this the right country or region?

Screen 21 selects the system locale.

![21_system_locale_2.png](images/21_system_locale_2.png)

Screen 21 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-International-Core">
    <UILanguage>en-US</UILanguage>
    <SystemLocale>en-US</SystemLocale>
    <UserLocale>en-US</UserLocale>
    <UILanguageFallback>en-US</UILanguageFallback>
  </component>
</settings>
```

## Screen 22 - Is this the right keyboard layout or input method?

Screen 22 selects the keyboard.

![22_system_keyboard.png](images/22_system_keyboard.png)

Screen 22 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-International-Core">
    <InputLocale>en-US</InputLocale>
  </component>
</settings>
```

## Screen 23 - Want to add a second keyboard layout?

Screen 23 selects a second keyboard.

![23_second_keyboard.png](images/23_second_keyboard.png)

Screen 23 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-International-Core">
    <InputLocale>en-US;fr-FR;es-ES</InputLocale>
  </component>
</settings>
```

## Screen 24 - Checking for updates

Screen 24 is an informational message.

![24_check_update_1_message.png](images/24_check_update_1_message.png)

## Screen 25 - Good things are coming your way

Screen 25 is an informational message.

![25_good_things_message.png](images/25_good_things_message.png)

## Screen 26 - your PC will restart before you continue

Screen 26 is an informational message.

![26_restart_1_message.png](images/26_restart_1_message.png)

## Screen 27 - Welcome

Screen 27 is an informational message.

![27_welcome_message.png](images/27_welcome_message.png)

## Screen 28 - Let's name your device

Screen 28 sets the system hostname.

![28_hostname.png](images/28_hostname.png)

Screen 28 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="specialize">
  <component name="Microsoft-Windows-Shell-Setup">
    <ComputerName>winauto1</ComputerName>
  </component>
</settings>
```

## Screen 29 - Just a moment

Screen 29 is an informational message.

![29_moment_3_message.png](images/29_moment_3_message.png)

## Screen 30 - How would you like to set up this device?

Screen 30 selects whether the system is personal or for work.

![30_personal_use.png](images/30_personal_use.png)

## Screen 31 - Checking for Windows updates

Screen 31 is an informational message.

![31_check_update_2_message.png](images/31_check_update_2_message.png)

## Screen 32 - Windows update - Step 1 of 3: Downloading

Screen 32 is an informational message.

![32_update_step_1_message.png](images/32_update_step_1_message.png)

## Screen 33 - Windows update - Step 2 of 3: Installing

Screen 33 is an informational message.

![33_update_step_2_message.png](images/33_update_step_2_message.png)

## Screen 34 - Windows update - Step 3 of 3: Preparing to restart

Screen 34 is an informational message.

![34_update_step_3_message.png](images/34_update_step_3_message.png)

## Screen 35 - Restarting

Screen 35 is an informational message.

![35_restart_2_message.png](images/35_restart_2_message.png)

## Screen 36 - Updates are underway

Screen 36 is an informational message.

![36_update_underway_message.png](images/36_update_underway_message.png)

## Screen 37 - Updates progress

Screen 37 is an informational message.

![37_update_progress_message.png](images/37_update_progress_message.png)

## Screen 38 - Welcome

Screen 38 is an informational message.

![38_welcome_message.png](images/38_welcome_message.png)

## Screen 39 - Unlock your Microsoft experience

Screen 39 has a Sign in button.

![39_microsoft_signin.png](images/39_microsoft_signin.png)

## Screen 40 - Who's going to use this device?

Screen 40 sets a local account username. This screen is enabled by running `oobe\bypassnro` in Screen 18.

![40_username.png](images/40_username.png)

Screen 40 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-Shell-Setup">
    <UserAccounts>
      <LocalAccounts>
        <LocalAccount wcm:action="add">
          <Name>demo1</Name>
          <Group>Administrators</Group>
          <DisplayName>demo1</DisplayName>
        </LocalAccount>
      </LocalAccounts>
    </UserAccounts>
  </component>
</settings>
```

## Screen 41 - Create a super memorable password

Screen 41 sets the user password.

![41_password.png](images/41_password.png)

Screen 41 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-Shell-Setup">
    <UserAccounts>
      <LocalAccounts>
        <LocalAccount wcm:action="add">
          <Password>
            <Value>ZABlAG0AbwAxAFAAYQBzAHMAdwBvAHIAZAA=</Value>
            <PlainText>false</PlainText>
          </Password>
        </LocalAccount>
      </LocalAccounts>
    </UserAccounts>
  </component>
</settings>
```

## Screen 42 - Confirm your password

Screen 42 asks to reenter the password.

![42_confirm_password.png](images/42_confirm_password.png)

## Screen 43 - Now add security questions

Screen 43 selects the first of 3 security questions

![43_security_question.png](images/43_security_question.png)

![43_security_question_dropdown.png](images/43_security_question_dropdown.png)

## Screen 44 - Choose privacy settings for your device

Screen 44 selects privacy settings.

![44_privacy_1.png](images/44_privacy_1.png)

![44_privacy_2.png](images/44_privacy_2.png)

![44_privacy_3.png](images/44_privacy_3.png)

Screen 44 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-Shell-Setup">
    <OOBE>
      <ProtectYourPC>1</ProtectYourPC>
    </OOBE>
  </component>
</settings>
```

## Screen 45 - Just a moment

Screen 45 is an informational message.

![45_moment_4_message.png](images/45_moment_4_message.png)

## Screen 46 - Hi

Screen 46 is shown when the environment is set up for the new local user account.

Screen 46 settings are set by the following section of `autounattend.xml`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-Shell-Setup">
    <AutoLogon>
      <Password>
        <Value>ZABlAG0AbwAxAFAAYQBzAHMAdwBvAHIAZAA=</Value>
        <PlainText>false</PlainText>
      </Password>
      <LogonCount>1</LogonCount>
      <Username>demo1</Username>
      <Enabled>true</Enabled>
    </AutoLogon>
    <FirstLogonCommands>
      <SynchronousCommand wcm:action="add">
        <CommandLine>reg add \&quot;hklm\software\microsoft\windows nt\currentversion\winlogon\&quot; /v autologoncount /t reg_dword /d 0 /f</CommandLine>
        <Order>1</Order>
      </SynchronousCommand>
    </FirstLogonCommands>
  </component>
</settings>
```

The settings above set auto login to occur once. The `<FirstLogonCommands>` section is needed for `<LogonCount>` of 1 to work as described at https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-autologon-logoncount

Any custom commands may be run on first login. Here's an example of simple sanity checks. It records the time and username of the first login. It also prints the value of `AutoLogonCount` in the registry that is used to track the number of auto logins specified in `<AutoLogon>`.

```xml
<settings pass="oobeSystem">
  <component name="Microsoft-Windows-Shell-Setup">
    <FirstLogonCommands>
      <SynchronousCommand wcm:action="add">
        <CommandLine>powershell &quot;&amp;{ [datetime]::now.tostring(\&quot;yyyy-MM-dd_HH:mm:ss.fffK: \&quot;) + [string](whoami); reg query \&quot;hklm\software\microsoft\windows nt\currentversion\winlogon\&quot;; reg add \&quot;hklm\software\microsoft\windows nt\currentversion\winlogon\&quot; /v autologoncount /t reg_dword /d 0 /f} &gt; autologincmds.txt&quot;</CommandLine>
        <Order>1</Order>
      </SynchronousCommand>
    </FirstLogonCommands>
  </component>
</settings>
```

The Powershell commands without XML escapes are

```powershell
powershell "&{
  [datetime]::now.tostring(\"yyyy-MM-dd_HH:mm:ss.fffK: \") + [string](whoami)
  reg query \"hklm\software\microsoft\windows nt\currentversion\winlogon\"
  reg add \"hklm\software\microsoft\windows nt\currentversion\winlogon\" /v autologoncount /t reg_dword /d 0 /f
} > autologincmds.txt"
```

![46_hi_message.png](images/46_hi_message.png)

## Screen 47 - Getting things ready for you

Screen 47 is an informational message.

![47_getting_ready_2_message.png](images/47_getting_ready_2_message.png)

## Screen 48 - This might take a few minutes

Screen 48 is an informational message.

![48_few_minutes_message.png](images/48_few_minutes_message.png)

## Screen 49 - Login

Screen 49 is the Windows login screen that shows the system is ready for use after the end of the installation.

![49_login.png](images/49_login.png)

