# Ansible Playbooks to upgrade Cisco Devices

## Introduction
These playbooks are used to upgrade Cisco IOS/IOS-XE Devices.

For Catalyst using INSTALL mode (e.g. Catalyst 9200, Catalyst 9300, Catalyst 3K using INSTALL mode), use playbooks under the **Catalyst_Install_Mode** directory.

For Routers running IOS-XE and already running in INSTALL mode (i.e. boot system is pointing to bootflash:/packages.conf), use playbooks under the **IOS_XE_Router_Install_Mode** directory.

Finall, for legacy IOS devices that use the "boot system flash" command to point to the firmware file on the flash, use playbooks under the **Legacy_Bundle_Mode** directory.


## File Structure
```
.
├── Catalyst_Install_Mode
│   ├── activate-image.yaml
│   ├── cleanup-flash.yaml
│   ├── file-validity-check.yaml
│   ├── tftp-upload-image.yaml
│   └── upload-image-scp.yaml
├── IOS_XE_Router_Install_Mode
│   ├── activate-image.yaml
│   ├── cleanup-flash.yaml
│   ├── file-validity-check.yaml
│   ├── tftp-upload-image.yaml
│   └── upload-image-scp.yaml
├── Legacy_Bundle_Mode
│   ├── activate-image.yaml
│   ├── file-validity-check.yaml
│   ├── tftp-upload-image.yaml
│   └── upload-image-scp.yaml
├── README.md
├── ansible.cfg
├── ansible_hosts
├── group_vars
│   ├── catalyst_vars.yaml
│   ├── generic_ios_vars.yaml
│   └── ios_xe_router_vars.yaml
├── reboot.yaml
└── show-version.yaml
```


## Usage

Group variables are stored in the **group_vars/** sub-directory. Use ```ansible-vault``` to encrypt when necessary.

All IOS/IOS-XE images should be placed under the **images/** sub-directory. 

Playbooks breaks down into 4 phases with 4 individual playbooks:
1. **cleanup-flash.yaml**       - Clean up any unused firmware files to make space on the switch flash.  
2. **upload-image-scp.yaml**    - Upload image file, if not already exist on the switch. This playbook uses SCP to copy file.
3. **tftp-upload-image.yaml**   - In some cases, when SCP is not available on the devices, use TFTP to transfer image file.
4. **file-validity-check.yaml** - Check the md5 checksum against calculated value on the host to verify validity of the file post upload.
5. **activate-image.yaml**      - Perform the upgrade using the install command.

These playbooks should be executed in the sequence listed above.


## Preparing the playbooks for execution

1. Modifying **catalyst_vars.yaml** , **ios_xe_router_vars.yaml** or **generic_ios_vars.yaml** depending on use case, under **group_var**s sub-directory:

*Before executing the playbooks, make sure to change the variables in **[catalyst|ios_xe_router|generic]_vars.yaml** to reflect targeted upgrade version, image file location, bootflash location etc. An ansible timeout value is also included in these vars files. In the case of uploading firmware to devices, it is recommended to tune up this timeout value to allow Ansible to finish the upload task without timeouting the session. For sites using 4G, it is particularly important to account for the time for these upload tasks.*

It is also recommended to test ssh connection from Ansible host to the devices before executing these playbooks.
Alternatively, run the ```show-version.yaml``` playbook which displays the running version of devices as well as proving SSH connection from Ansible host is successful.

Below list explains each variable:

- **username**                : username used to login from Ansible host to the device
- **password**                : password used to login from Ansible host to the device. Recommended to use ansible-vault to encrypt this file.
- **ansible_network_os**      : instruct Ansible to detect network device platform. Use "ios" for Cisco IOS or IOS-XE devices.
- **ansible_command_timeout** : Ansible default command timeout value. In a typical situation, 30 seconds is enough. However, larger value is required for longer operation like uploading file from the host. 
- **upgrade_ios_version**     : This is the version to be upgraded. Note that this should match the returned value from within the Ansible gather facts API. An Example is "16.12.11".
- **upgrade_file**            : This is the firmware name to be used to upgrade. Directly replace this field with the image name downloaded from Cisco. Example: "cat3k_caa-universalk9.16.06.09.SPA.bin"
- **upgrade_path**            : This is the path for the Ansible host to locate image file locally. Pay attention to the playbook running directory path. Consider using an absolute path. Example: "images/cat3k_caa-universalk9.16.06.09.SPA.bin"
- **switch_file**             : Specifies the location and the filename of the image to be uploaded on the device. Some devices may use bootflash instead of flash. Example: "flash:/cat3k_caa-universalk9.16.06.09.SPA.bin"
- **router_file**             : Specifies the location and the filename of the image to be uploaded on the device. Some devices may use bootflash instead of flash. Example: "flash:/cat3k_caa-universalk9.16.06.09.SPA.bin"
- **ftp_file**                : In some tests, SCP performs worse than FTP in uploading file. In case of using FTP, this variable specifies the image file location of the FTP server. Example: "~/ansible/images/cat3k_caa-universalk9.16.06.09.SPA.bin"
- **ftp_ip**                  : Specifies the FTP server ip address when FTP is used.
- **ftp_username**            : FTP user used to copy file.
- **ftp_password**            : FTP password used. 


2. Modify **ansible_hosts** to fit in devices. 

3. Modify playbooks to include required hosts to be executed against:

Update the ```hosts: all``` to reflect individual device names, or section of devices defined in **ansible_hosts**.
```
- name: Display running version on Cisco IOS XE switches 
  hosts: all
  connection: network_cli
  gather_facts: yes
  vars_files:
    - ../group_vars/catalyst_vars.yaml 
```

4. Test playbooks by using ```show-version.yaml```. Commands as below: (Note that ansible-vault will prompt for vault password if any of the variable files or playbooks are ansible-vault encrypted.)

```
   ansible-playbook -i ansible_hosts --ask-vault-pass show-version.yaml
```
   (omit --ask-vault-pass if ansible-vault is not in used)  OR to run the real playbooks:
```
   ansible-playbook -i ansible_hosts Catalyst_Install_Mode/cleanup-flash.yaml
```


5. Download required firmware file and upload to the Ansible host (assuming SCP will be used to transfer file from host to devices), 
and put it under the **images/** sub-directory. It is assumed that before uploading the firmware to devices, the image should be verified with MD5 checksum against the published value on Cisco site.

6. Deploy the upgrade by running the sequence of playbooks listed above.


## Contributor

The playbooks are developed by Gary Wong (ajidom@gmail.com).
