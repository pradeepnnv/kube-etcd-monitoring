### Create `etcd-monitoring` namespace
```
kubectl apply -f namespace.yaml
```

### Create the secret with etcd certs.


Execute below command to create the secret.

```
podname=$(kubectl get pods -o=jsonpath='{.items[0].metadata.name}' -l k8s-app=kube-apiserver -n kube-system)

kubectl create secret generic etcd-client-cert -n etcd-monitoring \
  --from-literal=etcd-ca="$(kubectl exec $podname -n kube-system -- cat /etc/kubernetes/pki/kube-apiserver/etcd-ca.crt)" \
  --from-literal=etcd-client="$(kubectl exec $podname -n kube-system -- cat /etc/kubernetes/pki/kube-apiserver/etcd-client.crt)" \
  --from-literal=etcd-client-key="$(kubectl exec $podname -n kube-system -- cat /etc/kubernetes/pki/kube-apiserver/etcd-client.key)"
```

### Create a service for etcd
```
kubectl apply -f kube-etcd-svc.yaml
```

Validate that the service `etcd-monitoring-svc` has endpoints.

### Create Prometheus for etcd

```
kubectl apply -f etcd-prometheus.yaml
```

This creates below components.
*  Prometheus
*  Static Target

Login to Prometheus UI.

Tunnel:
`kubectl -n etcd-monitoring port-forward svc/etcd-prometheus 9090:9090`

Prometheus UI URL: [Prometheus](http://localhost:9090)

Verify that etcd targets are working without issues.

### Create Grafana
```
kubectl apply -f etcd/poc/etcd-grafana.yaml
```

This creates below components.
*  Grafana
*  Prometheus Datasource
*  ETCD Dashboard

Login to Grafana UI.
Tunnel: `kubectl -n etcd-monitoring port-forward svc/etcd-grafana 8080:80`

Grafana URL: [Grafana](http://localhost:8080)
