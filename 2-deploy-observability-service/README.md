Next, we will learn how to monitor cloud infrastructure and services.
The set of services we will deploy will also be the subject of the next step.

# Background knowledge

If you know all these components, please skip this part and start the following operations directly.

all our service is running on EC2.

`node_exporter` is used to expose EC2 monitoring metrics, daemonset will ensure that a node_exporter will be deployed on each EC2.

`vmcluster` is a set of services for metrics storage and search, and its cluster mode helps it have higher availability than Prometheus. `vmcluster` consists of the following services
* vmstorage - stores the raw data and returns the queried data on the given time range for the given label filters.
* vminsert - accepts the ingested data and spreads it among vmstorage nodes according to consistent hashing over metric name and all its labels.
* vmselect - performs incoming queries by fetching the needed data from all the configured vmstorage nodes.

`vmagent` is a tiny agent which helps you collect metrics from various sources, and store them in vmcluster.

`vmalert` executes a list of the given alerting rules. For sending alerting notifications vmalert relies on Alertmanager.

`Alertmanager` handles alerts sent by `vmalert`, such as routing alerts to the correct receiver such as email. In this lab, we omit the part of alert notification.

# Operations
1. Deploy vmoperator.
    ```bash
    # cd ../2-deploy-observability-service
    $ kubectl apply -f 2-1-vmoperator/crd.yaml
    $ kubectl apply -f 2-1-vmoperator/rbac.yaml
    $ kubectl apply -f 2-1-vmoperator/manager.yaml
    $ kubectl get pod -n monitoring-system
    NAME                           READY   STATUS    RESTARTS   AGE
    vm-operator-848db5c8b4-fjm2w   1/1     Running   0          45s
    ```
2. Deploy vmcluster to provide metrics read and write services.
    ```bash
    $ kubectl apply -f 2-2-vmcluster/vmcluster.yaml
    $ kubectl get pod -n observability
    NAME                                  READY   STATUS    RESTARTS   AGE
    vminsert-vmcluster-7c78fc7bd8-rmjpj   1/1     Running   0          2m9s
    vmselect-vmcluster-0                  1/1     Running   0          2m9s
    vmstorage-vmcluster-0                 1/1     Running   0          2m9s
    ```
3. Deploy vmagent with monitoring metrics.
    ```bash
    $ kubectl apply -f 2-3-vmagent/exporter.yaml
    $ kubectl apply -f 2-3-vmagent/scrapes.yaml
    $ kubectl apply -f 2-3-vmagent/vmagent.yaml
    $ kubectl get pod -n observability
    node-exporter-552qv                      1/1     Running   0          35s
    node-exporter-mnwsk                      1/1     Running   0          35s
    vmagent-default-vmagent-f4c75dc7-n2bpb   2/2     Running   0          22s
    vminsert-vmcluster-7c78fc7bd8-9f2k5      1/1     Running   0          4m27s
    vmselect-vmcluster-0                     1/1     Running   0          4m28s
    vmstorage-vmcluster-0                    1/1     Running   0          4m28s
   
    # View the metrics we collected 
    $ kubectl port-forward svc/vmselect-vmcluster -n observability 8481:8481
    Forwarding from 127.0.0.1:8481 -> 8481
    Forwarding from [::1]:8481 -> 8481

    # visit
    http://127.0.0.1:8481/select/0:0/vmui/?#/cardinality
    ```
4. Deploy vmalert with alert rules.
    ```bash
    $ kubectl apply -f 2-4-vmalert/rule.yaml
    $ kubectl apply -f 2-4-vmalert/alertmanager.yaml
    $ kubectl apply -f 2-4-vmalert/vmalert.yaml
    
    # View the alerts
    $ kubectl port-forward svc/vmalertmanager-default-alertmanager 9093:9093
    Forwarding from 127.0.0.1:9093 -> 9093
    Forwarding from [::1]:9093 -> 9093

    # visit
    http://127.0.0.1:9093/#/alerts
    ```