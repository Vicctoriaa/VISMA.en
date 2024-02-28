## Aragog machine
We start with the Aragog machine, since it is the one found on our network. First we must find it and to do this we will need to become root.
If you want to learn how to violate it, follow the steps as indicated.

# 1. NETWORK
```
sudo nano /etc/network/interface
```
It has to look like the image shows.
![1 cap_red](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/19421e89-5d44-4ce0-bdd0-725cd4991c59)

In case we want to know what we have put in the interface we can use the following command:
```
cat /etc /network/interface
```
We restart the machine, and we go to our main machine, it is important to have the Aragog turned on so that it can find our IP, to know what IP it has we will scan the network
putting the following command:
```
arp-scan -I ens33 –localnet 
```
If we only want to search for the machine, and we are using VmWare, we can add the `grep` command and put the following in quotes:
```
arp-scan -I ens33 –localnet | grep "VMware, Inc."
```
![segunda cap_arp scan](https://github.com/Vicctoriaa/VISMA/assets/153718557/0adfdaf6-8c45-4ebb-8f52-129be138e098)

# 2. ATTACK

We will check if the machine is active or if there is a firewall blocking ICMP traces, to do this we must do a 'ping'.
```
ping -c 1 IP_OF_THE_ARAGOG
```
![tercera cap_ping](https://github.com/Vicctoriaa/VISMA/assets/153718557/a16ed55c-d8de-43e8-aeab-ed765bd07e2c)

Once we have sent a ping, we can check that the machine is on, we also see that the ttl = 64 so it is a Linux machine, if the ttl is “=” or less than 64 it means that we are probably facing a Linux machine.
We can also observe that no packet has been discarded, so we already know that it is in the same network segment and is ready to be compromised.

Once we have that information, we will have to do a port scan. For this we will use the nmap tool, specialized in scanning ports, we will set parameters since we only want specific things. (if you want to know about the parameters go here:.....)  



