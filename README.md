# Equinix Metal Customdata for ESXi
by: David Luke

This project provides examples and tutorials around the use of Customdata on Equinix Metal to configure an ESXi install at deployment time. This is best suited to deployments that will use Metal VLANs and the VCF/UV2 (Unbonded Vitualization L2) networking mode. However this same framework can be used to deploy ESXi with one of the traditional L3/Hybrid network modes as well. 

## Please Note
* __Do not__ leave an unprotected ESXi instance on the public internet, there are many known exploits for ESXi Management services that can lead to system compromise. 
 * It is highly recommended to move the ESXi Management interface behind a firewall at your earliest convenience.
* This project includes an example command to allow for Password Based SSH Authentication. This command in particular should be used with extreme caution for ESXi systems that will have internet facing Management services.
* This project is experimental and is intended for example purposes only. Please thoroughly read the [firstboot_shell_cmd-explanation.md](https://github.com/davidlukemt/metal-esxi-customdata/blob/main/firstboot_shell_cmd-explanation.md) file before using the example json file in this project

## What this project will do
* Provide an example of working Equinix Metal customdata for ESXi first boot customization
* Provide an explanation of all first boot commands in the example
* Provide tips for making the editing of the example file easier

## Example json file
The esxi-customdata-example.json file contains an example of the current Customdata format required to inject firstboot commands into the Metal ESXi provisioning process. We're working on enhancements to make this formatting easier, but this is what works today.

## Explanation of commands in firstboot_shell_cmd line of example json file
The firstboot_shell_cmd-explanation.md file goes over each command from the example and provides an explanation of what each command does.

## Make editing the firstboot_shell_cmd easier
Editing the esxi-customdata-example.json file can be made easier by doing a find and replace to replace the ';\r\n' string with a ';newline', updating the commands with relevant entries for IP addresses, portgroup names, DNS/NTP Server addresses, hostname/FQDN, etc, then performing another find and replace to remove the ';newline' sting and putting ';\r\n' back in place.

### Find and Replace Examples

#### Notepad++
Here are some examples of how to convert from Single Line to Multi-Line and back using Notepad++

##### Single to Multi
Find: ``;\\r\\n``

Replace: ``;\r\n``

Before:

![Alt text](assets/1-npp_to_multi-line_before.png?raw=true "Notepad++ Single to Multi - Before")

After:

![Alt text](assets/2-npp_to_multi-line_after.png?raw=true "Notepad++ Single to Multi - After")

##### Multi to Single
Find: ``;\r\n``

Replace: ``;\\r\\n``

Before:

![Alt text](assets/3-npp_to_single-line_before.png?raw=true "Notepad++ Single to Multi - Before")

After:

![Alt text](assets/4-npp_to_single-line_after.png?raw=true "Notepad++ Single to Multi - After")

#### VSCode
Here are some examples of how to convert from Single Line to Multi-Line and back using VSCode.

##### Single to Multi
Find: ``;\\r\\n``

Replace: ``;\n``

Before:

![Alt text](assets/5-vscode_to_multi-line_before.png?raw=true "Notepad++ Single to Multi - Before")

After:

![Alt text](assets/6-vscode_to_multi-line_after.png?raw=true "Notepad++ Single to Multi - After")

##### Multi to Single
Find: ``;\n``

Replace: ``;\\r\\n``

Before:

![Alt text](assets/7-vscode_to_single-line_before.png?raw=true "Notepad++ Single to Multi - Before")

After:

![Alt text](assets/8-vscode_to_single-line_after.png?raw=true "Notepad++ Single to Multi - After")

#### sed CLI
Here are some examples of how to convert from Single Line to Multi-Line and back using sed from the Linux CLI.

``sed -i 's/;\\r\\n/;\n/g' esxi-customdata-example_sed.json``

``sed -z 's/;\n/;\\r\\n/g' esxi-customdata-example_sed.json > esxi-customdata-example_sed-singleline.json``

## Custom root password

### Generating custom root password
To generate a password hash of your desired ESXi root password run the 'mkpasswd' command on a Linux system with the 'whois' package installed as follows

```shell
mkpasswd --method=SHA-512
```
You'll be prompted to enter the desired password sting you wish to hash, then press enter. The output will be the string you need to use in the rootpwcrypt entry near the end of the esxi-customdata-example.json file