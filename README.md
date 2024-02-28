# VISMA
This is a BSPWM installation script for Kali or Parrot Linux. It should be noted that it can work on other Debian base distributions,however, the only ones that remain stable and which we will support will be Kali Linux and Parrot Linux.

> [!NOTE]
> Follow the steps as indicated, there are words with a number that lead you to a definition of the word, in case there is one that you don't know, you should look for information yourself. This manual does not contain anything that is too complicated or that the information does not appear on the Internet.

> [!IMPORTANT]
> The IP we use will not be the same as that of your network, which is why you should pay attention to those points. We also indicate when you should enter the IP of the machine as follows: `IP_OF_X`

Pivoting is a technique used in cybersecurity where we use one machine as an intermediary to access another, non-visible machine, as this image explains:

![pivoting](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/aada852c-a3e2-4f60-9b92-22e983738993)

Once we have explained how pivoting works, we are going to explain how we have been able to compromise a laboratory with 6 machines (1 Windows and 5 Linux) where there are 4 internal networks configured. Although before that we must prepare the environment.

### 1. [Aragog](https://github.com/Vicctoriaa/VISMA/blob/main/aragog.md)
This is the first machine with 2 flags. This has a network interface that will work on the same network segment as the attacking machine. It has another interface connected to VMNET 2 where the second machine in the pack is located in that same network segment: Nagini.

### 2. Nagini
This also has 2 network interfaces, One working on the network segment 10.10.0.0/24 (VMNET 2) and the other on 192.168.1.0/24 (VMNET 3). It consists of 3 flags (in total there are 8 flags (called “horocrux” due to the theme) distributed among the 3 machines. In VMNet 3 there is also Fawkes & Visma.

### 3. Fawkes
The third and last machine is located within the special Harry Potter pack. Once this machine is accessed, a Docker container with port forwarding will be shown (simply looking at the IP address), we must escape from the Docker container and jump to the real machine, from where we can jump to the Visma Machine (the order does not alter the result).





