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
Continue through the prompts, selecting the defaults. The installation will continue. When the installation finishes, you will be shown the web interface address and the admin password. Select 'OK' and you are given the command to change the password. I recommend doing this right now.
```bash
sudo pihole -a -p [newpassword]
```
7. Enter the IP address of your pi-hole container and you'll be shown a login screen where you can enter the password you just set. You should now see the Pi-hole dashboard
<img width="1266" alt="pihole_dash" src="https://github.com/brentbuch/Homelab/assets/142106637/d4ae3fbf-bbf6-4199-b5c8-55b4ec019ecf">


## II. Install and Configure unbound
Now we will need to install unbound, which will allow us to use Pi-hole as our own recursive DNS server on our network. The documentation for the install can be found [here](https://docs.pi-hole.net/guides/dns/unbound/)

1. In the terminal of the Pi-Hole container, enter the command
```bash
sudo apt install unbound
```
2. Next we will need to create the configuration file for unbound. The path for the config file and the command to create the file is below:

<code>/etc/unbound/unbound.conf.d/pi-hole.conf</code>

```bash
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```
3. You should now be inside a blank nano document. Copy and paste the entire contents below into the document. Then use CTRL + X to save and exit. 
```yaml
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```
4. Restart the unbound service
```bash
sudo service unbound restart
```
5. Test that the unbound service is operational
```bash
dig pi-hole.net @127.0.0.1 -p 5335
```
Which should produce a result similar to below
<img width="698" alt="testdig" src="https://github.com/brentbuch/Homelab/assets/142106637/4e6ac24a-eb1f-46f2-900e-8f83534c2250">

## III. Configure Pi-hole to use unbound

1. Log in to the Pi-hole dashboard. Select 'Settings' from the left menu bar. On the settings page, select the 'DNS' tab. On this page, uncheck the boxes for Google. Under 'Custom 1 (IPv4)' enter the address below. Scroll down to the bottom and Save.
```
127.0.0.1#5335
```
<img width="1007" alt="piholedns" src="https://github.com/brentbuch/Homelab/assets/142106637/53428531-431b-4553-8b4a-fc7791d836a5">

## IV. Configure devices to use Pi-hole
Now we can add Pi-hole to our network devices to take advantage of ad-blocking and our new DNS service. You can do this in two different ways. You can add the address of our Pi-hole as the DNS server in your router's settings. This will enable Pi-hole for all devices on your network. The other way is to modify the DNS settings on individual devices you would like to utilize Pi-Hole. This is the method I will show below.

1. We can see that our Pi-Hole dashboard does not have any activity yet. Let's change that.
<img width="1025" alt="emptydash" src="https://github.com/brentbuch/Homelab/assets/142106637/85b4311b-df28-446b-9530-5abb649093c6">

2. On your device of choice, navigate to the connection settings for your home network. You may need to select 'Advanced' or 'Show details' depending on the device. Once there, replace the current settings with our Pi-hole address.
<img width="465" alt="wifidns" src="https://github.com/brentbuch/Homelab/assets/142106637/5def550b-9fd0-4c13-a77c-9f9bce033015"> 

3. Visit a site with a lot of ads. I chose Forbes.com. Notice how clean the page is.
<img width="1261" alt="forbesnoads" src="https://github.com/brentbuch/Homelab/assets/142106637/323b3cc1-c8f8-426e-a956-e8fd65933b34">

4. Back on our Pi-hole dashboard, we now see some queries and blocked activity.
<img width="1013" alt="blockdash" src="https://github.com/brentbuch/Homelab/assets/142106637/a502b401-1d7d-47d7-a395-948cff3a87c0">

