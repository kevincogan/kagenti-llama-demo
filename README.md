# Introduction
This README will provide guidance for using the `Kagenti` project with `Llama Stack`.

The model could optionally be provided in-cluster OR served external to the cluster. The scenario described here at minimum needs OpenShift AI specifically with the capabilities of `Llama Stack`.

## RHOAI
Ensure the RHOAI operator is installed through Operator hub.

An example model deployment is located at `kubernetes/llama3.2-3b/`

## LLama Stack
By default, OpenShift does not initialize Llama Stack. When deploying the default `Data Science Cluster` modify the YAML and enable `llamastackoperator`:

``` yaml
    llamastackoperator:
      managementState: Managed
```

The operator will become ready once Llama stack is initialized.

To validate `Llama Stack` is ready and available the following can be done.

``` bash
oc get llsd -n default
No resources found in default namespace.
```

``` bash
oc get po -n redhat-ods-applications | grep llama
llama-stack-k8s-operator-controller-manager-64554d8c6f-6hkp5      1/1     Running   0          4h33m
```

### Configuring Llama Stack
An example `LLama Stack` deployment exists in `kubernetes/llama-stack-dist` modify the VLLM_URL and INFERENCE_MODEL with your values before deploying.

``` bash
      env:
        - name: INFERENCE_MODEL
          value: "llama32-3b"
        - name: VLLM_URL
          value: "https://llama32-3b.serving.svc.cluster.local/v1"
```

### Testing Llama Stack
The `LLama Stack` endpoint can be tested by doing a port forward and a curl.

``` bash
kubectl port-forward -n serving svc/lsd-llama32-3b-service 8321:8321
curl -X POST http://localhost:8321/v1/inference/chat-completion \
  -H "Content-Type: application/json" \
  -d '{
    "model_id": "vllm-inference/llama32-3b",
    "messages": [
      {
        "role": "user",
        "content": "Hello, how are you?"
      }
    ],
    "sampling_params": {
      "max_tokens": 100
    },
    "stream": false
  }'
{"metrics":[{"metric":"prompt_tokens","value":16,"unit":null},{"metric":"completion_tokens","value":27,"unit":null},{"metric":"total_tokens","value":43,"unit":null}],"completion_message":{"role":"assistant","content":"I'm doing well, thank you for asking. How can I assist you today?","stop_reason":"end_of_turn","tool_calls":[]},"logprobs":null}%
```

# Kagenti
