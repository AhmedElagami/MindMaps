# Path-based routing

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

