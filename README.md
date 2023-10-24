# metal-esxi-customdata
This repository provides examples and tutorials about how to use Customdata on Equinix Metal to configure an ESXi install at deployment time.

## Example json file
The esxi-customdata-example.json file contains an example of the current Customdata format required to inject firstboot commands into the Metal ESXi provisioning process. We're working on enhancements to make this formatting easier, but this is what works today.

## Explanation of commands in firstboot_shell_cmd line of example json file
The esxi-customdata-firstboot_explanation.md file goes over each command from the example and provides an explanation of what each command does.

## Make editing the firstboot_shell_cmd easier
Editing the esxi-customdata-example.json file can be made easier by doing a find and replace to replace the ';\r\n' string with a ';newline', updating the commands with relevant entries for IP addresses, portgroup names, DNS/NTP Server addresses, hostname/FQDN, etc, then performing another find and replace to remove the ';newline' sting and putting ';\r\n' back in place.