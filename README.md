# OpenShift egress router in dns mode on CRC

## Start CRC
```bash
$ crc start
```

## Deploy test web server
```bash
$ eval $(crc podman-env)
$ podman run -d -p 8080:80 docker.io/nginx
```

## Deploy egress router
```bash
$ oc debug node/$(oc get nodes  --template='{{$n := index .items 0}}{{$n.metadata.name}}')
sh-4.4# chroot /host
sh-4.4# ip route | grep default
default via 192.168.130.1 dev enp2s0 proto dhcp src 192.168.130.11 metric 100

$ GTW_IP="192.168.130.1"
$ SRC_IP="$(crc ip | cut -d. -f1-3).$(($(crc ip | cut -d. -f4)+1))"
$ DST_IP="$(crc ip)"
$ DST_PORT="8080"
$ oc process -f template.yaml -p GATEWAY_IP="${GTW_IP}" -p SOURCE_IP="${SRC_IP}" -p DESTINATION_IP="${DST_IP}" -p DESTINATION_PORT="${DST_PORT}" | oc -n default apply -f -
```

## Test egress router
```bash
$ oc -n openshift-ingress exec deploy/router-default -- curl -s http://egress.default:8080
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
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Troubleshooting
```bash
$ oc logs egress -c egress-router
```

## Links
- [Using Egress Router](https://docs.openshift.com/container-platform/4.12/networking/openshift_sdn/using-an-egress-router.html)
- [Deploying Egress Router in DNS mode](https://docs.openshift.com/container-platform/4.12/networking/openshift_sdn/deploying-egress-router-dns-redirection.html)
- [Egress Router tutorial](https://cloud.redhat.com/blog/accessing-external-services-using-egress-router)
- [Introduction to MacVLAN](https://dockerlabs.collabnix.com/intermediate/macvlan.html)
