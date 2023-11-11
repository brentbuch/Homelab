# Home Assistant Install on Proxmox

This tutorial will show how to install a Home Assistant VM on ProxmoxVE. I will be using a Linux VM of Home Assistant OS. The VM is provided as a disk image, but Proxmox currently does not support direct upload of a disk image. The steps below detail how to work around this. 

## Create the VM in Proxmox

1. Login to Proxmox web interface and click "Create VM." Give the VM a name and an ID. For OS, select "Do not use any media." Under the "System" tab, Machine should be set to q35, BIOS set to UEFI, select where the VM should be stored and uncheck "Pre-enroll keys."
