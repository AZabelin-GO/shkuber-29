# Solution 02

## Task 01

0. Start minikube using commands: `minikube start` and `minikube addons enable ingress`
1. Deployment - `src-01/deployment.yaml`
2. Services - clusterIP: `src-01/service-ew.yaml`; nodePort: `src-01/service-ns.yaml`
3. To check E/W connection deploy pods `src-01/pods.yaml` and see their logs using commands: `kubectl logs pod/network-check-nginx` and `kubectl logs pod/network-check-multitool`
4. To check N/S connection in Minikube use commands: `minikube ssh curl $(minikube ip):32001` and `minikube ssh curl $(minikube ip):32002`
5. Results:

```sh
azabelin@ess-fearless 02 % kubectl logs pod/network-check-nginx
=== Check connection to Nginx ===
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host task01-ew:9001 was resolved.
* IPv6: (none)
* IPv4: 10.97.48.208
*   Trying 10.97.48.208:9001...
* Connected to task01-ew (10.97.48.208) port 9001
* using HTTP/1.x
> GET / HTTP/1.1
> Host: task01-ew:9001
> User-Agent: curl/8.14.1
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx/1.31.5
< Date: Tue, 15 Sep 2026 11:12:49 GMT
< Content-Type: text/html
< Content-Length: 896
< Last-Modified: Wed, 02 Sep 2026 11:17:13 GMT
< Connection: keep-alive
HTTP/1.1 200 OK
Server: nginx/1.31.5
Date: Tue, 15 Sep 2026 11:12:49 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Wed, 02 Sep 2026 11:17:13 GMT
Connection: keep-alive
ETag: "6a9805b9-380"
Accept-Ranges: bytes

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
< ETag: "6a9805b9-380"
< Accept-Ranges: bytes
< 
{ [896 bytes data]
100   896  100   896    0     0   280k      0 --:--:-- --:--:-- --:--:--  437k
* Connection #0 to host task01-ew left intact
azabelin@ess-fearless 02 % 
```

```sh
azabelin@ess-fearless 02 % kubectl logs pod/network-check-multitool
=== Check connection to Multitool ===
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host task01-ew:9002 was resolved.
* IPv6: (none)
* IPv4: 10.97.48.208
*   Trying 10.97.48.208:9002...
* Connected to task01-ew (10.97.48.208) port 9002
* using HTTP/1.x
> GET / HTTP/1.1
> Host: task01-ew:9002
> User-Agent: curl/8.14.1
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx/1.28.0
< Date: Tue, 15 Sep 2026 11:12:48 GMT
< Content-Type: text/html
< Content-Length: 140
< Last-Modified: Tue, 15 Sep 2026 11:06:50 GMT
< Connection: keep-alive
< ETag: "6aa926ca-8c"
< Accept-Ranges: bytes
< 
{ [140 bytes data]
100   140  100   140    0     0  53598      0 --:--:-- --:--:-- --:--:--  136k
HTTP/1.1 200 OK
* Connection #0 to host task01-ew left intact
Server: nginx/1.28.0
Date: Tue, 15 Sep 2026 11:12:48 GMT
Content-Type: text/html
Content-Length: 140
Last-Modified: Tue, 15 Sep 2026 11:06:50 GMT
Connection: keep-alive
ETag: "6aa926ca-8c"
Accept-Ranges: bytes

WBITT Network MultiTool (with NGINX) - task01-c78845698-pl5vb - 10.244.0.9 - HTTP: 8080 , HTTPS: 8443 . (Formerly praqma/network-multitool)
azabelin@ess-fearless 02 % 
```

```sh
azabelin@ess-fearless 02 % minikube ssh curl $(minikube ip):32001  
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
azabelin@ess-fearless 02 % 
```

```sh
azabelin@ess-fearless 02 % minikube ssh curl $(minikube ip):32002
WBITT Network MultiTool (with NGINX) - task01-c78845698-vnn9g - 10.244.0.7 - HTTP: 8080 , HTTPS: 8443 . (Formerly praqma/network-multitool)
azabelin@ess-fearless 02 % 
```

## Task02

1. Deployments: `src-02/deployment-frontend.yaml` and `src-02/deployment-backend.yaml`
2. Services: `src-02/service-frontend.yaml` and `src-02/service-backend.yaml`
3. Ingress: `src-02/ingress.yaml`
4. Results:

```sh
azabelin@ess-fearless 02 % k apply -f src-02
deployment.apps/task02-backend created
deployment.apps/task02-frontend created
ingress.networking.k8s.io/task02 created
service/task02-backend created
service/task02-frontend created
azabelin@ess-fearless 02 % minikube ip 
192.168.49.2
azabelin@ess-fearless 02 % minikube ssh     
Linux minikube 7.0.12-linuxkit #1 SMP PREEMPT Thu Aug 27 14:02:21 UTC 2026 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
docker@minikube:~$ curl --resolve task02.com:80:192.168.49.2 http://task02.com/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
docker@minikube:~$ curl --resolve task02.com:80:192.168.49.2 http://task02.com/api
WBITT Network MultiTool (with NGINX) - task02-backend-7d8ff49cb7-l66q7 - 10.244.0.21 - HTTP: 8080 , HTTPS: 8443 . (Formerly praqma/network-multitool)
docker@minikube:~$ 
```
