This Docker Image contains the Oracle WebLogic Server Kubernetes Operator, which is available and open sourced at (https://oracle.github.io/weblogic-kubernetes-operator).  The operator can manage any number of WebLogic domains running in a Kubernetes environment.  It provides a mechanism to create domains, automates domain startup, allows scaling WebLogic clusters up and down either manually (on-demand) or through integration with WLDF or Prometheus, manages load balancing for web applications deployed in WebLogic clusters, and provides integration to ElasticSearch, Logstash and Kibana.
The operator uses the standard Oracle WebLogic Server 12.2.1.3 Docker image, from the Docker store.  It treats this image as immutable, and all of the state is persisted in a Kubernetes persistent volume.  This allows us to treat all of the pods as throwaway and replaceable, and it completely eliminates the need to manage state written into Docker containers at runtime (because there is none).
The operator can expose the WebLogic administration console to external users (if desired), and can also allow external T3 access, e.g. for WLST.  Domains can talk to each other, allowing distributed transactions, etc. All of the pods are configured with Kubernetes liveness and readiness probes, so that Kubernetes can automatically restart failing pods, and the load balancer configuration can include only those managed servers in the cluster that are actually ready to service user requests.

# Getting Started
## The Oracle WebLogic Server Kubernetes Operator has the following requirements:
*	Kubernetes 1.7.5+, 1.8.0+, 1.9.0+, 1.10.0 (check with `kubectl version`).
*	Flannel networking v0.9.1-amd64 (check with `docker images | grep flannel`)
*	Docker 17.03.1.ce (check with `docker version`)
*	Oracle WebLogic Server 12.2.1.3.0

## Customizing the operator parameters file
The operator is deployed with the provided installation script (`kubernetes/create-weblogic-operator.sh`). The input to this script is the file (`kubernetes/create-operator-inputs.yaml`), which needs to updated to reflect the target environment.
The following parameters must be provided in the input file:
### CONFIGURATION PARAMETERS FOR THE OPERATOR

| Parameter | Definition | Default |
| --- | --- | --- |
| elkIntegrationEnabled | Determines whether the ELK integration will be enabled.  If set to `true`, then ElasticSearch, logstash and Kibana will be installed, and logstash will be configured to export the operator’s logs to ElasticSearch. | false |
| externalDebugHttpPort | The port number of the operator's debugging port outside of the Kubernetes cluster. | 30999 |
| externalOperatorCert | A base64 encoded string containing the X.509 certificate that the operator will present to clients accessing its REST endpoints. This value is only used when `externalRestOption` is set to `CUSTOM_CERT`. | |
| externalOperatorKey | A base64 encoded string containing the private key of the operator's X.509 certificate.  This value is only used when externalRestOption is set to `CUSTOM_CERT`. | |
| externalRestHttpsPort| The NodePort number that should be allocated for the operator REST server should listen for HTTPS requests on. | 31001 |
| externalRestOption | Which of the available REST options is desired.  Allowed values: <br/>- `NONE` Disable the REST interface.  <br/>- `SELF_SIGNED_CERT` The operator will use a self-signed certificate for its REST server.  If this value is specified, then the `externalSans` parameter must also be set. <br/>- `CUSTOM_CERT` Provide custom certificates, for example from an external certification authority. If this value is specified, then the `externalOperatorCert` and `externalOperatorKey` must also be provided.| NONE |
| externalSans| A comma-separated list of Subject Alternative Names that should be included in the X.509 Certificate.  This list should include ... <br/>Example:  `DNS:myhost,DNS:localhost,IP:127.0.0.1` | |
| internalDebugHttpPort | The port number of the operator's debugging port inside the Kubernetes cluster. | 30999 |
| javaLoggingLevel | The level of Java logging that should be enabled in the operator.  Allowed values are `SEVERE`, `WARNING`, `INFO`, `CONFIG`, `FINE`, `FINER`, and `FINEST` | INFO |
| namespace | The Kubernetes namespace that the operator will be deployed in.  It is recommended that a namespace be created for the operator rather than using the `default` namespace.| weblogic-operator |
| remoteDebugNodePortEnabled | Controls whether or not the operator will start a Java remote debug server on the provided port and suspend execution until a remote debugger has attached. | false |
| serviceAccount| The name of the service account that the operator will use to make requests to the Kubernetes API server. | weblogic-operator |
| targetNamespaces | A list of the Kubernetes namespaces that may contain WebLogic domains that the operator will manage.  The operator will not take any action against a domain that is in a namespace not listed here. | default |
| weblogicOperatorImage | The Docker image containing the operator code. | container-registry.oracle.com/middleware/weblogic-kubernetes-operator:latest |
| weblogicOperatorImagePullPolicy | The image pull policy for the operator docker image.  Allowed values are 'Always', 'Never' and 'IfNotPresent' | IfNotPresent |
| weblogicOperatorImagePullSecretName | Name of the Kubernetes secret to access the Docker Store to pull the WebLogic Server Docker image.  The presence of the secret will be validated when this parameter is enabled. | |

###Decide which REST configuration to use
The operator provides three REST certificate options:
*	none will disable the REST server.
*	self-signed-cert will generate self-signed certificates.
*	custom-cert provides a mechanism to provide certificates that were created and signed by some other means.
Decide which options to enable
The operator provides some optional features that can be enabled in the configuration file.
###Load Balancing
The operator can install the Traefik Ingress provider to provide load balancing for web applications running in WebLogic clusters. If enabled, an instance of Traefik and an Ingress will be created for each WebLogic cluster. Additional configuration is performed when creating the domain.

**Note:** that the Technology Preview release provides only basic load balancing:
*	Only HTTP(S) is supported. Other protocols are not supported.
*	A root path rule is created for each cluster. Rules based on the DNS name, or on URL paths other than ‘/’, are not supported.
*	No non-default configuration of the load balancer is performed in this release.
 
The default configuration gives round robin routing and WebLogic Server will provide cookie-based session affinity.

**Note:** that Ingresses are not created for servers that are not part of a WebLogic cluster, including the Administration Server. Such servers are exposed externally using NodePort services.

### Log integration with ELK
The operator can install the ELK stack and publish its logs into ELK. If enabled, ElasticSearch and Kibana will be installed in the default namespace, and a logstash pod will be created in the operator’s namespace. Logstash will be configured to publish the operator’s logs into Elasticsearch, and the log data will be available for visualization and analysis in Kibana.
To enable the ELK integration, set the enableELKintegration option to true.
 
### Deploying the operator to a Kubernetes cluster
To deploy the operator, run the deployment script and give it the location of your inputs file:
```./create-weblogic-operator.sh –i /path/to/create-operator-inputs.yaml```

#### What the script does
The script will carry out the following actions:
*	A set of Kubernetes YAML files will be created from the inputs provided.
*	A namespace will be created for the operator.
*	A service account will be created in that namespace.
*	If ELK integration was enabled, a persistent volume for ELK will be created.
*	A set of RBAC roles and bindings will be created.
*	The operator will be deployed.
*	If requested, the load balancer will be deployed.
*	If requested, ELK will be deployed and logstash will be configured for the operator’s logs.

The script will validate each action before it proceeds.
This will deploy the operator in your Kubernetes cluster.  Please refer to the documentation for next steps including using the REST services, creating a WebLogic domain, starting a domain, and so on. 


