
## Errors starting k3s
```
Dec 16 01:36:04 k3s-worker001 k3s[2805]: time="2019-12-16T01:36:04.334705566Z" level=info msg="Running load balancer 127.0.0.1:42149 -> [k3smaster:6443
Dec 16 01:36:04 k3s-worker001 k3s[2805]: time="2019-12-16T01:36:04.352606606Z" level=error msg="failed to get CA certs at https://127.0.0.1:42149/cacer
Dec 16 01:36:06 k3s-worker001 k3s[2805]: time="2019-12-16T01:36:06.372783700Z" level=error msg="failed to get CA certs at https://127.0.0.1:42149/cacer
```

## Try to pull the certs manually from the host
`pi@k3s-worker001:~ $ ping mak3r`
```
PING mak3r.lan (192.168.8.210) 56(84) bytes of data.
64 bytes from mak3r.lan (192.168.8.210): icmp_seq=1 ttl=64 time=3.20 ms
64 bytes from mak3r.lan (192.168.8.210): icmp_seq=2 ttl=64 time=5.51 ms
64 bytes from mak3r.lan (192.168.8.210): icmp_seq=3 ttl=64 time=2.95 ms
64 bytes from mak3r.lan (192.168.8.210): icmp_seq=4 ttl=64 time=5.07 ms
^C
--- mak3r.lan ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 2.956/4.189/5.518/1.125 ms
```
`pi@k3s-worker001:~ $ curl --insecure https://192.168.8.210:6443/cacerts`
```
curl: (35) Unknown SSL protocol error in connection to 192.168.8.210:6443
```

## It seems like trying to use the router to change the hostname is not working
* k3smaster <> mak3r
* Things to try:
    - remove the hostname from the WAP
    - use `mak3r` hostname when starting k3s server
    - retry curling the certs from mak3r
    - passthrough 80 and 443 in the nginx LB config
    
