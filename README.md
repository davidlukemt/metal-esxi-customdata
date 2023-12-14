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

![Alt text](assets/5-vscode_to_multi-line_before.png?raw=true "VSCode Single to Multi - Before")

After:

![Alt text](assets/6-vscode_to_multi-line_after.png?raw=true "VSCode Single to Multi - After")

##### Multi to Single
Find: ``;\n``

Replace: ``;\\r\\n``

Before:

![Alt text](assets/7-vscode_to_single-line_before.png?raw=true "VSCode Single to Multi - Before")

After:

![Alt text](assets/8-vscode_to_single-line_after.png?raw=true "VSCode Single to Multi - After")

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
You'll be prompted to enter the desired password sting you wish to hash, then press enter. 

![Alt text](assets/9-mkpasswd_example.png?raw=true "mkpasswd Example")

The output will be the string you need to use in the rootpwcrypt entry near the end of the esxi-customdata-example.json file

![Alt text](assets/10-mkpasswd_in_json.png?raw=true "mkpasswd Example in rootpwcrypt")

## Using final JSON in Metal Console

When provisioning an ESXi instance on the Equinix Metal console, select the Hardware Plan and Operating System like normal.
After setting the Hostname in the Select Number of Servers section, expand the Optional Settings and select the Custom Data tab.
Paste your modified JSON into the text box where it says "Paste your JSON content here", and hit the "Deploy Now" button.

![Alt text](assets/11-metal_console_Deploy_Now-custom_data.png?raw=true "Metal Console - Deploy Now")

![Alt text](assets/12-metal_console_Manage_Servers_Deploying.png?raw=true "Metal Console - Deploying")

After provisioning is complete, you should see that the settings you specified in your "firstboot_shell_cmd" block have been set.

Management IP seen on DCUI/SOS Console:

![Alt text](assets/13-Metal_SOS_DCUI.png?raw=true "Metal SOS - DCUI")

After assigning the appropriate Metal VLANs to the instance, you should be able to access the Host Web UI using the root password you specified to review the rest of the scripted config changes.

FQDN seen from ESXi Host Web UI:

![Alt text](assets/14-ESXi_WebUI-FQDN.png?raw=true "ESXi WebUI - FQDN")

NTP settings seen from ESXi Host Web UI:

![Alt text](assets/15-ESXi_WebUI-NTP.png?raw=true "ESXi WebUI - NTP")

DNS Server settings seen from ESXi Host Web UI:

![Alt text](assets/16-ESXi_WebUI-DNS.png?raw=true "ESXi WebUI - DNS")

VMK and Port Group settings seen from ESXi Host Web UI:

![Alt text](assets/17-ESXi_WebUI-VMK.png?raw=true "ESXi WebUI - VMK")

![Alt text](assets/18-ESXi_WebUI-Port_groups.png?raw=true "ESXi WebUI - Port Groups")