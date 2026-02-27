# vLLM Production Stack helm chart

This helm chart lets users deploy multiple serving engines and a router into the Kubernetes cluster.

## Key features

- Support running multiple serving engines with multiple different models
- Load the model weights directly from the existing PersistentVolumes

## Prerequisites

1. A running Kubernetes cluster with GPU. (You can set it up through `minikube`: <https://minikube.sigs.k8s.io/docs/tutorials/nvidia/>)
2. [Helm](https://helm.sh/docs/intro/install/)

## Install the helm chart

```bash
helm install llmstack . -f values-example.yaml
```

## Uninstall the deployment

run `helm uninstall llmstack`

## Configure the deployments

See `helm/values.yaml` for mode details.

## Values

### Other Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| extraObjects | list | `[]` | Array of extra K8s manifests to deploy. Supports use of custom Helm templates |

### LoRA Controller Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| loraController | object | `{"affinity":{},"containerSecurityContext":{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]}},"enableLoraController":false,"env":[],"extraArgs":[],"image":{"pullPolicy":"IfNotPresent","repository":"lmcache/lmstack-lora-controller","tag":"latest"},"imagePullSecrets":[],"kubernetesClusterDomain":"cluster.local","metrics":{"enabled":true},"nodeSelector":{},"podAnnotations":{},"podSecurityContext":{"runAsNonRoot":true,"seccompProfile":{"type":"RuntimeDefault"}},"replicaCount":1,"resources":{},"tolerations":[],"webhook":{"enabled":false}}` | lora controller Configuration |

### Router OpenTelemetry Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| routerSpec.otel | object | `{"endpoint":"","secure":false,"serviceName":"vllm-router"}` | OpenTelemetry tracing configuration When otelEndpoint is set, tracing is automatically enabled |

### Other Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| loraAdapters | list | `[]` | List of LoRA adapter instances to deploy. Required fields per entry: `name`, `baseModel`, `adapterSource.type` (local/s3/http/huggingface), `adapterSource.adapterName`. Optional fields: `vllmApiKey` (string value or secretRef with secretName/secretKey), `adapterSource.adapterPath`, `adapterSource.repository`, `adapterSource.pattern`, `adapterSource.maxAdapters`, `adapterSource.credentials` (secretName, secretKey), `loraAdapterDeploymentConfig.algorithm` (default/ordered/equalized), `loraAdapterDeploymentConfig.replicas`, `labels`. See the [LoRA Adapters Reference](#lora-adapters-reference) section for a complete example. |
| loraController.affinity | object | `{}` | Affinity |
| loraController.containerSecurityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]}}` | Container security context |
| loraController.env | list | `[]` | Environment variables |
| loraController.extraArgs | list | `[]` | Extra arguments for the lora controller |
| loraController.image | object | `{"pullPolicy":"IfNotPresent","repository":"lmcache/lmstack-lora-controller","tag":"latest"}` | lora controller image configuration |
| loraController.imagePullSecrets | list | `[]` | Image pull secrets |
| loraController.kubernetesClusterDomain | string | `"cluster.local"` | kubernetes cluster domain |
| loraController.metrics | object | `{"enabled":true}` | Metrics configuration |
| loraController.nodeSelector | object | `{}` | Node selector |
| loraController.podAnnotations | object | `{}` | Pod annotations |
| loraController.podSecurityContext | object | `{"runAsNonRoot":true,"seccompProfile":{"type":"RuntimeDefault"}}` | Pod security context |
| loraController.replicaCount | int | `1` | Number of lora controller replicas |
| loraController.resources | object | `{}` | lora controller resources |
| loraController.tolerations | list | `[]` | Tolerations |
| loraController.webhook | object | `{"enabled":false}` | Webhook configuration |
| routerSpec.affinity | object | `{}` |  |
| routerSpec.autoscaling | object | `{"enabled":false,"maxReplicas":3,"minReplicas":1,"targetCPUUtilizationPercentage":80}` | autoscaling configuration |
| routerSpec.containerPort | int | `8000` | Container port |
| routerSpec.containerSecurityContext | object | `{}` | Container-level security context configuration. https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#securitycontext-v1-core |
| routerSpec.enableRouter | bool | `true` | Whether to enable the router service |
| routerSpec.engineScrapeInterval | int | `15` | Interval in seconds to scrape the serving engine metrics |
| routerSpec.extraArgs | list | `[]` | extra router commandline arguments |
| routerSpec.extraPorts | list | `[]` | extra service ports |
| routerSpec.imagePullPolicy | string | `"Always"` |  |
| routerSpec.imagePullSecrets | list | `[]` | Image pull secrets for private container registries Example: imagePullSecrets:   - name: my-registry-secret |
| routerSpec.ingress.annotations | object | `{}` | Additional annotations for the Ingress resource |
| routerSpec.ingress.className | string | `""` | IngressClass that will be used to implement the Ingress |
| routerSpec.ingress.enabled | bool | `false` | Enable ingress controller resource |
| routerSpec.ingress.hosts[0].host | string | `"vllm-router.local"` |  |
| routerSpec.ingress.hosts[0].paths[0].path | string | `"/"` |  |
| routerSpec.ingress.hosts[0].paths[0].pathType | string | `"Prefix"` |  |
| routerSpec.ingress.tls | list | `[]` | The tls configuration for hostnames to be covered with this ingress record. |
| routerSpec.k8sServiceDiscoveryType | string | `"pod-ip"` | Service discovery mode type, supports "pod-ip" or "service-name". Defaults to "pod-ip" if not set. |
| routerSpec.labels | object | `{"environment":"router","release":"router"}` | Customized labels for the router deployment |
| routerSpec.livenessProbe.failureThreshold | int | `3` |  |
| routerSpec.livenessProbe.httpGet.path | string | `"/health"` |  |
| routerSpec.livenessProbe.initialDelaySeconds | int | `30` |  |
| routerSpec.livenessProbe.periodSeconds | int | `5` |  |
| routerSpec.nodeSelectorTerms | list | `[]` |  |
| routerSpec.otel.endpoint | string | `""` | OTLP endpoint for tracing (e.g., "localhost:4317" or "otel-collector:4317") |
| routerSpec.otel.secure | bool | `false` | Use secure (TLS) connection for OTLP exporter (default: false, i.e., insecure) |
| routerSpec.otel.serviceName | string | `"vllm-router"` | Service name for traces (default: "vllm-router") |
| routerSpec.podAnnotations | object | `{}` | Customized pod annotations for the router pods |
| routerSpec.priorityClassName | string | `""` | Priority Class |
| routerSpec.readinessProbe.failureThreshold | int | `3` |  |
| routerSpec.readinessProbe.httpGet.path | string | `"/health"` |  |
| routerSpec.readinessProbe.initialDelaySeconds | int | `30` |  |
| routerSpec.readinessProbe.periodSeconds | int | `5` |  |
| routerSpec.replicaCount | int | `1` | Number of replicas |
| routerSpec.repository | string | `"lmcache/lmstack-router"` | The docker image of the router. The following values are defaults: |
| routerSpec.requestStatsWindow | int | `60` | Window size in seconds to calculate the request statistics |
| routerSpec.resources | object | `{"limits":{"memory":"1000Mi"},"requests":{"cpu":"400m","memory":"1000Mi"}}` | router resource requests and limits |
| routerSpec.rootPath | string | `""` | FastAPI root path for hosting under a subpath (e.g. /vllm) |
| routerSpec.route | object | `{"main":{"additionalRules":[],"annotations":{},"apiVersion":"gateway.networking.k8s.io/v1","enabled":false,"filters":[],"hostnames":[],"httpsRedirect":false,"kind":"HTTPRoute","labels":{},"matches":[{"path":{"type":"PathPrefix","value":"/"}}],"parentRefs":[]}}` | Expose the service via gateway-api HTTPRoute More routes can be added by adding a dictionary key like the 'main' route. Requires Gateway API resources and suitable controller installed within the cluster (see: https://gateway-api.sigs.k8s.io/guides/) |
| routerSpec.route.main.apiVersion | string | `"gateway.networking.k8s.io/v1"` | Set the route apiVersion, e.g. gateway.networking.k8s.io/v1 or gateway.networking.k8s.io/v1alpha2 |
| routerSpec.route.main.enabled | bool | `false` | Enables or disables the route |
| routerSpec.route.main.httpsRedirect | bool | `false` | create http route for redirect (https://gateway-api.sigs.k8s.io/guides/http-redirect-rewrite/#http-to-https-redirects) # Take care that you only enable this on the http listener of the gateway to avoid an infinite redirect. # matches, filters and additionalRules will be ignored if this is set to true. |
| routerSpec.route.main.kind | string | `"HTTPRoute"` | Set the route kind Valid options are GRPCRoute, HTTPRoute, TCPRoute, TLSRoute, UDPRoute |
| routerSpec.routingLogic | string | `"roundrobin"` | routing logic, could be "roundrobin" or "session" |
| routerSpec.securityContext | object | `{}` | Pod-level security context configuration. https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#podsecuritycontext-v1-core |
| routerSpec.serviceAnnotations | object | `{}` | Service annotations if you use a LoadBalancer or NodePort service type |
| routerSpec.serviceDiscovery | string | `"k8s"` | Service discovery mode, supports "k8s" or "static". Defaults to "k8s" if not set. |
| routerSpec.servicePort | int | `80` | Service port |
| routerSpec.serviceType | string | `"ClusterIP"` | Service type |
| routerSpec.sessionKey | string | `""` | session key if using "session" routing logic |
| routerSpec.startupProbe.failureThreshold | int | `3` |  |
| routerSpec.startupProbe.httpGet.path | string | `"/health"` |  |
| routerSpec.startupProbe.initialDelaySeconds | int | `5` |  |
| routerSpec.startupProbe.periodSeconds | int | `5` |  |
| routerSpec.staticBackends | string | `""` | If serviceDiscovery is set to "static", the comma-separated values below are required. There needs to be the same number of backends and models |
| routerSpec.staticModels | string | `""` |  |
| routerSpec.strategy | object | `{}` | deployment strategy |
| routerSpec.tag | string | `"latest"` |  |
| servingEngineSpec.configs | object | `{}` | Set other environment variables from config map |
| servingEngineSpec.containerPort | int | `8000` | Container port |
| servingEngineSpec.containerSecurityContext | object | `{"runAsNonRoot":false}` | Container-level security context configuration. https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#securitycontext-v1-core |
| servingEngineSpec.containerSecurityContext.runAsNonRoot | bool | `false` | Run as non-root |
| servingEngineSpec.enableEngine | bool | `true` | Enable the serving engine deployment. |
| servingEngineSpec.extraPorts | list | `[]` | Extra service ports for models |
| servingEngineSpec.labels | object | `{"environment":"test","release":"test"}` | Additional deployment labels for the serving engine deployment |
| servingEngineSpec.livenessProbe | object | `{"failureThreshold":3,"httpGet":{"path":"/health","port":8000},"initialDelaySeconds":15,"periodSeconds":10}` | Liveness probe configuration |
| servingEngineSpec.livenessProbe.failureThreshold | int | `3` | Number of times after which if a probe fails in a row, Kubernetes considers that the overall check has failed: the container is not alive |
| servingEngineSpec.livenessProbe.httpGet | object | `{"path":"/health","port":8000}` | Configuration of the Kubelet http request on the server |
| servingEngineSpec.livenessProbe.httpGet.path | string | `"/health"` | Path to access on the HTTP server |
| servingEngineSpec.livenessProbe.httpGet.port | int | `8000` | Name or number of the port to access on the container, on which the server is listening |
| servingEngineSpec.livenessProbe.initialDelaySeconds | int | `15` | Number of seconds after the container has started before liveness probe is initiated |
| servingEngineSpec.livenessProbe.periodSeconds | int | `10` | How often (in seconds) to perform the liveness probe |
| servingEngineSpec.maxUnavailablePodDisruptionBudget | string | `""` | Disruption Budget Configuration |
| servingEngineSpec.modelSpec | list | `[]` | List of model serving engine deployment configurations. Each entry configures a separate vLLM model deployment. Required fields: `name`, `repository`, `tag`, `modelURL`. Optional fields: `replicaCount`, `requestCPU`, `requestMemory`, `requestGPU` (and HAMi variants: `requestGPUType`, `requestGPUMem`, `requestGPUMemPercentage`, `requestGPUCores`), `limitCPU`, `limitMemory`, `imagePullPolicy`, `imagePullSecret`, `annotations`, `podAnnotations`, `serviceAccountName`, `priorityClassName`, `runtimeClassName`, `vllmConfig` (v0, enablePrefixCaching, enableChunkedPrefill, maxModelLen, dtype, tensorParallelSize, maxNumSeqs, maxLoras, gpuMemoryUtilization, runner, convert, extraArgs), `lmcacheConfig` (enabled, cpuOffloadingBufferSize, logLevel), `hf_token` (string or secretRef), `envFromSecret`, `env`, `pvcStorage`, `pvcAccessMode`, `storageClass`, `pvcMatchLabels`, `pvcLabels`, `pvcAnnotations`, `extraVolumes`, `extraVolumeMounts`, `initContainer`, `enableLoRA`, `shmSize`, `chatTemplate`, `affinity`, `nodeSelectorTerms`, `nodeName`, `keda` (enabled, minReplicaCount, maxReplicaCount, pollingInterval, cooldownPeriod, idleReplicaCount, initialCooldownPeriod, triggers, advanced, fallback). See the [Model Spec Reference](#model-spec-reference) section for a complete example. |
| servingEngineSpec.readinessProbe | object | `{"failureThreshold":3,"httpGet":{"path":"/health","port":8000},"initialDelaySeconds":15,"periodSeconds":5}` | Readiness probe configuration |
| servingEngineSpec.readinessProbe.failureThreshold | int | `3` | Number of times after which if a probe fails in a row, Kubernetes considers that the overall check has failed: the container is not alive |
| servingEngineSpec.readinessProbe.httpGet | object | `{"path":"/health","port":8000}` | Configuration of the Kubelet http request on the server |
| servingEngineSpec.readinessProbe.httpGet.path | string | `"/health"` | Path to access on the HTTP server |
| servingEngineSpec.readinessProbe.httpGet.port | int | `8000` | Name or number of the port to access on the container, on which the server is listening |
| servingEngineSpec.readinessProbe.initialDelaySeconds | int | `15` | Number of seconds after the container has started before readiness probe is initiated |
| servingEngineSpec.readinessProbe.periodSeconds | int | `5` | How often (in seconds) to perform the readiness probe |
| servingEngineSpec.runtimeClassName | string | `"nvidia"` | RuntimeClassName configuration, set to "nvidia" if the model requires GPU |
| servingEngineSpec.schedulerName | string | `""` | SchedulerName configuration |
| servingEngineSpec.securityContext | object | `{}` | Pod-level security context configuration. https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#podsecuritycontext-v1-core |
| servingEngineSpec.servicePort | int | `80` | Service port |
| servingEngineSpec.sidecar | object | `{"image":"lmcache/lmstack-sidecar:latest","imagePullPolicy":"Always"}` | Sidecar configuration |
| servingEngineSpec.startupProbe | object | `{"failureThreshold":60,"httpGet":{"path":"/health","port":8000},"initialDelaySeconds":15,"periodSeconds":10}` | Readiness probe configuration |
| servingEngineSpec.startupProbe.failureThreshold | int | `60` | Number of times after which if a probe fails in a row, Kubernetes considers that the overall check has failed: the container is not ready |
| servingEngineSpec.startupProbe.httpGet | object | `{"path":"/health","port":8000}` | Configuration of the Kubelet http request on the server |
| servingEngineSpec.startupProbe.httpGet.path | string | `"/health"` | Path to access on the HTTP server |
| servingEngineSpec.startupProbe.httpGet.port | int | `8000` | Name or number of the port to access on the container, on which the server is listening |
| servingEngineSpec.startupProbe.initialDelaySeconds | int | `15` | Number of seconds after the container has started before startup probe is initiated |
| servingEngineSpec.startupProbe.periodSeconds | int | `10` | How often (in seconds) to perform the startup probe |
| servingEngineSpec.strategy | object | `{}` | deployment strategy |
| servingEngineSpec.tolerations | list | `[]` | Tolerations configuration (when there are taints on nodes) Example: tolerations:   - key: "node-role.kubernetes.io/control-plane"     operator: "Exists"     effect: "NoSchedule" |
