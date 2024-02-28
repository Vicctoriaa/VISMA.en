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
```
nmap -sS -p --open -T5 --min-rate 5000 IP_DE_LA_ARAGOG -n -Pn -vvv -oG
```
![4 cap_nmap1](https://github.com/Vicctoriaa/VISMA/assets/153718557/9f0cecac-277b-46ab-96c7-0a7fdd210233)

Once we have finished with the port scan, we must now detect which services or versions are running on these ports. To do this, once we have the open ports copied to the clipboard
```
nmap -sCV -p22,80 IP_DE_LA_ARAGOG -oN portServices
```
![5 cap_nmap2](https://github.com/Vicctoriaa/VISMA/assets/153718557/801bb9f3-6416-44bd-a619-4440f6e2bd8a)

Once the scan is done we find that port 22 is running ssh, but since we do not have any key at the moment we cannot do anything and a brute force attack would take a long time, so we leave it as the last option. We find that Apache is running on port 80, which means that there is a website running behind it.

If we look for the IP of the machine with port 80 we find the following

![6 cap_ipnavegadorWEB](https://github.com/Vicctoriaa/VISMA/assets/153718557/7f000f25-3155-43a7-8403-493871ef5ca3)

At first we don't see anything that catches our attention so let's look at the source code to see if there is anything interesting, to do this we will click on ctrl + u

![7 cap_ctrl_U_web](https://github.com/Vicctoriaa/VISMA/assets/153718557/1aee44e7-258b-4fa8-8144-20a09f3a22a8)

And we found nothing. The next thing to test would be the subdomains, but here we do not have a domain or DNS server to ask the possible subdomains or subdomains indexed by the SSL certificate. So we will do fuzzing, this will help us determine if there is a hidden directory within the web server. To do this we will use `gobuster`, although we can also use tools like `fuff` or `wfuzz`.
(In case it is not installed you can see how it is installed here......and to know what each parameter is, go here......)
```
gobuster dir -u http://IP_DE_LA_ARAGOG:80 -w /usr/share/Discovery/Web-Content/directory-list-2.3-medium.txt -t 400 -x html,php,jpg,png,png,txt,docx,pdf 2>/dev/null
```
![8 cap_gobuster1](https://github.com/Vicctoriaa/VISMA/assets/153718557/36899e51-7b8a-491c-96d1-059992d80b2d)

Once the scan is finished we find that the directory “/blog” and “/javascript” redirect us elsewhere while “/server-status” returns a 403 (the server receives the request but denies access to the action), yes We enter the java script, it will give us an error called FORBIDDEN, however if we enter the blog we see the website

![9 cap_web blog](https://github.com/Vicctoriaa/VISMA/assets/153718557/30f6ed9f-d43e-412f-b8f2-8c3b7a7ff6cf)

We can see that it is a Wordpress[^1] page, but for some reason the website is not loading the “css”[^2] styles, so first we are going to review the source code.
First, looking for comments from a developer that gives us valuable information and then we will search the "head" to see what happens and why it does not call the CSS style correctly, as we did previously, ctrl + u

![10 cap_CSS](https://github.com/Vicctoriaa/VISMA/assets/153718557/aff25656-3d4f-40ba-9d19-e90a5d6bd107)

Since it calls the styles from a domain, in order to resolve it we must add the domain wordpress.aragog.hogwarts and aragog.hogwarts, so that our PC is able to resolve the domain, in the following directory:
```
sudo nano /etc/hosts
```
![11 cap_poner dominio CSS](https://github.com/Vicctoriaa/VISMA/assets/153718557/7465ef38-2401-4c8a-ba4b-d621605aa587)

Now if we search for the domain “wordpress.aragog.hogwarts” in the browser, the website will appear correctly.

This is what wappalyzer[^3] reports to us on the web





















[^1]: It has Wordpress Comment and Wp-Admin which is a very common wordpress user apart from the fact that it says “Proudly powered by Wordpress”.
[^2]: language that manages the design and presentation of web pages, that is, how they look when a user visits them.
[^3]: It inspects the internal data of the websites, identifying the programming that has been used to develop it. It is also an extension that we have to add by searching for it directly from the Internet.



