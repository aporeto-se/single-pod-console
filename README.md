# Aporeto console

This is a brief walkthrough of requirements and setup of the aporeto console as a single pod. The deployment works on any kubernetes cluster, but doc assumes GKE

## Prerequisites Details

* Kubernetes 1.8+
* Aporeto console license (otherwise a trial version up to 2 enforcers will be used)

## Chart Details

This chart will do the following:

* Deploy Aporeto console into one deployment

You can download the chart, and the values file, if you want more details. The './console` directory has the chart and the details for your reference. 

```bash
  helm pull aporeto/console
```

## Using the Chart

### Add Aporeto Helm repository

Before installing Aporeto helm charts, you need to add the [Aporeto helm repository](https://charts.aporeto.com/svcs) to your helm client

```bash
helm repo add aporeto http://helm.aporeto.us/master/stable/svcs
```

If you have already added the Helm repo, update it regularly

```bash
  helm repo update 
```

### Deploying Aporeto console

Deploy the helm chart

```bash
helm install console aporeto/console  -n aporeto-console \
  --set console.url=https://demo.prismacloud.us \
  --set storage.class=standard \
  --set networking.serviceType=LoadBalancer\
```

Note: `console.url` can be updated afterward as explained in the note once deployed.

Verify the pods are coming online

```bash
  kubectl get pods
```

If the pod comes up just fine, then you must update the console config with the new external IP assigned to the load balancer. Use the 2 commands below:

```bash
  CONSOLE_LB=$(kubectl -n aporeto-console get service console -o jsonpath="{.status.loadBalancer.ingress[0]['address','ip','hostname']}")
  helm upgrade -n aporeto-console   console   aporeto/console   --reuse-values   --set console.url=https://$CONSOLE_LB   

```

After a minute or so, the pod should restart and come online. You can now visit the aporeto web console via the external IP. Execute kubectl get svc and plug the console svc external IP into your web browser. 

Create a Aporeto account at the login screen to get started.

### Uninstalling Console

  Delete the console env

```bash
  helm uninstall console aporeto/console
```

  Delete the persistent volume claims

```bash
  kubectl delete pvc backup-console-0 data-console-0
```

### POC/Demo License

```bash
  helm upgrade console aporeto/console --reuse-values  --set console.license="$(cat aporeto-demo.lic)"
```

### Upgrading Aporeto console

```bash
  helm upgrade console aporeto/console

```

### Charts options

You can find the `values.yaml` file all the options you can set. Parameters are listed below.

General parameters:

| parameter               | default value       | comment                                                                                                                                    |
| ----------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| imageRegistry           | `docker.io/aporeto` | The docker image registry to use. Change it if you are using a private registry                                                            |
| imagePullSecret         |                     | The secret name to use as [imagePullSecret](https://kubernetes.io/docs/concepts/containers/images/#creating-a-secret-with-a-docker-config) |
| nodeAffinity.enabled    | false               | To enable the node affinity if you want to schedule that container to a specific set of nodes                                              |
| nodeAffinity.key        | type                | The key of the label to use                                                                                                                |
| nodeAffinity.value      | console             | the value of the label to use                                                                                                              |
| resources.enabled       | false               | Enable the resource limit and request for the pod                                                                                          |
| resources.limits.cpu    | 20000m              | The CPU limit in milli-core                                                                                                                |
| resources.limits.memory | 32Gi                | The memory limit                                                                                                                           |

Console parameters:

| parameter              | default value       | comment                                                                                |
| ---------------------- | ------------------- | -------------------------------------------------------------------------------------- |
| console.url            | `https://localhost` | Required, This is the url you will use to reach the Aporeto control plane              |
| console.api            | console.url:4443    | Optional, This is the url you can use to reach the Aporeto control plane api           |
| console.license        | trial version       | Optional, The aporeto license to use for this deployment.                              |
| console.certificates   |                     | Optional, the kubernetes secret name that contains the certificates to use (see below) |
| console.custom_env     |                     | Optional, Used to pass custom settings as a config map name                            |
| console.encryption_key |                     | Optional, Set the encryption_key to encrypt secrets on the storage                     |

Third party apps parameters:

| parameter                  | default value   | comment                                                               |
| -------------------------- | --------------- | --------------------------------------------------------------------- |
| apps.namespace             | same as console | Optional, Set the namespace where the 3rd party apps will be deployed |
| apps.imageRegistry         | same as console | Optional, Set the docker registry if different than console           |
| apps.helmRepository        | automatic       | Optional, Set the Helm repository to use                              |
| apps.nodeAffinity.required | false           | Optional, Enable the node affinity for 3rd party apps                 |
| apps.nodeAffinity.tag      |                 | Set the Kubernetes label to use for node affinity selector            |

Storage parameters:

| parameter     | default value | comment                                                   |
| ------------- | ------------- | --------------------------------------------------------- |
| storage.class | none          | Required, the storage class to use to store database data |
| storage.size  | 500           | The size of the storage in GiB                            |

Network parameters:

| parameter                         | default value | comment                                                                                                       |
| --------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------- |
| networking.serviceType            | NodePort      | The networking type to use between be LoadBalancer, NodePort or ClusterIP                                     |
| networking.nodePort.consolePort   |               | Optional, Use this to set a static node port for console web interface endpoint                               |
| networking.nodePort.apiPort       |               | Optional, Use this to set a static node port for api endpoint                                                 |
| networking.loadBalancer.consoleIP |               | Optional, Use this to set a static IP for console endpoint                                                    |
| networking.loadBalancer.apiIP     | consoleIP     | Optional, Use this to set a static IP for console api endpoint. By default it uses the consoleIP on port 4443 |

Backup parameters:

| parameter        | default value | comment                                             |
| ---------------- | ------------- | --------------------------------------------------- |
| backend.schedule | `0 0 * * *`   | Optional, the schedule of the backup in cron format |
