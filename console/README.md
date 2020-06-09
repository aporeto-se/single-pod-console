# Aporeto console

This chart deploy the Aporeto console into one single pod.

## Prerequisites Details

* Kubernetes 1.8+
* Aporeto console license (otherwise a trial version up to 2 enforcers will be used)

## Chart Details

This chart will do the following:

* Deploy Aporeto console into one deployment

## Using the Chart

### Add Aporeto Helm repository

Before installing Aporeto helm charts, you need to add the [Aporeto helm repository](https://charts.aporeto.com/svcs) to your helm client

```bash
helm repo add aporeto https://charts.aporeto.com/svcs
```

### Deploying Aporeto console

As simple as:

```bash
helm install console aporeto/console
  --set console.url=https://myconsole.aporeto.com \
  --set networking.serviceType=LoadBalancer \
  --set storage.class=standard
```

Note: `console.url` can be updated afterward as explained in the note once deployed.

### Upgrading Aporeto console

As simple as:

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

### Use your own certificates

You can provide you custom certificates for two things:

* As a root certificate authority
* As a servicer certificate to use for console access

#### Use your own certificate authority

> Note: To use a custom root CA you must pass the volume with the root CA **during the first launch**.
> If done afterward nothing will be done and the custom root CA will not be used.

In a local folder make sure to have the following:

For a custom root CA in a `certs` folder:

* `ca-cert.pem` in x509 format
* `ca-key.pem` in x509 format **not encrypted**

Then create a Kubernetes secret as follow:

```console
kubectl create secret generic my-ca --from-file certs/
```

Then use the `--set console.certificates=my-ca` when deploying with helm.

### Use your own service certificate for console endpoints

For a custom service certificate in a `certs` folder:

* `public-ca.pem` a x509 chain of certificate to validate the following cert
* `public-cert.pem` in x509 format
* `public-key.pem` in x509 format **not encrypted**

> Note: Make sure that `APORETO_CONSOLE_URL` is matching in your certificate either:
>
> * in the common name (CN)
> * in the subject alternative names (SAN)

Then create a Kubernetes secret as follow:

```console
kubectl create secret generic my-service-certs --from-file certs/
```

Then use the `--set console.certificates=my-service-certs` when deploying with helm.

> Note: To renew the certificate if needed repeat the previous step and restart the pod.

### Customize the console

You can customize some settings via a configuration file, here is an example of configuration you can set:

```console
CLAD_ALLOW_GOOGLE_ACCESS                    Set it to false to disable  Sign in with google
CLAD_GOOGLE_OAUTH_CLIENT_ID                 To set a google oauth client id to enable the Signin with google
CLAD_ALLOW_LDAP_ACCESS                      Set to false to disable Sign in with ldap
CLAD_ALLOW_VINCE_ACCESS                     Set to false to disable Sign in with aporeto account
CLAD_ALLOW_OIDC_ACCESS                      Set to false to disable Sign in with oidc
CLAD_ALLOW_SAML_ACCESS                      Set to false to disable Sign in with saml
CLAD_ALLOW_CERT_ACCESS                      Set to false to disable Sign in with a certificate
CLAD_RECAPTCHA_KEY                          To enable CAPTCHA test during account creation
CLAD_BANNER_COLOR                           To set a banner color for the UI, Hex value as #RRGGBB
CLAD_TOKEN_VALIDITY                         Set the token validity. Detault is 720h
CLAD_CHAT_URL                               Use a custom URL for the chat button on loging page
CLAD_SUPPORT_URL                            Use a custom URL for the support button in top navigation bar
CACTUAR_AUTHORIZED_TLDS                     strings List of TLDs authorized to make network calls
CACTUAR_EXCLUDED_NETWORKS                   strings List of networks excluded for remote http calls
CANYON_WHOIS_SERVER                         strings List of whois serverd (default [whois.iana.org,whois.ripe.net])
HIGHWIND_CONTROLLER_TYPE                    The type of controller highwind will use to deploy apps [required] [allowed: kubernetes,swarm]
HIGHWIND_DOCKER_CA                          Path to docker ca cert
HIGHWIND_DOCKER_CLIENT_KEY                  Path to docker client key
HIGHWIND_DOCKER_HOST                        Docker host (default "unix:///var/run/docker.sock")
HIGHWIND_DOCKER_PORT                        Docker port (default "2376")
HIGHWIND_DOCKER_REGISTRY                    Docker registry (default "docker.io")
HIGHWIND_DOCKER_REGISTRY_PASSWORD           Docker registry password
HIGHWIND_DOCKER_REGISTRY_USERNAME           Docker registry username
HIGHWIND_DOCKER_VERSION                     Docker version (default "1.31")
HIGHWIND_HELM_CACHE_DURATION                Helm cache duration (default 24h0m0s)
HIGHWIND_HELM_DEVEL                         If set, will only get charts on version 0.0.0-dev
HIGHWIND_HELM_HOME_DIRECTORY                Helm home directory (default "/tmp/.helm")
HIGHWIND_HELM_REPOSITORY                    Helm repository address [required]
HIGHWIND_HELM_REPOSITORY_CONFIG             Helm repository config (default "repositories.yaml")
HIGHWIND_K8S_NAMESPACE                      Kubernetes namespace where apps will be deployed (default "highwind")
HIGHWIND_KUBECONFIG                         Path to your kubeconfig
HIGHWIND_KUBECONTEXT                        Context to use from the config. Empty means use the default
HIGHWIND_MONITORING_INTERVAL                Interval between each status check (default 5s)
HIGHWIND_NODE_AFFINITY_REQUIRED             Force node affinity
HIGHWIND_NODE_AFFINITY_TAG                  key=value
HIGHWIND_PUBLIC_HOST                        If given, apps will use this for their public hosts
HIGHWIND_UPGRADE_INTERVAL                   Interval between each upgrade check (default 12h0m0s)
MIDGARD_GOOGLE_OAUTH_CLIENT_ID              Google client ID
MIDGARD_GOOGLE_OAUTH_TOKEN_VALIDATION_URL   Google url validate Google issued JWT (default "https://www.googleapis.com/oauth2/v3/tokeninfo")
MIDGARD_JWT_COOKIE_DOMAIN                   Defines the domain for the cookie. Empty will use the api gateway domain
MIDGARD_JWT_COOKIE_SAME_SITE                Define same site policy applied to token cookies [allowed: strict,lax,none] (default "strict")
SEPHIROTH_AUTOMATIONS_AUTHORIZED_TLDS       strings List of TLDs authorized to make network calls
SEPHIROTH_AUTOMATIONS_EXCLUDED_NETWORKS     strings List of networks excluded for remote http calls
SEPHIROTH_AUTOMATIONS_EXEC_LIMIT            Maximum duration of JS function execution (default 10s)
SQUALL_AUTHORIZED_TLDS                      strings List of TLDs authorized to make network calls
SQUALL_EXCLUDED_NETWORKS                    strings List of networks excluded for remote http calls
VINCE_ACCOUNT_LOCKOUT_ATTEMPTS              The maximum number of failed password before locking out (default 5)
VINCE_ACCOUNT_LOCKOUT_DURATION              The time during the account will stay locked out (default 5m0s)
VINCE_DISABLE_ACCOUNT_CREATION              Disable completely new account creation
VINCE_EMAIL_INFO_RECIPIENT                  Information email will be sent to the provided email if provided
VINCE_EMAIL_INVITATION_RECIPIENT            Activation link will be sent to the provided email
VINCE_ENABLE_IMMEDIATE_ACTIVATION           Immediately activate accounts
VINCE_ENABLE_PASSWORD_LEAK_CHECK            If set, verify the password from haveibeenpawned
VINCE_RECAPTCHA_KEY                         Use recaptha to validate acccount creation
WUTAI_BLOCK_OPENTRACING_HEADERS             If set, opentracing headers sent by the clients will be stripped
WUTAI_GOOGLE_OAUTH_CLIENT_ID                Set the Google oauth client id used by the plaform
YUFFIE_SENDER                               Email address used to send email [required]
YUFFIE_SMTP_PASSWORD                        Password to connect to the SMTP server
YUFFIE_SMTP_SERVER                          SMTP server to send email [required]
YUFFIE_SMTP_USER                            Username to connect to the SMTP server
```

To use custom settings, create a file named `aporeto.env` (the name is important), then add what you want to set as follow:

```shell
# Configure google auth id login
export CLAD_GOOGLE_OAUTH_CLIENT_ID="<id>"
export MIDGARD_GOOGLE_OAUTH_CLIENT_ID="<id>"
export WUTAI_GOOGLE_OAUTH_CLIENT_ID="<id>"
```

Then create a config map as `kubectl create configmap myconfig --from-file=aporeto.env`.

Finally use the `myconfig` as a value for the `console.custom_env` setting.
