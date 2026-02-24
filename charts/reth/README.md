# Reth Helm Chart

Deploy and scale [Reth](https://github.com/paradigmxyz/reth) inside Kubernetes with ease

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) ![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.1.3](https://img.shields.io/badge/AppVersion-v1.1.3-informational?style=flat-square)

## Chart Features

- Actively maintained by [GraphOps](https://graphops.xyz) and contributors
- Strong security defaults (non-root execution, read-only root filesystem, drops all capabilities)
- Readiness checks to ensure traffic only hits `Pod`s that are healthy and ready to serve requests
- Support for `ServiceMonitor`s to configure Prometheus to scrape metrics ([prometheus-operator](https://github.com/prometheus-operator/prometheus-operator))
- Support for configuring Grafana dashboards ([grafana](https://github.com/grafana/helm-charts/tree/main/charts/grafana))
- Support for exposing a NodePort to enable inbound P2P dials for better peering

## Quickstart

To install the chart with the release name `my-release`:

```console
$ helm repo add graphops http://graphops.github.io/launchpad-charts
$ helm install my-release graphops/reth
```

## Specifying the Engine API JWT

To use Reth on a network that requires a Consensus Client, you will need to configure a JWT that is used by the Consensus Client to authenticate with the Engine API on port `8551`. You will need to pass the same JWT to your Consensus Client.

You can specify the JWT for Reth either as a literal value, or as a reference to a key in an existing Kubernetes Secret. If you specify a literal value, it will be wrapped into a new Kubernetes Secret and passed into the Reth Pod.

Using a literal value:

```yaml
# values.yaml

reth:
  jwt:
    fromLiteral: some-secure-random-value-that-you-generate # You can generate this with: openssl rand -hex 32
```

Using an existing Kubernetes Secret:

```yaml
# values.yaml

reth:
  jwt:
    existingSecret:
      name: my-ethereum-mainnet-jwt-secret
      key: jwt
```

## Configuring Reth

Reth can be configured via CLI arguments and via a TOML configuration file. This chart supports both methods.

### CLI Arguments

You can pass additional CLI arguments using the `reth.extraArgs` value:

```yaml
# values.yaml

reth:
  extraArgs:
    - "--log.file.filter=debug"
    - "--log.file.directory=/data/logs"
```

### TOML Configuration

You can provide a custom `reth.toml` configuration file using the `reth.config` value. See [Reth Configuration Documentation](https://reth.rs/run/configuration/) for all available options:

```yaml
# values.yaml

reth:
  config: |
    [peers]
    max_outbound = 100
    max_inbound = 30
   
    [stages.headers]
    downloader_max_concurrent_requests = 100
```

## Upgrading

We recommend that you pin the version of the Chart that you deploy. You can use the `--version` flag with `helm install` and `helm upgrade` to specify a chart version constraint.

This project uses [Semantic Versioning](https://semver.org/). Changes to the version of the application (the `appVersion`) that the Chart deploys will generally result in a patch version bump for the Chart. Breaking changes to the Chart or its `values.yaml` interface will be reflected with a major version bump.

We do not recommend that you upgrade the application by overriding `image.tag`. Instead, use the version of the Chart that is built for your desired `appVersion`.

## Values

| Key | Description | Type | Default |
|-----|-------------|------|---------|
 | fullnameOverride |  | string | `""` |
 | grafana.dashboards | Enable creation of Grafana dashboards. [Grafana chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana#grafana-helm-chart) must be configured to search this namespace, see `sidecar.dashboards.searchNamespace` | bool | `false` |
 | grafana.dashboardsConfigMapLabel | Must match `sidecar.dashboards.label` value for the [Grafana chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana#grafana-helm-chart) | string | `"grafana_dashboard"` |
 | grafana.dashboardsConfigMapLabelValue | Must match `sidecar.dashboards.labelValue` value for the [Grafana chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana#grafana-helm-chart) | string | `"1"` |
 | grafana.operatorDashboards | Create GrafanaDashboard CRDs via Grafana Operator from files in `dashboards/` | object | `{"allowCrossNamespaceImport":false,"annotations":{},"enabled":false,"extraSpec":{},"folder":"","folderUID":"","instanceSelector":{"matchLabels":{}},"labels":{},"namespace":"","resyncPeriod":"","suspend":false,"uid":""}` |
 | grafana.operatorDashboards.allowCrossNamespaceImport | Allow matching Grafana instances outside current namespace | bool | `false` |
 | grafana.operatorDashboards.extraSpec | Additional spec fields to merge into GrafanaDashboard.spec | object | `{}` |
 | grafana.operatorDashboards.folder | Optional folder metadata | string | `""` |
 | grafana.operatorDashboards.instanceSelector | Selector to match Grafana instances managed by the operator | object | `{"matchLabels":{}}` |
 | grafana.operatorDashboards.labels | Extra labels and annotations on the GrafanaDashboard resources | object | `{}` |
 | grafana.operatorDashboards.namespace | Optional target namespace for the GrafanaDashboard CRDs (defaults to release namespace) | string | `""` |
 | grafana.operatorDashboards.resyncPeriod | Operator sync behavior | string | `""` |
 | image.pullPolicy |  | string | `"IfNotPresent"` |
 | image.repository | Image for Reth | string | `"ghcr.io/paradigmxyz/reth"` |
 | image.tag | Overrides the image tag | string | Chart.appVersion |
 | imagePullSecrets | Pull secrets required to fetch the Image | list | `[]` |
 | nameOverride |  | string | `""` |
 | prometheus.serviceMonitors.enabled | Enable monitoring by creating `ServiceMonitor` CRDs ([prometheus-operator](https://github.com/prometheus-operator/prometheus-operator)) | bool | `false` |
 | prometheus.serviceMonitors.interval |  | string | `nil` |
 | prometheus.serviceMonitors.labels |  | object | `{}` |
 | prometheus.serviceMonitors.relabelings |  | list | `[]` |
 | prometheus.serviceMonitors.scrapeTimeout |  | string | `nil` |
 | rbac.clusterRules | Required ClusterRole rules | list | See `values.yaml` |
 | rbac.create | Specifies whether RBAC resources are to be created | bool | `true` |
 | rbac.rules | Required ClusterRole rules | list | See `values.yaml` |
 | reth.affinity |  | object | `{}` |
 | reth.affinityPresets.antiAffinityByHostname | Configure anti-affinity rules to prevent multiple Reth instances on the same host | bool | `true` |
 | reth.authrpc | Engine API configuration (for consensus client) | object | `{"addr":"0.0.0.0","jwtsecret":"/data/jwt.hex","port":8551}` |
 | reth.authrpc.addr | Engine API listening address | string | `"0.0.0.0"` |
 | reth.authrpc.jwtsecret | JWT secret path | string | `"/data/jwt.hex"` |
 | reth.authrpc.port | Engine API listening port | int | `8551` |
 | reth.chain | Chain to sync (mainnet, sepolia, goerli, holesky, etc.) | string | `"mainnet"` |
 | reth.config | Custom reth.toml configuration Will be mounted as a ConfigMap at /config/reth.toml See https://reth.rs/run/configuration/ for all options | string | `""` |
 | reth.datadir | The path to the Reth data directory | string | `"/data"` |
 | reth.extraArgs | Additional CLI arguments to pass to `reth node` | list | `[]` |
 | reth.extraContainers | Additional containers | list | `[]` |
 | reth.extraInitContainers | Additional init containers | list | `[]` |
 | reth.extraLabels | Extra labels to attach to the Pod for matching against | object | `{}` |
 | reth.full | Enable full node (archive mode if false) | bool | `true` |
 | reth.jwt | JWT for clients to authenticate with the Engine API. Specify either `existingSecret` OR `fromLiteral`. | object | `{"existingSecret":{"key":null,"name":null},"fromLiteral":null}` |
 | reth.jwt.existingSecret | Load the JWT from an existing Kubernetes Secret. Takes precedence over `fromLiteral` if set. | object | `{"key":null,"name":null}` |
 | reth.jwt.existingSecret.key | Data key for the JWT in the Secret | string | `nil` |
 | reth.jwt.existingSecret.name | Name of the Secret resource in the same namespace | string | `nil` |
 | reth.jwt.fromLiteral | Use this literal value for the JWT | string | `nil` |
 | reth.metrics | Metrics configuration | object | `{"addr":"0.0.0.0","enabled":true,"port":9001}` |
 | reth.metrics.addr | Metrics listening address | string | `"0.0.0.0"` |
 | reth.metrics.enabled | Enable metrics | bool | `true` |
 | reth.metrics.port | Metrics listening port | int | `9001` |
 | reth.nodeSelector |  | object | `{}` |
 | reth.p2p | P2P networking configuration | object | `{"maxInboundPeers":30,"maxOutboundPeers":100,"port":30303}` |
 | reth.p2p.maxInboundPeers | Maximum number of inbound peers | int | `30` |
 | reth.p2p.maxOutboundPeers | Maximum number of outbound peers | int | `100` |
 | reth.p2p.port | P2P listening port | int | `30303` |
 | reth.p2pNodePort.enabled | Expose P2P port via NodePort | bool | `false` |
 | reth.p2pNodePort.initContainer.image.pullPolicy | Container pull policy | string | `"IfNotPresent"` |
 | reth.p2pNodePort.initContainer.image.repository | Container image to fetch nodeport information | string | `"lachlanevenson/k8s-kubectl"` |
 | reth.p2pNodePort.initContainer.image.tag | Container tag | string | `"v1.25.4"` |
 | reth.p2pNodePort.port | NodePort to be used. Must be unique. | int | `31000` |
 | reth.podAnnotations | Annotations for the `Pod` | object | `{}` |
 | reth.podSecurityContext | Pod-wide security context | object | `{"fsGroup":1000,"runAsGroup":1000,"runAsNonRoot":true,"runAsUser":1000}` |
 | reth.resources |  | object | `{}` |
 | reth.rpc | RPC configuration | object | `{"addr":"0.0.0.0","api":"eth,net,web3","corsdomain":"*","enabled":true,"port":8545}` |
 | reth.rpc.addr | HTTP RPC server listening address | string | `"0.0.0.0"` |
 | reth.rpc.api | RPC API modules to enable (eth, net, web3, debug, trace, etc.) | string | `"eth,net,web3"` |
 | reth.rpc.corsdomain | CORS origins to allow | string | `"*"` |
 | reth.rpc.enabled | Enable HTTP RPC server | bool | `true` |
 | reth.rpc.port | HTTP RPC server listening port | int | `8545` |
 | reth.service.ports.http-engineapi | Service Port to expose Engine API interface on | int | `8551` |
 | reth.service.ports.http-jsonrpc | Service Port to expose JSON-RPC interface on | int | `8545` |
 | reth.service.ports.http-metrics | Service Port to expose Prometheus metrics on | int | `9001` |
 | reth.service.ports.ws | Service Port to expose WebSocket interface on | int | `8546` |
 | reth.service.topologyAwareRouting.enabled | Toggle for topology aware routing | bool | `false` |
 | reth.service.type |  | string | `"ClusterIP"` |
 | reth.terminationGracePeriodSeconds | Amount of time to wait before force-killing the Reth process | int | `300` |
 | reth.tolerations |  | list | `[]` |
 | reth.volumeClaimSpec | [PersistentVolumeClaimSpec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#persistentvolumeclaimspec-v1-core) for Reth storage | object | `{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"2Ti"}},"storageClassName":null}` |
 | reth.volumeClaimSpec.resources.requests.storage | The amount of disk space to provision for Reth | string | `"2Ti"` |
 | reth.volumeClaimSpec.storageClassName | The storage class to use when provisioning a persistent volume for Reth | string | `nil` |
 | reth.ws | WebSocket configuration | object | `{"addr":"0.0.0.0","api":"eth,net,web3","enabled":true,"origins":"*","port":8546}` |
 | reth.ws.addr | WebSocket server listening address | string | `"0.0.0.0"` |
 | reth.ws.api | WebSocket API modules to enable | string | `"eth,net,web3"` |
 | reth.ws.enabled | Enable WebSocket server | bool | `true` |
 | reth.ws.origins | WebSocket origins to allow | string | `"*"` |
 | reth.ws.port | WebSocket server listening port | int | `8546` |
 | serviceAccount.annotations | Annotations to add to the service account | object | `{}` |
 | serviceAccount.create | Specifies whether a service account should be created | bool | `true` |
 | serviceAccount.name | The name of the service account to use. If not set and create is true, a name is generated using the fullname template | string | `""` |

## Contributing

We welcome and appreciate your contributions! Please see the [Contributor Guide](/CONTRIBUTING.md), [Code Of Conduct](/CODE_OF_CONDUCT.md) and [Security Notes](/SECURITY.md) for this repository.
