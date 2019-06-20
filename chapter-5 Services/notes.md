`kubectl exec kubia-7nog1 -- curl -s http://10.111.249.15`

Run curl in the kubia-7nog1 pod's main container
The double dash signals the end of command options for kubectl

If you want all requests made by a client to go to the same pod every time set the service's sessionAffinity to ClientIP in the spec

```
apiVersion: v1
kind: Service
spec:
  sessionAffinity: ClientIP
```

Pods created after a service get service related env variables

```
$ kubectl exec kubia-3inly env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubia-3inly
KUBERNETES_SERVICE_HOST=10.111.240.1
KUBERNETES_SERVICE_PORT=443
...
KUBIA_SERVICE_HOST=10.111.249.153
KUBIA_SERVICE_PORT=80
...
```

Dashes in the service name are converted to underscores and all letters are uppercased when the service name is used as the prefix in the environment variable’s name.

Can also connect via DNS
`backend-database.default.svc.cluster.local`

`backend-database` corresponds to the service name, `default` stands for the namespace the service is defined in, and `svc.cluster.local` is a configurable cluster domain suffix used in all cluster local service names. When in the same namespace can just use `backend-database.default`

Services build an Endpoints resource from the pod selector. For external services you need to create a manual endpoints resource

ExternalName services are implemented solely at the DNS level—a simple CNAME DNS record is created for the service. Therefore, clients connecting to the service will connect to the external service directly, bypassing the service proxy completely. For this reason, these types of services don’t even get a cluster IP.

To allow external traffic can use:

- NodePort Service
- Loadbalancer Service
- Ingress

To configure Google Cloud Platform's firewalls to allow connections to nodes on that port (when using NodePort)
`gcloud compute firewall-rules create kubia-svc-rule --allow=tcp:30123`

For minikube ingress needs to be enabled
`minikube addons list`
`minikube addons enable ingress`

Ingress controllers on cloud providers (in GKE, for example) require the Ingress to point to a NodePort service.

To access your service through http://kubia.example.com, you’ll need to make sure the domain name resolves to the IP of the Ingress controller.

`kubectl get ingresses` to get IP

> You should always define a readiness probe, even if it’s as simple as sending an HTTP request to the base URL
