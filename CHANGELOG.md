# Changelog

## Table of Contents

- [v0.4.0](#v040)
- [v0.3.0](#v030)
- [v0.2.0](#v020)
- [v0.1.1](#v011)
- [v0.1.0](#v010)

## v0.4.0 

> Release date: 2022-01-25

### Added

- Added machinery for ControlPlanes to communicate with DataPlanes
  directly via Pod IPs. The Admin API has been removed from the LoadBalancer service.
  [#609](https://github.com/Kong/gateway-operator/pull/609)
- The Gateway Listeners status is set and kept up to date by the Gateway controller.
  [#627](https://github.com/Kong/gateway-operator/pull/627)

## v0.3.0

> Release date: 2022-11-30

**Maturity: ALPHA**

### Changed

- Bumped DataPlane default image to 3.0.1
  [#561](https://github.com/Kong/gateway-operator/pull/561)

### Added

- Gateway statuses now include all addresses from their DataPlane Service.
  [#535](https://github.com/Kong/gateway-operator/pull/535)
- Dataplane Deployment strategy enforced as RollingUpdate.
  [#537](https://github.com/Kong/gateway-operator/pull/537)

### Fixes

- Regenerate DataPlane's TLS secret upon deletion
  [#500](https://github.com/Kong/gateway-operator/pull/500)
- Gateway statuses no longer list cluster IPs if their DataPlane Service is a
  LoadBalancer.
  [#535](https://github.com/Kong/gateway-operator/pull/535)

## v0.2.0

> Release date: 2022-10-26

**Maturity: ALPHA**

### Added

- Updated default Kong version to 3.0.0
- Updated default Kubernetes Ingress Controller version to 2.7
- Update DataPlane and ControlPlane Ready condition when underlying Deployment
  changes Ready condition
  [#451](https://github.com/Kong/gateway-operator/pull/451)
- Update DataPlane NetworkPolicy to match KONG_PROXY_LISTEN and KONG_ADMIN_LISTEN
  environment variables set in DataPlane
  [#473](https://github.com/Kong/gateway-operator/pull/473)
- Added Container image and version validation for ControlPlanes and DataPlanes.
  The operator now only supports the Kubernetes-ingress-controller (2.7) as
  the ControlPlane, and Kong (3.0) as the DataPlane.
  [#490](https://github.com/Kong/gateway-operator/pull/490)
- DataPlane resources get a new `Status` field: `Addresses` which will contain
  backing service addresses.
  [#483](https://github.com/Kong/gateway-operator/pull/483)

## v0.1.1

> Release date:  2022-09-24

**Maturity: ALPHA**

### Added

- `HTTPRoute` support was added. If version of control plane image is at 
  least 2.6, the `Gateway=true` feature gate is enabled, so the 
  control plane can pick up the `HTTPRoute` and configure it on data plane.
  [#302](https://github.com/Kong/gateway-operator/pull/302)  

## v0.1.0

> Release date: 2022-09-15

**Maturity: ALPHA**

This is the initial release which includes basic functionality at an alpha
level of maturity and includes some of the fundamental APIs needed to create
gateways for ingress traffic.

### Initial Features

- The `GatewayConfiguration` API was added to enable configuring `Gateway`
  resources with the options needed to influence the configuration of
  the underlying `ControlPlane` and `DataPlane` resources.
  [#43](https://github.com/Kong/gateway-operator/pull/43)
- `GatewayClass` support was added to delineate which `Gateway` resources the
  operator supports.
  [#22](https://github.com/Kong/gateway-operator/issues/22)
- `Gateway` support was added: used to create edge proxies for ingress traffic.
  [#6](https://github.com/Kong/gateway-operator/issues/6)
- The `ControlPlane` API was added to deploy Kong Ingress Controllers which
  can be attached to `DataPlane` resources.
  [#5](https://github.com/Kong/gateway-operator/issues/5)
- The `DataPlane` API was added to deploy Kong Gateways.
  [#4](https://github.com/Kong/gateway-operator/issues/4)
- The operator manages certificates for control and data plane communication
  and configures mutual TLS between them. It cannot yet replace expired
  certificates.
  [#103](https://github.com/Kong/gateway-operator/issues/103)

### Known issues

When deploying the gateway-operator through the bundle, there might be some
leftovers from previous operator deployments in the cluster. The user needs to delete all the cluster-wide leftovers
(clusterrole, clusterrolebinding, validatingWebhookConfiguration) before
re-installing the operator through the bundle.

[v0.4.0]: https://github.com/Kong/gateway-operator/tree/v0.4.0
[v0.3.0]: https://github.com/Kong/gateway-operator/tree/v0.3.0
[v0.2.0]: https://github.com/Kong/gateway-operator/tree/v0.2.0
[v0.1.1]: https://github.com/Kong/gateway-operator/tree/v0.1.1
[v0.1.0]: https://github.com/Kong/gateway-operator/tree/v0.1.0
