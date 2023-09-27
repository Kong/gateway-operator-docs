# Changelog

## Table of Contents

- [v1.0.0](#v100)
- [v0.7.0](#v070)
- [v0.6.0](#v060)
- [v0.5.0](#v050)
- [v0.4.0](#v040)
- [v0.3.0](#v030)
- [v0.2.0](#v020)
- [v0.1.1](#v011)
- [v0.1.0](#v010)

## v1.0.0

> Release date: 2023-09-26

### Changes

- Operator managed subresources are now labelled with `gateway-operator.konghq.com/managed-by`
  additionally to the old `konghq.com/gateway-operator` label.
  The value associated with this label stays the same and it still indicates the
  type of a resource that owns the subresrouce.
  The old label should not be used as it will be deleted in the future.
  [#1098](https://github.com/Kong/gateway-operator/pull/1098)
- Enable `DataPlane` Blue Green rollouts controller by default.
  [#1106](https://github.com/Kong/gateway-operator/pull/1106)

### Fixes

- Fixes handling `Volume`s and `VolumeMount`s when customizing through `DataPlane`'s
  `spec.deployment.podTemplateSpec.spec.containers[*].volumeMounts` and/or
  `spec.deployment.podTemplateSpec.spec.volumes`.
  Sample manifests are updated accordingly.
  [#1095](https://github.com/Kong/gateway-operator/pull/1095)

## v0.7.0

> Release date: 2023-09-13

### Added

- Added `gateway-operator.konghq.com/service-selector-override` as the dataplane
  annotation to override the default `Selector` of both the admin and proxy services.
  [#921](https://github.com/Kong/gateway-operator/pull/921)
- Added deploying of preview Admin API service when Blue Green rollout strategy
  is enabled for `DataPlane`s.
  `DataPlane`'s `status.rollout.service` is updated accordingly.
  [#931](https://github.com/Kong/gateway-operator/pull/931)
- Added `gateway-operator.konghq.com/promote-when-ready` `DataPlane` annotation to allow
  users to signal the operator should proceed with promoting the new resources when
  `BreakBeforePromotion` promotion strategy is used.
  [#938](https://github.com/Kong/gateway-operator/pull/938)
- Added deploying of preview Deployment when Blue Green rollout strategy
  is enabled for `DataPlane`s.
  [#930](https://github.com/Kong/gateway-operator/pull/930)
- Added appropriate label selectors to `DataPlane`s with enabled Blue Green rollout
  strategy. Now Admin Service and `DataPlane` Deployments correctly select their
  Pods.
  Added `DataPlane`'s `status.selector` and `status.rollout.deployment.selector` fields.
  [#951](https://github.com/Kong/gateway-operator/pull/951)
- Added setting rollout status with `RolledOut` condition
  [#960](https://github.com/Kong/gateway-operator/pull/960)
- Added deploying of preview ingress service for Blue Green rollout strategy.
  [#956](https://github.com/Kong/gateway-operator/pull/956)
- Implemented an actual promotion of a preview deployment to live state when BlueGreen
  rollout strategy is used.
  [#966](https://github.com/Kong/gateway-operator/pull/966)
- Added `DataPlane` count in telemetry reports.
  [#941](https://github.com/Kong/gateway-operator/pull/941)
- Added `PromotionFailed` condition which is set on `DataPlane`s with Blue Green
  rollout strategy when promotion related activities (like updating `DataPlane`
  service selector) fail.
  [#1005](https://github.com/Kong/gateway-operator/pull/1005)
- Added `spec.deployment.rollout.strategy.blueGreen.resources.plan.deployment`
  which controls how operator manages `DataPlane` `Deployment`'s during and after
  a rollout. This can currently take 1 value:
  - `ScaleDownOnPromotionScaleUpOnRollout` which will scale down the `DataPlane`
  preview deployment to 0 replicas before a rollout is triggered via a spec change.
  [#1000](https://github.com/Kong/gateway-operator/pull/1000)
- Added admission webhook validation on of `DataPlane` spec updates when the
  Blue Green promotion is in progress.
  [#1051](https://github.com/Kong/gateway-operator/pull/1051)
- Added `gateway-operator.konghq.com/wait-for-owner` finalizer to all dependent
  resources owned by `DataPlane` to prevent them from being mistakenly deleted.
  [#1052](https://github.com/Kong/gateway-operator/pull/1052)

### Fixes

- Fixes setting `status.ready` and `status.conditions` on the `DataPlane` when
  it's waiting for an address to be assigned to its LoadBalancer Ingress Service.
  [#942](https://github.com/Kong/gateway-operator/pull/942)
- Correctly set the `observedGeneration` on `DataPlane` and `ControlPlane` status
  conditions.
  [#944](https://github.com/Kong/gateway-operator/pull/944)
- Added annotation `gateway-operator.konghq.com/last-applied-annotations` to
  resources (e.g, Ingress `Services`s) owned by `DataPlane`s to store last
   applied annotations to the owned resource. If an annotation is present in the
  `gateway-operator.konghq.com/last-applied-annotations` annotation of an
  ingress `Service` but not present in the current specification of ingress
  `Service` annotations of the owning `DataPlane`, the annotation will be removed
  in the ingress `Service`.
  [#936](https://github.com/Kong/gateway-operator/pull/936)
- Correctly set the `Ready` condition in `DataPlane` status field during Blue
  Green promotion. The `DataPlane` is considered ready whenever it has its
  Deployment's `AvailableReplicas` equal to desired number of replicas (as per
  `spec.replicas`) and its Service has an IP assigned if it's of type `LoadBalancer`.
  [#986](https://github.com/Kong/gateway-operator/pull/986)
- Properly handles missing CRD during controller startup. Now whenever a CRD
  is missing during startup a clean log entry will be printed to inform a user
  why the controller was disabled.
  Additionally a check for `discovery.ErrGroupDiscoveryFailed` was added during
  CRD lookup.
  [#1059](https://github.com/Kong/gateway-operator/pull/1059)

### Changes

- Default the leader election namespace to controller namespace (`POD_NAMESPACE` env)
  instead of hardcoded "kong-system"
  [#927](https://github.com/Kong/gateway-operator/pull/927)
- Renamed `DataPlane` proxy service name and label to ingress
  [#971](https://github.com/Kong/gateway-operator/pull/971)
- Removed `DataPlane` `status.ready` as it couldn't be used reliably to represent
  `DataPlane`'s status. Users should now use `status.conditions`'s `Ready` condition
  and compare its `observedGeneration` with `DataPlane` `metadata.generation`
  to get an accurate representation of `DataPlane`'s readiness.
  [#989](https://github.com/Kong/gateway-operator/pull/989)
- Disable `ControlPlane` and `Gateway` controllers by default.
  Users who want to enable those can use the command line flags:
  - `-enable-controller-controlplane` and
  - `-enable-controller-gateway`
  At this time, the Gateway API and `ControlPlane` resources that these
  flags are considered a feature preview, and are not supported. Use these
  only in non-production scenarios until these features are graduated to GA.
  [#1026](https://github.com/Kong/gateway-operator/pull/1026)
- Bump `ControlPlane` default version to `v2.11.1` and remove support for older versions.
  To satisfy this change, use `Programmed` condition instead of `Ready` in Gateway
  Listeners status conditions to make `ControlPlane` be able to attach routes
  to those listeners.
  This stems from the fact that KIC `v2.11` bumped support for Gateway API to `v0.7.1`.
  [#1041](https://github.com/Kong/gateway-operator/pull/1041)
- Bump Gateway API to v0.7.1.
  [#1047](https://github.com/Kong/gateway-operator/pull/1047)
- Operator doesn't change the `DataPlane` resource anymore by filling it with
  Kong Gateway environment variables. Instead this is now happening on the fly
  so the `DataPlane` resources applied by users stay as submitted.
  [#1034](https://github.com/Kong/gateway-operator/pull/1034)
- Don't use `Provisioned` status condition type on `DataPlane`s.
  From now on `DataPlane`s are only expressing their status through `Ready` status
  condtion.
  [#1043](https://github.com/Kong/gateway-operator/pull/1043)
- Bump default `DataPlane` image to 3.4
  [#1067](https://github.com/Kong/gateway-operator/pull/1067)
- When rollout strategy is removed from a `DataPlane` spec, preview subresources
  are removed.
  [#1066](https://github.com/Kong/gateway-operator/pull/1066)

## v0.6.0

> Release date: 2023-07-20

### Added

- Added `Ready`, `ReadyReplicas` and `Replicas` fields to `DataPlane`'s Status
  [#854](https://github.com/Kong/gateway-operator/pull/854)
- Added `Rollout` field to `DataPlane` CRD. This allows specification of rollout
  strategy and behavior (e.g. to enable blue/green rollouts for upgrades).
  [#879](https://github.com/Kong/gateway-operator/pull/879)
- Added `Rollout` status fields to `DataPlane` CRD.
  [#896](https://github.com/Kong/gateway-operator/pull/896)

### Changes

> **WARN**: Breaking changes included

- Renamed `Services` options in `DataPlaneOptions` to `Network` options, which
  now includes `IngressService` as one of the sub-attributes.
  This is a **breaking change** which requires some renaming and reworking of
  struct attribute access.
  [#849](https://github.com/Kong/gateway-operator/pull/849)
- Bump Gateway API to v0.6.2 and enable Gateway API conformance testing.
  [#853](https://github.com/Kong/gateway-operator/pull/853)
- Add `PodTemplateSpec` to `DeploymentOptions` to allow applying strategic merge
  patcher on top of `Pod`s generated by the operator.
  This is a **breaking change** which requires manual porting from `Pods` field
  to `PodTemplateSpec`.
  More info on strategic merge patch can be found in official Kubernetes docs at
  [sig-api-machinery/strategic-merge-patch.md][strategic-merge-patch].
  [#862](https://github.com/Kong/gateway-operator/pull/862)
- Added `v1beta1` version of the `DataPlane` API, which replaces the `v1alpha1`
  version. The `v1alpha1` version of the API has been removed entirely in favor
  of the new version to reduce maintenance costs.
  [#905](https://github.com/Kong/gateway-operator/pull/905)

[strategic-merge-patch]: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md

### Fixes

- Fixes setting `Affinity` when generating `Deployment`s for `DataPlane`s
  `ControlPlane`s which caused 2 `ReplicaSet`s to be created where the first
  one should already have the `Affinity` set making the update unnecessary.
  [#894](https://github.com/Kong/gateway-operator/pull/894)

## v0.5.0

> Release date: 2023-06-20

### Added

- Added `AddressSourceType` to `DataPlane` status `Address`
  [#798](https://github.com/Kong/gateway-operator/pull/798)
- Add pod Affinity field to `PodOptions` and support for both `DataPlane` and `ControlPlane`
- Add Kong Gateway enterprise image - `kong/kong-gateway` - to the set of supported
  `DataPlane` images.
  [#749](https://github.com/Kong/gateway-operator/pull/749)
- Moved pod related options in `DeploymentOptions` to `PodsOptions` and added pod
  labels option.
  [#742](https://github.com/Kong/gateway-operator/pull/742)
- Added `Volumes` and `VolumeMounts` field in `DeploymentOptions` of `DataPlane`
  specs. Users can attach custom volumes and mount the volumes to proxy container
  of pods in `Deployments` of dataplanes.
  Note: `Volumes` and `VolumeMounts` are not supported for `ControlPlane` specs now.
  [#681](https://github.com/Kong/gateway-operator/pull/681)
- Added possibility to replicas on `DataPlane` deployments
  This allows users to define `DataPlane`s - without `ControlPlane` - to be
  horizontally scalable.
  [#737](https://github.com/Kong/gateway-operator/pull/737)
- Added possibility to specify `DataPlane` proxy service type
  [#739](https://github.com/Kong/gateway-operator/pull/739)
- Added possibility to specify resources through `DataPlane` and `ControlPlane`
  `spec.deployment.resources`
  [#712](https://github.com/Kong/gateway-operator/pull/712)
- The `DataPlane` spec has been updated with a new field related
  to the proxy service. By using such a field, it is possible to
  specify annotations to be set on the `DataPlane` proxy service.
  [#682](https://github.com/Kong/gateway-operator/pull/682)

### Changed

- Bumped default ControlPlane image to 2.9.3
  [#712](https://github.com/Kong/gateway-operator/pull/712)
  [#719](https://github.com/Kong/gateway-operator/pull/719)
- Bumped default DataPlane image to 3.2.2
  [#728](https://github.com/Kong/gateway-operator/pull/728)
- Bumped Gateway API to 0.6.1. Along with it, the deprecated `Gateway`
  `scheduled` condition has been replaced by the `accepted` condition.
  [#618](https://github.com/Kong/gateway-operator/issues/618)
- `ControlPlane` and `DataPlane` specs have been refactored by explicitly setting
  the deployment field (instead of having it inline).
  [#725](https://github.com/Kong/gateway-operator/pull/725)
- `ControlPlane` and `DataPlane` specs now require users to provide `containerImage`
  and `version` fields.
  This is being enforced in the admission webhook.
  [#758](https://github.com/Kong/gateway-operator/pull/758)
- Validation for `ControlPlane` and `DataPlane` components no longer has a
  "ceiling", or maximum version. This due to popular demand, but now puts more
  emphasis on the user to troubleshoot when things go wrong. It's no longer
  possible to use a tag that's not semver compatible (e.g. 2.10.0) for these
  components (for instance, a branch such as `main`) without enabling developer
  mode.
  [#819](https://github.com/Kong/gateway-operator/pull/819)
- `ControlPlane` and `DataPlane` image validation now supports enterprise image
  flavours, e.g. `3.3.0-ubuntu`, `3.2.0.0-rhel` etc.
  [#830](https://github.com/Kong/gateway-operator/pull/830)

### Fixes

- Fix admission webhook certificates Job which caused TLS handshake errors when
  webhook was being called.
  [#716](https://github.com/Kong/gateway-operator/pull/716)
- Include leader election related role when generating `ControlPlane` RBAC
  manifests so that Gateway Discovery can be used by KIC.
  [#743](https://github.com/Kong/gateway-operator/pull/743)

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

[v1.0.0]: https://github.com/Kong/gateway-operator/tree/v1.0.0
[v0.7.0]: https://github.com/Kong/gateway-operator/tree/v0.7.0
[v0.6.0]: https://github.com/Kong/gateway-operator/tree/v0.6.0
[v0.5.0]: https://github.com/Kong/gateway-operator/tree/v0.5.0
[v0.4.0]: https://github.com/Kong/gateway-operator/tree/v0.4.0
[v0.3.0]: https://github.com/Kong/gateway-operator/tree/v0.3.0
[v0.2.0]: https://github.com/Kong/gateway-operator/tree/v0.2.0
[v0.1.1]: https://github.com/Kong/gateway-operator/tree/v0.1.1
[v0.1.0]: https://github.com/Kong/gateway-operator/tree/v0.1.0
