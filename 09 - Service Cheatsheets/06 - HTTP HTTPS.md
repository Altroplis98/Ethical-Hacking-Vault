---
tags: [pentest, cheatsheet, http, https, web, service, both]
port: [80, 443, 8080, 8443]
phase: reference
---
# HTTP / HTTPS (80 / 443 / 8080 / 8443)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Web app surface — the largest and most variable attack surface on any modern target. Start with directory brute force and tech fingerprinting; look for version disclosure in headers and admin panels. HTTP = cleartext (sniff anything you can). HTTPS = same attack surface inside the tunnel plus weak TLS / cert info to enumerate. **Alt ports (8080/8443)** are dev servers, admin panels, Tomcat, Jenkins, Jupyter — often less hardened than 80/443 with default creds extremely common. **Common attack vectors:** directory/file brute force, SQLi, XSS, LFI/RFI, default creds, exposed admin panels, weak TLS, CVE exploits on identified frameworks, Tomcat WAR upload, Jenkins Groovy console, Jupyter unauthenticated notebook execution.

## Enumerate

```bash
nmap -sV -sC -p 80,443,8080,8443 $IP
whatweb http://$IP
curl -I http://$IP   # headers
```

## Directory brute-force

```bash
gobuster dir -u http://10.129.3.94 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,txt
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
feroxbuster -u http://$IP
```

## Vhost discovery

```bash
ffuf -u http://$IP -H "Host: FUZZ.domain.com" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <default_size>
gobuster vhost -u http://$IP -w subdomains.txt --append-domain
```

## Technology fingerprinting

```bash
whatweb http://$IP
nikto -h http://$IP
wafw00f http://$IP
```

## Check for common files

```bash
curl http://$IP/robots.txt
curl http://$IP/sitemap.xml
curl http://$IP/.htaccess
curl http://$IP/crossdomain.xml
curl http://$IP/wp-config.php.bak
curl http://$IP/.git/HEAD
curl http://$IP/.env
```

## CMS-specific

```bash
wpscan --url http://$IP -e u,vp   # WordPress
joomscan -u http://$IP             # Joomla
droopescan scan drupal -u http://$IP  # Drupal
```

## Vulnerability scanning

```bash
nikto -h http://$IP
nuclei -u http://$IP -s critical,high
```
