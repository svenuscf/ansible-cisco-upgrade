# Ansible Playbooks to upgrade Cisco Devices

## Introduction
These playbooks are used to upgrade Cisco IOS/IOS-XE Devices.

- For Catalyst using INSTALL mode (e.g. Catalyst 9200, Catalyst 9300, Catalyst 3K using INSTALL mode), use playbooks under the **Catalyst_Install_Mode** directory.

- For Routers running IOS-XE and already running in INSTALL mode (i.e. boot system is pointing to ```bootflash:/packages.conf```), use playbooks under the **IOS_XE_Router_Install_Mode** directory.

- For Legacy IOS devices that use the "boot system flash" command to point to the firmware file on the flash, use playbooks under the **Legacy_Bundle_Mode** directory.


## File Structure
```
.
├── Catalyst_Install_Mode
│   ├── activate-image.yaml
│   ├── cleanup-flash.yaml
│   ├── file-validity-check.yaml
│   ├── pre-upgrade-check.yaml
│   ├── tftp-upload-image.yaml
│   └── upload-image-scp.yaml
├── IOS_XE_Router_Install_Mode
│   ├── activate-image.yaml
│   ├── cleanup-flash.yaml
│   ├── file-validity-check.yaml
│   ├── pre-upgrade-check.yaml
│   ├── tftp-upload-image.yaml
│   └── upload-image-scp.yaml
├── Legacy_Bundle_Mode
│   ├── activate-image.yaml
│   ├── file-validity-check.yaml
│   ├── pre-upgrade-check.yaml
│   ├── tftp-upload-image.yaml
│   └── upload-image-scp.yaml
├── README.md
├── ansible.cfg
├── ansible_hosts
├── group_vars
│   ├── catalyst_vars.yaml
│   ├── generic_ios_vars.yaml
│   └── ios_xe_router_vars.yaml
├── log
├── reboot.yaml
└── show-version.yaml
```


## Usage

Group variables are stored in the **group_vars/** sub-directory. Use ```ansible-vault``` to encrypt these variable files in production as there are senstive credentials.

All IOS/IOS-XE images should be placed under the **images/** sub-directory. 

Playbooks breaks down into 4 phases with 4 individual playbooks:
1. **cleanup-flash.yaml**       - Clean up any unused firmware files to free up space on the network devices.  
2. **upload-image-scp.yaml**    - Upload the image file using SCP. This playbook assumes the image doesn't already exist on the device.
3. **tftp-upload-image.yaml**   - Alternatively, use TFTP to upload the image file to the device. TFTP is found to be faster than SCP in some testings. 
4. **file-validity-check.yaml** - Validate the uploaded image by comparing its MD5 checksum against the host's calculated value.
5. **pre-upgrade-check.yaml**   - Perform a pre-upgrade check and write the results to a logfile under **log/** directory.
6. **activate-image.yaml**      - Perform the upgrade using the install command.

The playbooks should be executed in this sequence.


## Preparing the playbooks for execution

### 1. Modify Group Variables
The group variables for different types of devices are stored in **catalyst_vars.yaml**, **ios_xe_router_vars.yaml** and **generic_ios_vars.yaml**, under the **group_var**s sub-directory:

Make sure to update these variables based on your target upgrade version, image file location, bootflash location, etc. Pay particular attention to the ```ansible_command_timeout``` value for longer tasks like file uploads, especially on slower networks like 4G.

Key variables include:
- **username**                : Login username for the device.
- **password**                : Login password. Use ```ansible-vault``` to encrypt this for security.
- **ansible_network_os**      : Set to ```"ios"``` for Cisco IOS or IOS-XE devices.
- **ansible_command_timeout** : Timeout for long-running operations (e.g., image uploads).
- **upgrade_ios_version**     : Target IOS version (e.g., ```"16.12.11"```).
- **upgrade_file**            : Firmware file name (e.g., ```"cat3k_caa-universalk9.16.06.09.SPA.bin"```).
- **upgrade_path**            : Path to the image on the Ansible host (e.g., ```"images/cat3k_caa-universalk9.16.06.09.SPA.bin"```).
- **switch_file**             : Target location and name on the device for Catalyst switches (e.g., ```"flash:/cat3k_caa-universalk9.16.06.09.SPA.bin"```).
- **router_file**             : Target location and name on the device for routers (e.g., ```"bootflash:/cat3k_caa-universalk9.16.06.09.SPA.bin"```).
- **tftp_server_ip**          : IP address of the TFTP server.


### 2. Modify the Inventory (```ansible_hosts```) 

Ensure your ```ansible_hosts``` file reflects the devices to be upgraded.

### 3. Update Playbooks with Correct Hosts

In each playbook, replace ```hosts: all``` with the appropriate device group or individual host from your ```ansible_hosts``` file.

Example:
```
- name: Display running version on Cisco IOS XE switches 
  hosts: all
  connection: network_cli
  gather_facts: yes
  vars_files:
    - ../group_vars/catalyst_vars.yaml 
```

### 4. Test **SSH** Connectivity

Before running any upgrade playbooks, it’s recommended to test the SSH connection from the Ansible host to the target devices by running the **show-version.yaml** playbook.
```
   ansible-playbook -i ansible_hosts --ask-vault-pass show-version.yaml
```
(Use --ask-vault-pass if any of the files are encrypted with ansible-vault.)

### 5. Upload the Firmware File

Download the required firmware image and place it in the **images/** directory on your Ansible host. Verify the MD5 checksum of the image before uploading it to the devices.

### 6. Execute the Upgrade Playbooks

Run the playbooks in sequence:
```
ansible-playbook -i ansible_hosts Catalyst_Install_Mode/cleanup-flash.yaml
```

Once the flash is cleared, proceed with the image upload, verification, and activation.

Alternatively, before proceeding to activate the image, perform a pre-upgrade check by running the ```pre-upgrade-check.yaml``` playbook. The results will be stored in a log file under the **log/** directory.

To view a summary of device upgrade readiness, use the following command:
```
grep Report log/*
```

### 7. Logging Playbook Outputs

Ansible has built-in logging capabilities. It is recommended to direct playbook outputs to a log file for later tracing of playbook execution results.

A specific playbook, **pre-upgrade-check.yaml**, has been developed to perform a check against devices and output results to a readable log file. This makes it easier for administrators to see which devices are ready for upgrade and which are not.

For example, to capture the output of the ```show-version.yaml``` playbook into a timestamped log file, use the following command:

```
ansible-playbook -i ansible_hosts show-version.yaml | tee log/show-version-$(date +%Y-%m-%d_%H-%M-%S).log
```


## Contributor

The playbooks are developed by Gary Wong (ajidom@gmail.com).
