---
Order: 20
xref: v2-qde
Title: Setup
Description: How to setup QDE
RedirectFrom: docs/quick-deployment-setup
---

> :choco-info: **NOTE**
>
> This document is for **Version 2** of the Quick Deployment Environment.
> If you're using an older version of QDE, please refer to the [QDEv1 Setup page](xref:v1-qde).

This document contains instructions for importing the QuickDeploy appliance/VM, or creating a VM and attaching the QuickDeploy disk image to it.
You will receive a download link via email for an archive of the VM image.
Once you have this downloaded, it will be ready for extraction and import into your environment.

> :choco-warning: **WARNING**
>
> Please follow these steps in **exact** order.
> These will be very important later when you are trying to use the environment.

## Step 0: Setup Considerations

Keep the following points in mind during initial setup:

* You will need access to AWS to download the VM image (specifically `s3.amazonaws.com`).
* Hostname/FQDN changes made after setup will invalidate all scripts and certificates.
  Thus, if you plan to change the hostname, you must do so prior to running any setup scripts.
  If you run into issues, refer to the README file on the desktop of the VM.
* Self-signed certificates are generated by default.
  If you plan to use your own certificates, pay special attention to the setup instructions in the provided [README](xref:v2-desktop-readme).
* The back-end database is configured with no outbound connections.
  If you plan to change this, please reach out to Support for assistance.

### Step 0.1: Changing the Hostname

> :choco-warning: **WARNING**
>
> Renaming the QDE host needs to be completed prior to ANYTHING that is done on the QDE box.
> It is recommended to use the default hostname unless you really need to change it.
>
> Please contact support if you run into issues.

If you rename the QDE Environment after running the initial setup scripts, there are some additional changes you'll need to make:

1. Update client setup scripts that are initialized to the current QDE hostname during setup.
   You will need to go through the existing scripts in `C:/choco-setup/files` and replace the default `-Hostname` value of ALL scripts that have it.
   Please contact Support if you're unsure how to proceed here.
1. Upload the `ClientSetup.ps1` and `ChocolateyInstall.ps1` to the Nexus repository.
1. Replace the IIS-hosted `Import-ChocoServerCertificate.ps1` script using the `New-IISCertificateHost.ps1` script.
1. Deploy the new certificate to all clients.
   See [the Client Setup](xref:v2-client-setup) documentation for instructions on how to do this.
1. There may be additional places impacted
   Check with support if you need confirmation.

## Step 1: Import the Virtual Environment

Choose one of the following methods for what your hypervisor environment supports.

### Platform: Azure

> :choco-warning: **WARNING**
>
> We now recommend using the [Chocolatey for Business Azure Environment](xref:c4b-azure).
> Please refer to the link above for steps to quickly deploy a Chocolatey for Business environment into Azure.

### Platform: Hyper-V

> :choco-info: **NOTE**
>
> Windows 10 and Windows Server 2016/2019 version of Hyper-V now come with built-in support for Hyper-V Integration Services, so they automatically get pushed to guest VMs.
> In older versions of Hyper-V, you should see an option to `Insert Integration Services Setup Disk`.
> You can use this option to install and enable Hyper-V Integration Services.

#### Hyper-V Appliance

1. Download zip archive containing the Hyper-V VM directory structure, and unzip it to the directory you wish to store it in.
1. Open Hyper-V Manager, and select `Import Virtual Machine` from the right sidebar (or  **Action** menu), and choose the folder you extracted from the above archive (e.g. ChocoServer).
1. Increase the size of the VHD, for example, to 500GB.
   Increase to what you feel comfortable with, keeping in mind QDE is in part a repository server, and will require a fair amount of space to operate for any length of time if you're utilizing that.

    ```powershell
    # This only increases the available size of the image.
    # You will still need to increase the allocated space for the C drive when you start the VM.
    Resize-VHD -Path C:\path\to\QuickDeploy Environment.vhd -Size 500GB
    ```

#### See it in Action

<p>
<div class="ratio ratio-16x9">
    <iframe src="https://www.youtube.com/embed/A2pm_e75qfo?list=PL84yg23i9GBirbiuF9tAn3aZxsAEw5x1W" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen>
    </iframe>
</div>
<br>
</p>

#### Hyper-V VHD

1. Download the VHD from provided link, and unzip it to the directory you wish to store it in.
1. Increase the size of the VHD, for example, to 500GB.
   Increase to what you feel comfortable with, keeping in mind QDE is in part a repository server, and will require a fair amount of space to operate for any length of time if you're utilizing that.

    ```powershell
    # This only increases the available size of the image.
    # You will still need to increase the allocated space for the C drive when you start the VM.
    Resize-VHD -Path C:\path\to\QuickDeploy Environment.vhd -Size 500GB
    ```

1. Open Hyper-V Manager.
1. Create a new VM.
1. If prompted, choose a `Generation 1` virtual machine.
1. Set startup memory to `8192 MB` (you can change this later if you need to).
1. When asked to create a new disk or use an existing one, select `Use existing virtual disk` and browse to the VHD you unzipped in Step 1.
1. Adjust the hardware specifications of the VM.
   For a performant system, the following are recommended:
    * 4 vCPUs
    * 8 GB RAM

#### See it in Action

<p>
<div class="ratio ratio-16x9">
    <iframe src="https://www.youtube.com/embed/A2pm_e75qfo?list=PL84yg23i9GBirbiuF9tAn3aZxsAEw5x1W" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen>
    </iframe>
</div>
<br>
</p>

### Platform: VMware / Other (OVF Template)

1. Download the OVF from the provided link, and unzip to the directory you'd like to store it.
1. Import the template into your hypervisor of choice.
   You can typically directly import the OVF template file, and most settings should be pre-configured for you.
    * Instructions on doing this for VMware are [here](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.html.hostclient.doc/GUID-FBEED81C-F9D9-4193-BDCC-CC4A60C20A4E_copy.html).
1. Once the import completes, go into the VM's hardware settings and expand the main disk to at least 500GB.
1. Boot up the VM, and (if needed) install any software required / recommended for your hypervisor.

> :choco-info: **NOTE**
>
> Upon first booting the environment in ESXi with the HTML modal console control, the mouse will not be available to the VM.
> We recommend you use either RDP or VMRC (VMWare Remote Console) to access the VM initially and install VMWare Tools.
> Once VMWare Tools is installed, the mouse and other advanced VM functionality will be made available to the VM.

### Platform: VMware (VMDK file)

1. Download VMDK from provided link, and unzip it to the directory you wish to store it.
1. For ESX/ESXi, open vSphere and upload the downloaded VMDK to your datastore.
1. Create a new VM.
1. When prompted for OS type, choose `Windows Server 2019` (if available), or `Windows Server 2016 or later`.
1. Adjust the hardware specifications of the VM. For a performant system, the following are recommended:

* 4 vCPUs
* 8 GB RAM

1. Use the "x" icon at the far right of `Hard disk 1` to remove it.
1. In the `VM Options` tab, and under the `Boot Options` dropdown, set your `Firmware` to `BIOS` (**not** UEFI). Click `Next`, then `Finish`.
1. Now go back into the VM configuration using the `Edit Settings` context menu.
1. Click the `Add hard disk` button, select `Existing hard disk`, and browse to the VMDK you uploaded. Once slected, expand the `New hard disk` dropdown, and set the `Controller location` to `IDE controller 0`, or whichever IDE controllers you have available. Click `Save`.
    > :choco-danger: **IMPORTANT**
    >
    > [vCenter/ESX/ESXi] You **must** select an `IDE controller` under the "Controller Location" setting of the disk. If you leave the controller as SCSI (default), your VM will not boot.
1. Once again, go back into the `Edit settings` context menu for the VM, and expand the disk you attached to 500GB (double-check in OS, and extend if needed).
    > :choco-info: **NOTE**
    >
    > Likely you will need to allocate the additional space to the C drive.
1. Boot up VM, and Install VMware Tools using the console menus (this will require a reboot).

#### See it in Action

<p>
<div class="ratio ratio-16x9">
    <iframe src="https://www.youtube.com/embed/UbEWgHNXPW8?list=PL84yg23i9GBirbiuF9tAn3aZxsAEw5x1W" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen>
    </iframe>
</div>
<br>
</p>

<p>
<div class="ratio ratio-16x9">
    <iframe src="https://www.youtube.com/embed/H--woorTI2U?list=PL84yg23i9GBirbiuF9tAn3aZxsAEw5x1W" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen>
    </iframe>
</div>
<br>
</p>

### Platform: Other

If your hypervisor supports importing from an OVA or OVF, you should be able to import that image in most cases.
Otherwise, or if you run into issues doing that, we recommend downloading the VMDK file and converting it to a format supported by your hypervisor.

Please ensure you know what you're doing before attempting this.
We can only support the hypervisors listed here, and will not be able to provide assistance in conversion and ensuring the QDE image works in your environment.
However, once you have the image running in your hypervisor, our support team will be happy to assist you however they can.

## Step 2: Other Considerations

### Step 2.1: DNS Settings

The QDE environment is configured by default to use DHCP for easier initial setup.
You will likely need to reconfigure it with a static IP address depending on your organization's policies.
Also check the additional [high level requirements for CCM](xref:ccm-setup#high-level-requirements) if making changes to DNS settings.

## Step 3: Virtual Environment Setup

> :choco-warning: **WARNING**
>
> If you have an existing corporate environment you will be servicing with the QDE VM, be sure to perform your organization-specific initial configuration **_before_** running setup scripts.

### Step 3.1: Expand Disk Size

On the machine, please check the size of the C drive.
If the volume needs to be expanded, expand it to the space you've allocated for the machine by running this command in an administrative PowerShell console:

```powershell
# This should increase the space available on the C drive to the maximum available size.
Resize-Partition -DriveLetter C -Size ((Get-PartitionSupportedSize -DriveLetter C).SizeMax)
```

Alternatively, you can use the Disk Management utility to expand the disk, if you prefer.

### Step 3.2: Add Chocolatey License File

In the [Quick Deployment Desktop Readme](xref:v2-desktop-readme), you will be instructed to copy your `chocolatey.license.xml` to the VM.
You should already have been given a license file prior to downloading QDE.
We generally recommend you copy the file into the VM directly, if possible.

> :choco-warning: **WARNING**
>
> If you find that you need to copy the file contents in order to get the license file text into a new file in QDE, the file format and name is extremely important to get right.
> If you don't save the file in UTF-8 encoding or there is extra whitespace, Chocolatey will consider it invalid.
>
> Please contact support if you need help here.

### Step 3.3: Follow README instructions to finish environment setup

On the desktop of your QDE VM, there is a `Readme.html` file, that will guide you through the rest of the setup process once you are logged in.
A version of this readme file can be found in the [Quick Deployment Desktop Readme](xref:v2-desktop-readme).

> :choco-info: **NOTE**
>
> The online version is likely more up to date than the ReadMe you will find on the desktop (not including redacted items like credentials).
> If there are conflicts between the desktop readme and what you see online, prefer the online version.
>
> When running the `Set-QDEnvironment` script ensure you are logged into the QDE VM with the _local admininistrator_ account.
> This is necessary for SQL to work properly initially.

### Step 3.4: Database Password Changes (Optional)

The database credentials are currently pre-set.
If you would like to change the credentials associated with the database, you will need to follow these steps.

1. Change the database access credentials
1. Reinstall the `chocolatey-management-service` package

    ```powershell
    choco uninstall chocolatey-management-service -y
    choco install chocolatey-management-service -y --package-parameters-sensitive="'/ConnectionString=""Server=localhost\SQLEXPRESS;Database=ChocolateyManagement;User ID=ChocoUser;Password=NewPassword;""'"
    ```

1. Reinstall the chocolatey-management-web package

    ```powershell
    choco uninstall chocolatey-management-web -y
    Choco install chocolatey-management-web -y --package-parameters-sensitive="'/ConnectionString=""Server=Localhost\SQLEXPRESS;Database=ChocolateyManagement;User ID=ChocoUser;Password=NewPassword;""'"
    ```

## Step 4: Firewall Changes

See [QDE Firewall Changes](xref:v2-firewall-changes).

## Step 5: Install and Configure Chocolatey on Clients

See [QDE Client Setup](xref:v2-client-setup).

## FAQ

### How do I upgrade QDE?

While we will continue to make improvements to the QDE, there is no upgrade path for the Virtual Machine itself.
You can choose to start over with a newer version, but that would mean going through the entire setup process from the beginning once again.

Instead, it is much easier to upgrade the components individually, and that is how we recommend upgrading aspects of QDE.
Should you want to upgrade say Central Management, you can follow the Central Management steps for upgrade at [Upgrade Central Management](xref:ccm-upgrade).

### Should I upgrade Jenkins?

The Jenkins package in QDE is pinned at version **2.222.4**; this is the latest version we have currently tested.
An upgrade is possible, however we recommend you approach this with caution for the time being.
Upgrading to a newer version via the typical Jenkins installer package requires some additional work due to the way Jenkins handles upgrades with their installer.

<details>
    <summary><strong>Upgrading through the Web UI</strong></summary>

Upgrading through the Web UI is the simpler way to go, and doesn't seem to carry any of the complications that can arise when upgrading via their MSI installer.
When an upgrade is available, Jenkins will show a new notification that looks like this when clicked:

![Jenkins upgrade notification, showing the notification message prompting to upgrade](/assets/images/jenkins-upgrade-notification.png)

To upgrade via the web UI:

1. Login to Jenkins
1. Click the :bell: notifications bell
1. Select **Or Upgrade Automatically** &mdash; _not_ the **download** link
1. When you are taken to the **Installing Plugins/Upgrades** screen, check the **Restart Jenkins when installation is complete and no jobs are running** checkbox
1. Once Jenkins indicates the process is complete and has restarted, log back in

</details>

If you would like to upgrade Jenkins to a newer version using the installer, please refer to [Jenkins' Upgrade Guide](https://www.jenkins.io/doc/upgrade-guide/2.235/#upgrading-to-jenkins-lts-2-235-5).
Some or all of the complexities in the upgrade process will likely be made automatic in the Chocolatey package as we're able to test and verify that the upgrade path works.
Until then, approach with caution and follow the Jenkins documentation closely when attempting an upgrade.

### What are the requirements for installing Chocolatey on a TLS1.2 hardened endpoint using ClientSetup.ps1 script?

The following TLS Ciphers will need to be applied:

* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

In addition, the following registry keys will need to be applied:

**[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v2.0.50727]**

* _SystemDefaultTlsVersions_ = `dword:00000001`
* _SchUseStrongCrypto_ = `dword:00000001`

**[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]**

* _SystemDefaultTlsVersions_ = `dword:00000001`
* _SchUseStrongCrypto_ = `dword:00000001`

The following PowerShell code will set these items appropriately:

```powershell
# set strong cryptography on 64 bit .Net Framework (version 4 and above)
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NetFramework\v2.0.50727' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord

# set strong cryptography on 32 bit .Net Framework (version 4 and above)
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NetFramework\v2.0.50727' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord
```

[Quick Deployment Environment](xref:qde)