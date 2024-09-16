# Ansible Playbooks to upgrade Cisco IOS XE Switches

These playbooks are used to upgrade Cisco IOS XE Switches.

Assume switches are running in INSTALL mode and will also be upgraded using INSTALL mode.

If BUNDLE mode is involed, further development on new playbook is required. 

all.yaml should be put into group_vars/ sub-directory. If needed, use ansible-vault to encrypt.

All IOS XE images should be placed at the images/ sub-directory. 

Playbooks breaks down into 4 phases:
1. Clean up any unused firmware files to make space on the switche flash.
2. Upload image file, if not already exist on the switch.
3. Check the md5 checksum against calculated value on the host to verify validity of the file post upload.
4. Perform the upgrade using the install command.

Playbooks should be run in sequence as listed above. 
