# Istio AuthorizationPolicy Helm Template

This Helm chart generates Istio `AuthorizationPolicy` resources for specified Kubernetes namespaces, allowing ingress traffic only from configured IP blocks.

## Overview

This chart dynamically creates an `AuthorizationPolicy` named `allow-all` in each namespace listed in your values. The policy allows traffic **only** from the provided IP CIDR blocks (`ipBlocks`). All other sources are denied by default (unless permitted by other Istio policies).

## Template Example

```yaml
{{- range .Values.namespaces }}
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-all
  namespace: {{ . }}
spec:
  rules:
    - from:
        - source:
            ipBlocks:
              {{- range $.Values.ipBlocks }}
              - {{ . }}
              {{- end }}
{{- end }}
```

## Values

| Key             | Type     | Description                                             | Example                      |
|-----------------|----------|---------------------------------------------------------|------------------------------|
| `namespaces`    | list     | List of namespaces to apply the AuthorizationPolicy to  | `[dev, prod, staging]`       |
| `ipBlocks`      | list     | List of allowed source CIDR IP blocks                   | `[10.0.0.0/8, 192.168.0.0/16]` |

## Usage

1. **Set your values** in a `values.yaml` file:
    ```yaml
    namespaces:
      - dev
      - prod
    ipBlocks:
      - 10.0.0.0/8
      - 192.168.1.0/24
    ```

2. **Install the chart**:
    ```sh
    helm install my-authz-policy ./path-to-chart -f values.yaml
    ```

3. **Result:** An `AuthorizationPolicy` named `allow-all` will be created in each specified namespace, permitting traffic only from the listed IP blocks.

## Example Rendered Output

```yaml
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-all
  namespace: dev
spec:
  rules:
    - from:
        - source:
            ipBlocks:
              - 10.0.0.0/8
              - 192.168.1.0/24
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-all
  namespace: prod
spec:
  rules:
    - from:
        - source:
            ipBlocks:
              - 10.0.0.0/8
              - 192.168.1.0/24
```

## Notes

- This chart assumes you are using [Istio](https://istio.io/) as your service mesh.
- The policy is permissive for the listed IP blocks and restrictive for all others.
- You may need additional policies for more granular control.

## References

- [Istio Authorization Policy Documentation](https://istio.io/latest/docs/reference/config/security/authorization-policy/)
- [Helm Templates](https://helm.sh/docs/topics/chart_template_guide/)
