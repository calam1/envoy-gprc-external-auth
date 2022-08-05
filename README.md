example envoy filter

```


---
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: core-authz-filter-patch
spec:
  workloadSelector:
    labels:
      app: python-api
  configPatches:
    - applyTo: CLUSTER
      match:
        cluster:
          service: authz.resiliency-dev.svc.cluster.local
      patch:
        operation: MERGE
        value:
          name: external.authz.resiliency-dev.svc.cluster.local
---

apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: core-authz-filter
spec:
  workloadSelector:
    labels:
      app: python-api
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_INBOUND
        listener:
          filterChain:
            filter:
              name: "envoy.http_connection_manager"
              subFilter:
                name: "envoy.router"
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.ext_authz
          connect_timeout: 1.0s
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthz            
            grpc_service:
              envoy_grpc:
                cluster_name: external.authz.resiliency-dev.svc.cluster.local
              timeout: 1.0s
            transport_api_version: V3



```
