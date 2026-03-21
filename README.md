# Spin Hello JS

This app was created with the official Spin CLI `http-js` template and is meant
as the simplest working SpinKube smoke test in this repository.

Public traffic reaches provisioned services through Traefik, which terminates
TLS and forwards plain HTTP to the internal KEDA HTTP add-on interceptor.

## Build

From this directory:

```bash
spin build
```

## Push to ttl.sh

Use a unique image name each time because `ttl.sh` images are temporary:

```bash
IMAGE=ttl.sh/aido-spin-hello-js-$(date +%s):1h
spin registry push "$IMAGE"
```

## Use this image in provisioning

1. Update `charts/metacontroller-hooks/values.yaml` and set:

```bash
config:
  spinImage: ttl.sh/aido-spin-hello-js-<timestamp>:1h
```

2. Sync hooks service:

```bash
helmfile -l name=metacontroller-hooks sync
```

3. Trigger a new service host and verify:

```bash
SERVICE=test$(date +%H%M%S)
curl -k https://$SERVICE.ernoaapa.haara.run/
kubectl -n project-ernoaapa get site,spinapp,httpscaledobject | grep "$SERVICE"
```

## Notes

- `ttl.sh` images expire, so redeploys require building and pushing a fresh image.
- Public traffic terminates at Traefik, not at the KEDA interceptor directly.
- The KEDA HTTP add-on is currently beta upstream.
- Trusted HTTPS for this sample is issued by cert-manager over ACME HTTP-01.
- The `hetzner-dns-api-key` secret is only needed if you later want DNS-01 certificates such as wildcard certs.
