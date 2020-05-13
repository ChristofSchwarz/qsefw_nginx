# Using NGINX in front of Qlik Sense Enterprise for Windows

NGINX (pronounced "Engine X") is a web server and reverse proxy, load balancer and more. NGINX is free and open-source 
software and has become extremely popular. In Feb 2020, estimated Netcraft, NGIX served over 36% of all active websites (!!).

We will use it "in front of" our Windows Qlik Sense Server to expose other services (maybe a self-written one like my Qlik
Sense LDAP Login https://github.com/ChristofSchwarz/qsefw_ldap_login or Qlik's QPS API), which typically run on other ports 
than 443, as a route.

<img src="https://github.com/ChristofSchwarz/pics/raw/master/nginx.png" width="530"/>

By default, the https port 443 is taken by Qlik Sense Proxy Service already, so we have to move it to another port. We can 
also enable http traffic internally between NGINX and Qlik Sense.

Follow these steps
 - Log into Qlik Sense QMC, Proxies settings on https://localhost/qmc/proxies
 - Edit the "Ports" section of the page like this:
 <img src="https://github.com/ChristofSchwarz/pics/raw/master/nginx_qmc.png"/>
 - Download nginx Stable Version for Windows from http://nginx.org/en/download.html
 - Extract all files into a new folder e.g. `C:\nginx`
 - Download the [/conf/nginx.conf](https://raw.githubusercontent.com/ChristofSchwarz/qsefw_nginx/master/conf/nginx.conf) file and overwrite the default file in your local installation on `C:\nginx\conf\nginx.conf`

Note: after that port change in the QMC, you can (unless NGINX is installed later) no more reach the QMC or Hub
under https://localhost/ but either under https://localhost:444/ or http://localhost

## Run NGINX from Command Prompt

To see if any errors will occur, open an elevated Command Prompt (as Administrator) and go to your NGINX folder.
Run `nginx.exe`. It will pick the default config file from /conf/nginx.conf ... 

 - If you see no errors, you are all set.
 - To stop nginx again, there is no Ctrl+C or other keyboard combination. You have to start Task Manager in Windows
 - Kill two NGINX processes (it always runs with 2 processes at least) from Task Manager
 - Then you will get the prompt again in your Command Prompt window
 
## Making NGINX a Windows Service using NSSM

Although NSSM ("Non-sucking Service Manager") is old and hasn't seen updates for a while, it is what it is: 
An amazingly light-weight executeable that you run **once** in order to register a given .exe (nginx.exe) as 
a Windows Service. After you do that, you will be able to start/stop and auto-start NGINX as a Service and
don't need NSSM again.

- Download nssm.exe from http://nssm.cc and save the Win64 version of nssm.exe somewhere in the nginx folder e.g. in `C:\nginx\nssm\nssm.exe` (alternatively get the Win64 version straight from [this repository](https://github.com/ChristofSchwarz/qsefw_nginx/raw/master/nssm/nssm.exe))
- Run an elevated Command Prompt (run as Administrator) 
- Execute this command: `C:\nginx\nssm\nssm.exe install`
- Set the parameters on the first page of NSSM. The other settings can be as default or change if you know what you are doing.
<img src="https://github.com/ChristofSchwarz/pics/raw/master/nssm.png" width="440"/>
- Use the "Services" window to start the new NGINX service or simply type `net start "NGINX"` from your open command prompt

### uninstalling NGINX service

- Run an elevated Command Prompt (run as Administrator)
- Stop the NGINX Service in Windows or type `net stop "NGINX"` in the command prompt
- Execute this command: `C:\nginx\nssm\nssm.exe remove NGINX`

## Understand the settings for Qlik Sense on Windows

In this nginx.conf we are routing everything to http://localhost, so any path like /hub, /qmc, /sense, ... will end up on 
port 80. But to the outside world, the nginx should use certificates and https. For sake of simplicity, I am reusing the
certificates server.pem and server_key.pem that Qlik Sense has created during installation and saved to 
`C:\ProgramData\Qlik\Sense\Repository\Exported Certificates\.Local Certificates`

Note: It is not necessary to introduce a separate Virtual Proxy to have a main parent path after the host name in your url.


## Secure API route with basic authentication

NGINX is able to secure certain paths for which it is proxy. We would like to expose Qlik's QPS API under route /api/qps but 
use at least a basic authentication with username+password. See https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/

The 
https://www.web2generators.com/apache-tools/htpasswd-generator
