# LeaderWorkerSet (LWS) Integration

Kthena ModelServing provides native support for the **LeaderWorkerSet (LWS)** API. This capability allows users to define inference workloads using the standard LWS Custom Resource Definition (CRD) while leveraging Kthena's powerful orchestration, routing, and autoscaling features underneath.

This guide explains how to use LeaderWorkerSet resources with Kthena.

## Overview

LeaderWorkerSet (LWS) is a widely adopted API for describing multi-host inference workloads (e.g., LLM inference). Kthena integrates LWS support by directly watching and handling `LeaderWorkerSet` Custom Resources.

**Key Features:**

- **LWS API Compatibility**: Users can submit standard LeaderWorkerSet CRs directly.
- **Zero Extra Infrastructure**: No need to deploy the native LWS Controller; Kthena's ModelServing Controller handles the logic.
- **Kthena Powered**: Automatically inherits Kthena's capabilities like ModelRoute and Autoscaling.
- **Seamless Migration**: Ideal for users already using LWS who want to migrate to Kthena without rewriting their manifests.

## How It Works

When Kthena's LWS integration is enabled:

1.  **Direct Processing**: The Model Serving Controller listens for `LeaderWorkerSet` resources.
2.  **One-Way Conversion**: It automatically converts the LWS specification into Kthena's internal `ModelServing` resources.
3.  **Status Sync**: The status of the underlying pods is aggregated and written back to the `LeaderWorkerSet` status, allowing you to use standard `kubectl get lws` commands to monitor progress.

> **Note**: This is a one-way synchronization. Changes should be made to the `LeaderWorkerSet` resource, which will propagate to the underlying Kthena resources.

## Configuration Mapping

Kthena maps `LeaderWorkerSet` fields to `ModelServing` concepts as follows:

| LeaderWorkerSet Field | Kthena ModelServing Equivalent | Description |
|-----------------------|--------------------------------|-------------|
| `metadata.name` | `ModelServing` Name | Identifies the workload. |
| `spec.replicas` | `spec.replicas` | The number of independent serving groups (e.g., number of model replicas). |
| `spec.leaderWorkerTemplate.leaderTemplate` | Role: `leader` | Defines the configuration for the Leader Pod. |
| `spec.leaderWorkerTemplate.workerTemplate` | Role: `worker` | Defines the configuration for Worker Pods. |
| `spec.leaderWorkerTemplate.size` | Worker Count Calculation | Determines the number of workers per group. `Worker Replicas = Size - 1`. |
| `spec.startupPolicy` | Startup Policy | Maps to the startup ordering (e.g., LeaderFirst). |

## Deployment Example

Below is an example of deploying an inference workload using `LeaderWorkerSet`.

### Prerequisites

Ensure the `LeaderWorkerSet` CRD is installed in your cluster. You do **not** need to install the LWS controller or operator.

```bash
# Example: Install LWS CRD only
kubectl apply -f https://github.com/kubernetes-sigs/lws/releases/download/v0.3.0/crd.yaml
```

### LWS Configuration

This example defines a deployment with 1 replica group. Each group consists of 1 Leader and 1 Worker (Size = 2).

<details>
<summary>
<b>lws-inference-example.yaml</b>
</summary>

```yaml
apiVersion: leaderworkerset.x-k8s.io/v1
kind: LeaderWorkerSet
metadata:
  name: qwen-72b-inference
  namespace: default
spec:
  # Number of independent model replicas (Serving Groups)
  replicas: 1
  
  leaderWorkerTemplate:
    # Total size of the group (1 Leader + 1 Worker)
    size: 2
    
    # Leader Pod Configuration
    leaderTemplate:
      metadata:
        labels:
          role: leader
          model: qwen-72b
      spec:
        containers:
        - name: inference-server
          image: vllm/vllm-openai:latest
          env:
          - name: ROLE
            value: "leader"
          - name: MASTER_ADDR
            value: "localhost" 
          ports:
          - containerPort: 8000
            name: http
          resources:
            limits:
              nvidia.com/gpu: "8"
            
    # Worker Pod Configuration
    workerTemplate:
      metadata:
        labels:
          role: worker
          model: qwen-72b
      spec:
        containers:
        - name: inference-worker
          image: vllm/vllm-openai:latest
          env:
          - name: ROLE
            value: "worker"
          command: ["/bin/sh", "-c", "python3 -m vllm.entrypoints.worker ..."]
          resources:
            limits:
              nvidia.com/gpu: "8"
```

</details>

### Verifying Deployment

After applying the YAML, you can check the status using standard kubectl commands:

```bash
# Check the LeaderWorkerSet status
kubectl get lws qwen-72b-inference

# Check the underlying Pods created by Kthena
kubectl get pods -l leaderworkerset.x-k8s.io/name=qwen-72b-inference
```

## Status & Troubleshooting

The `LeaderWorkerSet` status is automatically updated by Kthena:

- **ReadyReplicas**: Indicates how many serving groups are fully ready.
- **Conditions**: Provides details on the health and state of the deployment.

If the `LeaderWorkerSet` is not progressing:
1. Check if the CRD is installed correctly.
2. Inspect the Kthena Controller logs for any validation errors regarding the LWS spec.
3. Verify that the resource requests (GPUs, CPU) can be satisfied by the cluster.
