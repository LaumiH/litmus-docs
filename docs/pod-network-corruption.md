---
id: pod-network-corruption
title: Pod Network Corruption Experiment Details
sidebar_label: Pod Network Corruption  
---
------

## Experiment Metadata

| Type      | Description              | Tested K8s Platform                                               |
| ----------| ------------------------ | ------------------------------------------------------------------|
| Generic   | Inject Packet Loss Into Application Pod | GKE, Konvoy(AWS), Packet(Kubeadm), OpenShift(Baremetal), minikube >= 1.6 |

## Prerequisites

- Ensure that the Litmus Chaos Operator is running
- Ensure that the `pod-network-corruption` experiment resource is available in the cluster. If not, install from [here](https://hub.litmuschaos.io/charts/generic/experiments/pod-network-corruption)
- <div class="danger">
    <strong>NOTE</strong>: 
        Experiment is supported only on Docker Runtime. Support for containerd/CRIO runtimes will be added in subsequent releases.
</div>

## Entry Criteria

- Application pods are healthy before chaos injection

## Exit Criteria

- Application pods are healthy post chaos injection

## Details

- Pod-network-corruption causes a specified percentage of corrupt network packets being sent from kubernetes pods.
- Pumba is used to cause the corruption.
- The application pod should be healthy once chaos is stopped. Service-requests should be served despite chaos.


## Steps to Execute the Chaos Experiment

- This Chaos Experiment can be triggered by creating a ChaosEngine resource on the cluster. To understand the values to provide in a ChaosEngine specification, refer [Getting Started](getstarted.md/#prepare-chaosengine)

- Follow the steps in the sections below to prepare the ChaosEngine & execute the experiment.

### Prepare ChaosEngine

- Provide the application info in `spec.appinfo`
- Override the experiment tunables if desired

#### Supported Experiment Tunables

| Variables             | Description                                                  | Type      | Notes            |
| ----------------------| ------------------------------------------------------------ |-----------|------------------|
| NETWORK_INTERFACE     | Name of ethernet interface inside tthe container to be targeted | Mandatory | Default: eth0 |
| TARGET_CONTAINER      | Name of container which is subjected to packet corruption    | Mandatory |                  |
| NETWORK_PACKET_CORRUPTION_PERCENTAGE | The packet corruption in percentage	       | Mandatory | Default: 100     |
| TOTAL_CHAOS_DURATION  | The time duration for chaos insertion in milliseconds        | Mandatory | Default: 60000ms |                                           
| LIB                   | The chaos lib used to inject the chaos                       | Optional  | Default: pumba   |
| LIB_IMAGE             | The docker image to be used from the LIB                     | Optional  | Default: gaiaadm/pumba:0.6.5; note: versions < 0.6 are not supported! |
| CHAOSENGINE           | ChaosEngine name associated with the experiment instance     | Optional  |                  |
| CHAOS_SERVICE_ACCOUNT | Service account used by the LIB                              | Optional  |                  |

#### Sample ChaosEngine Manifest

```yaml
# chaosengine.yaml
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: nginx-network-chaos
  namespace: default
spec:
  jobCleanUpPolicy: retain
  monitoring: false
  appinfo: 
    appns: default
    # FYI, To see app label, apply kubectl get pods --show-labels
    applabel: "app=nginx"
    appkind: deployment
  chaosServiceAccount: nginx 
  experiments:
    - name: pod-network-corruption
      spec:
        components:
        - name: ANSIBLE_STDOUT_CALLBACK
          value: default
        - name: TARGET_CONTAINER
          value: "nginx" 
        - name: LIB_IMAGE
          value: gaiaadm/pumba:0.6.5
        - name: NETWORK_INTERFACE
          value: eth0                    
        - name: NETWORK_PACKET_CORRUPTION_PERCENTAGE
          value: "100"
        - name: TOTAL_CHAOS_DURATION
          value: "60000"
        - name: LIB
          value: pumba
```
### Create the ChaosEngine Resource

- Create the ChaosEngine to trigger chaos.

  `kubectl apply -f chaosengine.yml`

### Watch chaos progress

- View network packet corruption by pinging from the affected container:

  `kubectl exec -it pod-name -- /bin/bash` 
  `ping <ip_address>`

  The ping should stop without any notice of host unreachable. After the chaos is finished, restart the ping (it does not resume automatically).

### Check Chaos Experiment Result

- Check whether the application is resilient to the Pod Network Corruption, once the experiment (job) is completed.

  `kubectl describe chaosresult <ChaosEngine-Name>-<ChaosExperiment-Name> -n <application-namespace>`


## Application Pod Network Corruption Demo [TODO]

- A sample recording of this experiment execution is provided here.
