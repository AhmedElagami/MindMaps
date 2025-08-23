# Ingress Routing

## Host-based Routing

## Content

### ❓ What does the diagram in Figure 8.1 illustrate about host-based routing with Ingress?
![Figure 8.1 Host-based routing](media/figure8-1.png)

Figure 8.1 shows two requests hitting the same cloud load balancer. Behind the scenes, DNS name resolution maps both hostnames to the same load balancer IP. An Ingress controller watches the load balancer and routes the requests based on the hostnames in the HTTP headers. In this example, it routes shield.mcu.com to the shield ClusterIP Service, and hydra.mcu.com to the hydra Service.

### ❓ Explain what the host-based Ingress rule configuration does, including the hostname, path, and backend service details, and what conditions trigger it and where it routes traffic.
The host-based rule triggers on traffic arriving via the hostname `shield.mcu.com` at the root "/" path and forwards it to the ClusterIP back-end Service called `svc-shield` on port 8080.

```bash
- host: shield.mcu.com            <<---- Traffic arriving via this hostname
  http:
    paths:
    - path: /                     <<---- Arriving at root (no subpath specified)
      pathType: Prefix
      backend:                    <<---- The next five lines send traffic to an
        service:                  <<---- existing "backend" ClusterIP Service
          name: svc-shield        <<---- called "svc-shield"
          port:                   <<---- that's listening on
            number: 8080          <<---- port 8080
```


## Path-based Routing
## Content

### ❓ Explain what the path-based Ingress rule configuration does, including the hostname, specific path, and backend service details.
The path-based rule triggers when traffic arrives on the hostname `mcu.com` at the subpath `/shield`. It gets routed to the same `svc-shield` back-end Service on port 8080.

```bash
- host: mcu.com                 <<---- Traffic arriving via this hostname
    http:
      paths:
      - path: /shield             <<---- Arriving on this subpath
        pathType: Prefix
        backend:                  <<---- The next five lines send traffic to an
          service:                <<---- existing "backend" ClusterIP Service
            name: svc-shield      <<---- called "svc-shield"
            port:                 <<---- that's listening on
              number: 8080        <<---- port 8080
```

