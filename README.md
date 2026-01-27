# ingress-nginx
A repo for building Chainguard's fork of ingress-nginx.

## Using these images

We're currently hosting these built images at [porterhub/controller](https://hub.docker.com/r/porterhub/controller). Due to the way the Makefile for `ingress-nginx` functions, we're stuck with using `controller` as the repo name on Docker Hub, since the Makefile typically expects the owner name to be `ingress-nginx`. This isn't something I'm interesting in tweaking atm. The [build process](./.github/workflows/build.yml) pulls in [Chainguard's fork of `ingress-nginx`](https://github.com/chainguard-forks/ingress-nginx), and builds off the `helm-chart-4.12.1` tag and builds multi-arch images. More up-to-date tags will be added gradually.

To reconfigure `ingress-nginx` on a customer's cluster to use this image, simply modify the default `controller.image` block so that it looks like this:

```yaml
controller:
  image:
    registry: docker.io
    image: porterhub/controller
    tag: "v1.12.1"
    digest: null
```

## Supported versions

We currently build the following versions:

| Helm Chart Version | Controller Image Tag | Image Repository |
|--------------------|---------------------|------------------|
| 4.11.5             | v1.11.5             | porterhub/controller |
| 4.12.1             | v1.12.1             | porterhub/controller |