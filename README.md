# azdo-aks-agents

This repository contains the code and pipeline for provisioning one or more build agents for azure devops. See my blog post for further detail on setup and usage.

## References

- https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#linux
- https://github.com/emberstack/docker-azure-pipelines-agent

## Prerequisites

1. An already created ACR, AKS cluster, and Key Vault.
2. In Azure Devops, create a new Agent Pool.
3. Create a PAT token with a service account.
4. In Azure Devops, at the **Organization** level, grant this service account Administrator permissions to Agent Pools (Organization Settings - Agent Pools - Security). Granting administrator access to individual pools at the ORG level, or to pools at the PROJECT level, does not seem to be enough privilege.

## Images and Charts

The `aks-build-agent` docker image and `azure-pipelines-agent` Helm can be built nightly and pushed to an ACR via the Azure Pipeline. See the example in this repo.

## Deployment

### Requests and Limits - Performance Observations

Requests/Limits are set in the HelmRelease file in Flux. When the agents first start, their idle utilization looks like this...

```bash
azure-pipelines-agent-0   1m           103Mi
```

When builds run on the agents...

- CPU utilization can be as high as 1-1.5 cores but generally hovers in the 300-700m range
- Memory utilization runs in the 300-800Mi range

After builds have run and the agents are idle again, CPU utilization returns to `1m`, but memory settles around `300-500Mi` even though idle.

### Horizontal Pod Autoscaling

The HPA settings are configured here at `helm/azure-pipelines-agent/values.yaml`. CPU and memory targets are set so that additional agents will spin up pretty quickly to meet build demand.

The memory target is set fairly high(300%) so that when pods go idle the agents will de-scale. In testing this has seemed to work fairly well.

### Rescheduled Agents Impact Builds

If an agent pod is rescheduled, it kills the running job and a notification appears in the Azure Devops Pipeline.

Agents pods can get rescheduled during a Kubernetes cluster upgrade, or the Helm Release changes (when the upstream images change for example).
