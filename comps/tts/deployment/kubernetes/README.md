# Deploy tts microservice on Kubernetes cluster

- You should have Helm (version >= 3.15) installed. Refer to the [Helm Installation Guide](https://helm.sh/docs/intro/install/) for more information.
- For more deployment options, refer to [helm charts README](https://github.com/opea-project/GenAIInfra/tree/main/helm-charts#readme).

## Deploy on Kubernetes

```
export HFTOKEN="insert-your-huggingface-token-here"
helm install tts oci://ghcr.io/opea-project/charts/tts --set global.HUGGINGFACEHUB_API_TOKEN=${HFTOKEN} -f cpu-values.yaml
```