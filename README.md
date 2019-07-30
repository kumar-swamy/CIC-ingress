Kubernetes Ingress gives you a way to route requests to services based on the request host or path, centralizing a number of services into a single entry point.
Citrix ingress controller is built around Kubernetes Ingress and automatically configures one or more Citrix ADC based on the Ingress resource configuration.

# Hostname based routing
This example shows setting up ingress to route the traffic based on hostname.

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: Virtual-Host-Ingress
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: service1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: service2
          servicePort: 80
```
All HTTP requests with host header “foo.bar.com”, Citrix ADC  load-balances to service1 and HTTP requests with host header “bar.foo.com” will be loadbalanced to service2.

# Path based Routing
This example shows setting up ingress to route traffic based on path

```
apiVersion: extension/v1beta1
kind: Ingress
metadata:
  name: Path-Ingress
  namespace: default
spec:
  rules:
  - host:test.example.com
    http:
      paths:
      - path: "/foo"
        backend:
          serviceName: service1
          servicePort: 80
      - path: "/"
        backend:
          serviceName: service2
          servicePort: 80
```
Any HTTP requests with host `test.example.com` and URL path with prefix `/foo` is routed to service1 and all other requests will be routed to service2.
Citrix ingress controller will follow first match policy to evaluate paths. For effective matching, Citrix ingress controller will order the paths based on descending order of length of paths.
CIC will also order the paths belonging to same hosts across multiple ingress resources.

# Wildcard host routing
Following example shows configuring ingress with wildcard host.

```
apiVersion: extension/v1beta1
kind: Ingress
metadata:
  name: Wildcard-Ingress
  namespace: default
spec:
  rules:
  - host:’*.example.com’
    http:
      paths:
      - path: "/"
        backend:
          serviceName: service1
          servicePort: 80
```
HTTP requests to all subdomains of example.com is routed to service1.
Note: Rules with non-wildcard hosts are given higher priority than wildcard hosts. Among different wildcard hosts, rules are ordered on the descending order of length of hosts.


# Exact path matching
By default Ingress paths are treated as prefix expressions. Adding annotation ingress.citrix.com/path-match-method: “exact” makes CIC treat the path for exact match.

```
apiVersion: extension/v1beta1
kind: Ingress
metadata:
  name: Path-exact-Ingress
  namespace: default
  annotations:  ingress.citrix.com/path-match-method: “exact”
spec:
  rules:
  - host:test.example.com
    http:
      paths:
      - path: /exact
        backend:
          serviceName: service1
          servicePort: '80'
```
HTTP requests with path `/exact` is routed to service1 but not `/exact/somepath`

# Non-Hostname routing
Following example shows path based routing for the default traffic whoch doesn’t match any host based routes. This rule applies to all inbound HTTP traffic through the IP address specified.

```
apiVersion: extension/v1beta1
kind: Ingress
metadata:
  name: Default-Path-Ingress
  namespace: default
spec:
  rules:
-	http:
      paths:
      - path: "/foo"
        backend:
          serviceName: service1
          servicePort: 80
      - path: "/"
        backend:
          serviceName: service2
          servicePort: 80
```
All incoming traffic which doesn't match ingress rules with hostname is matched here for the paths for routing.


# Default backend
Default backend is a service which handles all traffic that is not matched against any of the Ingress rules.
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: Default-Ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80

```
Note: A global default backend can be specified if CPX is loadbalancing your traffic.You can create a default backend per frontend-ip:port combination  in case of  VPX/MPX  is the ingress device.

