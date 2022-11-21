- Put timeout=x on each request
- Patch in logging of interesting requests & responses into service. Use docker logs to view logs
- Dump database of service using scp and view with proper viewer

## Parallelize attacks
TODO: This froze for some reason
```python
from multiprocessing.pool import ThreadPool
p = ThreadPool(15)
p.map(exploit_single_ip, entries)
```


## Register and login with CSRF & SSL
```python
import requests
import ssl
import uuid
import urllib3

def get_login_session(ip):
    username = str(uuid.uuid4()).replace("-","")
    passwd = str(uuid.uuid4())
    ua = "Your mom" # TODO: change this to checker user agent
    referrrer = f"https://{ip}:4242/"
    s = requests.Session()
    s.headers.update({"User-Agent": ua})

    csrf = s.get(f"https://{ip}:4242/csrf", verify=False, timeout=5).json()["csrfToken"]
    
    res = s.post(f"https://{ip}:4242/register", json={
        "username": username,
        "password": passwd,
        "confirm": passwd
    }, headers={"X-CSRF-Token": csrf, "Referer": referrrer}, verify=False, timeout=5)
    assert res.status_code == 200

    csrf = s.get(f"https://{ip}:4242/csrf", verify=False, timeout=5).json()["csrfToken"]
    res = s.post(f"https://{ip}:4242/login", json={
        "username": username,
        "password": passwd,
    }, headers={"X-CSRF-Token": csrf, "Referer": referrrer}, verify=False, timeout=5)
    assert res.status_code == 200
    
    #print(s.cookies.get_dict())
    return s
```

## Connect to websocket with CSRF & SSL
```python
import ssl
import websocket
import uuid
import urllib3

s = get_login_session(ip) # requests session
ws = websocket.WebSocket(sslopt={"cert_reqs": ssl.CERT_NONE, "check_hostname": False})

ws.connect(f"wss://{ip}:4242/sock", cookie="; ".join(["%s=%s" %(i, j) for i, j in s.cookies.get_dict().items()]), header={"User-Agent": ua}, ping_timeout=5)

ws.send(d)
r = ws.recv()
ws.close()
```
