# Pi-hole w/ Recursive DNS
This project is to setup a Pi-Hole DNS sinkhole in docker. We will then add recursive DNS functionality using Unbound. It is prefered to setup Pi-hole on a static IP address as that address will need to be added to devices and you would need to update it everytime the IP address changes.
## Tools
- [Pi-hole](https://pi-hole.net/)
- [Unbound](https://nlnetlabs.nl/projects/unbound/about/)

## Environments Used
- Proxmox VE (8.0.3)
- LXC container
- Ubuntu Server (23.04)
- Docker

## I. Create the Container and Install Pi-hole
I use Proxmox, but VMWare, Virtualbox, Hyper-V, or others can be used. 
1. Click on create container.
```
Name:pihole
Pass: Super_Secret_Password
Template: ubuntu-23.04-standard_23.04-1_amd64.tar.zst
Storage: 8GB
CPU: 1 Core
Memory: 512MB
Network: 
 - Static: 192.168.1.250/24
 - Gateway: 192.168.1.1
```
2. Once the container is created, start the container and then access the console. Login as root with the password created during setup of the container
3. To avoid using root, add a user
```bash
adduser [username]
```
Fill in whatever info you want for the user, and set the password.
Then add the new user to the sudo group
```bash
adduser [username] sudo
```
Then switch to the new user that was created
```bash
su [username]
```
4. Update the Ubuntu container
```bash
sudo apt update && sudo apt upgrade -y
```
5. Next we need to install Curl, which we will use to install Pi-hole
```bash
sudo apt install curl -y
```
6. Install Pi-hole
```bash
curl -sSL https://install.pi-hole.net | bash
```
Continue through the prompts, selecting the defaults. The installation will continue. When the installation finishes, you will be shown the web interface address and the admin password. Select 'OK' and you are given the command to change the password
```bash
sudo pihole -a -p [newpassword]
```
7. Enter the IP address of your pi-hole container and you'll be shown a login screen where you can enter the password you just set. You should now see the Pi-hole dashboard


## II. Configuring
