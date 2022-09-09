**WARNING**: This project is under active development and is currently considered **EXPERIMENTAL**.
             For information about timelines for initial releases please see our [milestones][mst].
             Please check in with us on the [#kong][ch] on Kubernetes Slack or open a
             [discussion][discuss] if you have questions, want to contribute, or just want to chat
             about the project.

[mst]:https://github.com/Kong/gateway-operator/milestones
[ch]:https://kubernetes.slack.com/messages/kong
[discuss]:https://github.com/kong/gateway-operator/discussions

# Kong Gateway Operator

A [Kubernetes Operator][k8soperator] for the [Kong Gateway][kong].

[k8soperator]:https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
[kong]:https://konghq.com
[helmop]:https://github.com/kong/kong-operator

### Deployment

Prior to deployment, Kong and [Gateway API][gwapi] CRDs need to be deployed:

```console
kubectl kustomize https://github.com/Kong/kubernetes-ingress-controller/config/crd | kubectl apply -f -
kubectl kustomize https://github.com/kubernetes-sigs/gateway-api/config/crd | kubectl apply -f -
```

Deploy the operator with the following one-liner:

```console
kubectl kustomize https://github.com/kong/gateway-operator/config/default?submodules=false | kubectl apply -f -
```

Optionally, you can wait for the operator with:

```console
kubectl -n kong-system wait --for=condition=Available=true --timeout=120s deployment/gateway-operator-controller-manager
```

[gwapi]:https://github.com/kubernetes-sigs/gateway-api

### Usage

After deployment usage is driven primarily via the [Gateway][gwapi] resource.
You can deploy a `Gateway` resource to the cluster which will result in the
underlying control-plane (the [Kong Kubernetes Ingress Controller][kic]) and
the data-plane (the [Kong Gateway][kong-ce]).

For example, deploy the following `GatewayClass`:

```yaml
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: kong
spec:
  controllerName: konghq.com/gateway-operator
```

and `Gateway`:

```yaml
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: kong
spec:
  gatewayClassName: kong
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```

Wait for the `Gateway` to be `Ready`:

```console
kubectl wait --for=condition=Ready=true gateways.gateway.networking.k8s.io/kong
```

Once `Ready` you'll be able to receive the default IP address of the `Gateway`:

```console
$ kubectl get gateway kong
NAME   CLASS   ADDRESS        READY   AGE
kong   kong    172.18.0.240   True    97s
```

The `Gateway` is now accessible via that IP:

```console
$ curl -s -w '\n' http://172.18.0.240
{"message":"no Route matched with those values"}
```

(NOTE: if your cluster can not provision `LoadBalancer` type `Services` then
the IP you receive may only be routable from within the cluster).

(NOTE: the `no Route matched` result is normal for a `Gateway` with no
configuration. Create `Ingress`, `HTTPRoute` and other resources to start
routing traffic to your applications. See the [Ingress Controller
Guides][kic-guides] for more information).

[gwapi]:https://github.com/kubernetes-sigs/gateway-api
[kic]:https://github.com/kong/kubernetes-ingress-controller
[kong-ce]:https://github.com/kong/kong
[kic-guides]:https://docs.konghq.com/kubernetes-ingress-controller/latest/guides/overview/

#### Configuring Gateways

A `Gateway` resource has subcomponents such as a `ControlPlane` and a
`DataPlane` which are created and managed on its behalf. At a deeper technical
level, `ControlPlane` corresponds with the [Kong Kubernetes Ingress
Controller (KIC)][kic] and `DataPlane` corresponds with the [Kong Gateway][gw].

While not required for basic usage, it is possible to provide configuration for
these subcomponents using the `GatewayConfiguration` API. That configuration
can include the container image and image version to use for the subcomponents,
as well as environment and volume mount overrides that will be passed down to
`Pods` created for that component. For example:

```yaml
kind: GatewayConfiguration
apiVersion: gateway-operator.konghq.com/v1alpha1
metadata:
  name: kong
  namespace: default
spec:
  dataPlaneDeploymentOptions:
    containerImage: kong/kubernetes-ingress-controller
    version: 2.4.2
    env:
    - name: TEST_VAR
      value: TEST_VAL
  controlPlaneDeploymentOptions:
    containerImage: kong/kong
    version: 2.8.0
    env:
    - name: TEST_VAR
      value: TEST_VAL
```

Configurations like the above can be created on the API but wont be active
until referenced by a `GatewayClass`:

```yaml
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: kong
spec:
  controllerName: konghq.com/gateway-operator
  parametersRef:
    group: gateway-operator.konghq.com
    kind: GatewayConfiguration
    name: kong
    namespace: default
```

With the `parametersRef` in the above `GatewayClass` being used to attach the
`GatewayConfiguration`, that configuration will start applying to all `Gateway`
resources created for that class, and will retroactively apply to any `Gateway`
resources previously created.

[kic]:https://github.com/kong/kubernetes-ingress-controller
[gw]:https://github.com/kong/kong

#### Gateway upgrades/downgrades

The `GatewayConfiguration` API can be used to provide the image and the image
version desired for either the `ControlPlane` or `DataPlane` component of the
`Gateway` e.g.:

```yaml
kind: GatewayConfiguration
apiVersion: gateway-operator.konghq.com/v1alpha1
metadata:
  name: kong
  namespace: default
spec:
  dataPlaneDeploymentOptions:
    containerImage: kong/kong
    version: 2.7.0
  controlPlaneDeploymentOptions:
    containerImage: kong/kubernetes-ingress-controller
    version: 2.4.2
```

The above configuration will deploy all `DataPlane` resources connected to the
`GatewayConfiguration` (by way of `GatewayClass`) using `kong/kong:2.7.0` and
any `ControlPlane` will be deployed with
`kong/kubernetes-ingress-controller:2.4.2`.

Given the above a manual upgrade or downgrade can be performed simply by
changing the version. For example: assuming that at least one `Gateway` is
currently deployed and running using the above `GatewayConfiguration`, an
upgrade could be performed by running the following:

```console
kubectl edit gatewayconfiguration kong
```

And updating the `dataPlaneDeploymentOptions.version` to `2.8.0`. The result
will be a replacement `Pod` will roll out with the new version and once healthy
the old `Pod` will be terminated.

### Usage with Kong enterprise as dataplane

You can use Kong enterprise as the dataplane by doing as follows:

1. Create a secret with the Kong license in the namespace that you intend to use for deploying the gateway.

    ```console
    kubectl create secret generic kong-enterprise-license --from-file=license=<license-file> -n <your-namespace>
    ```

2. Create a `GatewayConfiguration` specifying the enterprise container image and the environment variable referencing the license secret. The operator will use the image and the environment variable specified in the `GatewayConfiguration` to customize the dataplane. As the result, the dataplane will use `kong/kong-gateway:2.8` as the image and mount the license secret.

    ```yaml
    kind: GatewayConfiguration
    apiVersion: gateway-operator.konghq.com/v1alpha1
    metadata:
      name: kong
      namespace: <your-namespace>
    spec:
      dataPlaneDeploymentOptions:
        containerImage: kong/kong-gateway:2.8
        env:
        - name: KONG_LICENSE_DATA
          valueFrom:
            secretKeyRef:
              key: license
              name: kong-enterprise-license
    ```

3. Create a `GatewayClass` that references the `GatewayConfiguration` above.

    ```yaml
    kind: GatewayClass
    apiVersion: gateway.networking.k8s.io/v1beta1
    metadata:
      name: kong
    spec:
      controllerName: konghq.com/gateway-operator
      parametersRef:
        group: gateway-operator.konghq.com
        kind: GatewayConfiguration
        name: kong
        namespace: <your-namespace>
    ```

4. And finally create a Gateway that uses the `GatewayClass` above:

    ```yaml
    kind: Gateway
    apiVersion: gateway.networking.k8s.io/v1beta1
    metadata:
      name: kong
      namespace: <your-namespace>
    spec:
      gatewayClassName: kong
      listeners:
      - name: http
        protocol: HTTP
        port: 80
    ```

5. Wait for the `Gateway` to be `Ready`:

    ```console
    kubectl wait --for=condition=Ready=true gateways.gateway.networking.k8s.io/kong
    ```

6. Check that the dataplane is using the enterprise image:

    ```console
    $ kubectl get deployment -l konghq.com/gateway-operator=dataplane -o jsonpath='{.items[0].spec.template.spec.containers[0].image}'

    kong/kong-gateway:2.8
    ```

7. A log message should describe the status of the provided license. Check it through:

    ```console
    $ kubectl logs $(kubectl get po -l app=$(kubectl get dataplane -o=jsonpath='{.items[0].metadata.name}') -o=jsonpath="{.items[0].metadata.name}") | grep license_helpers.lua

    2022/08/29 10:50:55 [error] 2111#0: *8 [lua] license_helpers.lua:194: log_license_state(): The Kong Enterprise license will expire on 2022-09-20. Please contact <support@konghq.com> to renew your license., context: ngx.timer
    ```

> **Note**: the license secret, the `GatewayConfiguration`, and the `Gateway` MUST be created in the same namespace.

### Seeking Help

Please search through the posts on the [discussions page][disc] as it's likely
that another user has run into the same problem. If you don't find an answer,
please feel free to post a question.

If you've found a bug, please [open an issue][issues].

For a feature request, please open an issue using the feature request template.

You can also talk to the developers behind Kong in the [#kong][slack] channel on
the Kubernetes Slack server.

[disc]:https://github.com/kong/gateway-operator/discussions
[issues]:https://github.com/kong/kubernetes-ingress-controller/issues
[slack]:https://kubernetes.slack.com/messages/kong

### Community Meetings

You can join monthly meetups hosted by [Kong][kong] to ask questions, provide
feedback, or just to listen and hang out.

See the [Online Meetups Page][kong-meet] to sign up and receive meeting invites
and [Zoom][zoom] links.

[kong]:https://konghq.com
[kong-meet]:https://konghq.com/online-meetups/
[zoom]:https://zoom.us
