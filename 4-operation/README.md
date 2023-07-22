We will answer the questions in 3-solve-availability-related-issues and improve the availability of the service through practical operations.

# cd ../4-optimize-observability-service

1. Add limits to resource
    ```
    resources:
      limits:
        cpu: 120m
        memory: 520Mi
      requests:
        cpu: 80m
        memory: 120Mi
    ```
   Update the yaml of vmoperator:
    ```
    $ kubectl apply -f 4-1-resource-limits/manager.yaml
    $ kubectl get pod -n monitoring-system -oyaml
    ```

2. We are making the following changes to vmcluster to improve the availability of our services.
   - Deploy multiple replicas of the same service. Checkout the yaml:
     ```
       replicaCount: 2
       # This configuration will start two replicas (pods) of the vminsert service.
       # When one of the pods crashes, the other pod can still provide services
     ```
   - Deploy multiple copies of the same service to different EC2. Checkout the yaml:
       ```
       podAntiAffinity:
       #the replicas get the label app.kubernetes.io/name=vmstorage.
       #The podAntiAffinity rule tells the scheduler to avoid placing multiple replicas with the app.kubernetes.io/name=vmstorage label on a single node.
       preferredDuringSchedulingIgnoredDuringExecution:
       - podAffinityTerm:
         labelSelector:
         matchExpressions:
         - key: app.kubernetes.io/name
         operator: In
         values:
         - vminsert
         topologyKey: kubernetes.io/hostname
         weight: 100     
       ```
   - Enable the data multi-copy mode in the service.
       ```
       replicationFactor: 2
       #replicationFactor instructs vminsert to store N copies for every ingested sample on N distinct vmstorage nodes.
       #This guarantees that all the stored data remains available for querying if up to N-1 vmstorage nodes are unavailable.
       ```

   Update the yaml of vmcluster:
    ```
    $ kubectl apply -f 4-2-high-availability-deployment/vmcluster.yaml
    $ kubectl get pod -n observability | grep vmcluster
    # we can see vmstorage > vminsert > vmselect
    vminsert-vmcluster-786bd859db-cltwh        1/1     Running            0               97s
    vminsert-vmcluster-786bd859db-tvl5m        1/1     Running            0               91s
    vmselect-vmcluster-0                       1/1     Running            0               113s
    vmselect-vmcluster-1                       1/1     Running            0               2m16s
    vmstorage-vmcluster-0                      1/1     Running            0               2m31s
    vmstorage-vmcluster-1                      1/1     Running            0               3m9s
    vmstorage-vmcluster-2                      1/1     Running            0               3m9s
    ```
    
3. Isolate data or services of different tenants.
   - Deploy a separate vmagent for each tenant, these vmagents may be on different VPCs or even different clouds in production environment.  Checkout the yaml:
     ```
     metadata:
     name: vmagent-1
     namespace: observability
     ```
   
   - Each tenant's vmagent writes data to its own space.  Checkout the yaml:
     ```
     - url: "http://vminsert-vmcluster.observability.svc.cluster.local:8480/insert/1/prometheus/api/v1/write"
     ```
   - Apply
    ```
    $ kubectl apply -f 4-3-multi-tenant-isolation/vmagent-tenant-1.yaml
    $ kubectl apply -f 4-3-multi-tenant-isolation/vmagent-tenant-2.yaml
    $ kubectl get pod -n observability | grep vmagent
    vmagent-default-vmagent-f4c75dc7-n2bpb      2/2     Running            0              22m
    vmagent-vmagent-1-5464876db8-8mq7t          2/2     Running            0              2m30s
    vmagent-vmagent-2-b99c94cbb-cjd6v           2/2     Running            0              2m25s

    # Let's confirm that vmcluster collects data from different tenants
   
    $ kubectl port-forward svc/vmselect-vmcluster -n observability 8481:8481 
   
    #  http://127.0.0.1:8481/select/1/vmui/?#/cardinality
    #  http://127.0.0.1:8481/select/2/vmui/?#/cardinality
    ```

   - Let's deploy a dedicated VMAlert for each tenant, they have different configurations same as vmagent.
    ```
    $ kubectl apply -f 4-3-multi-tenant-isolation/vmalert-tenant-1.yaml
    $ kubectl apply -f 4-3-multi-tenant-isolation/vmalert-tenant-2.yaml
    $ kubectl get pod -n observability | grep vmalert
    vmalert-default-vmalert-79566d98d6-2q5rb    2/2     Running   0     19m
    vmalert-vmalert-tenant-1-5dcc49556d-rqwcc   2/2     Running   0     2m5s
    vmalert-vmalert-tenant-2-5f65f87f94-nq4rf   2/2     Running   0     2m1s
    ```

4. Provides disaster recovery for possible failures.
   Enable the temporary storage function of VMAgent.
   ```
    statefulMode: true
    statefulStorage:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 2Gi
   $ kubectl apply -f 4-4-disaster-recovery/vmagent-tenant-1.yaml
   ```

   Now we set the replicaCount of VMInsert to 0 to simulate a failure of the vmcluster service
   ```
   $ kubectl apply -f 4-4-disaster-recovery/vmcluster.yaml
   
   # Wait at least 5 minutes
   
   # There's no recent metrics data

   $ kubectl port-forward svc/vmselect-vmcluster -n observability 8481:8481
   # http://127.0.0.1:8481/select/1/vmui/#/?g0.expr=node_load5&g0.range_input=15m&g0.end_input=2023-07-20T15%3A22%3A00&g0.relative_time=last_15_minutes&g0.tenantID=1%3A0
   ```

   Wait another 5 minutes
   Now, let's recover the VMInsert service with 4-2-high-availability-deployment/vmcluster.yaml

   ```
   $ kubectl apply -f 4-2-high-availability-deployment/vmcluster.yaml
   
   # Wait for a while so that VMAgent can upload the cached data to VMCluster.
   ...
   # OK. Let's check the data difference between tenant-1 and tenant-2
   $ kubectl port-forward svc/vmselect-vmcluster -n observability 8481:8481 
   
   # http://127.0.0.1:8481/select/2/vmui/#/?g0.expr=node_load5&g0.range_input=15m&g0.end_input=2023-07-20T15%3A23%3A31&g0.relative_time=last_15_minutes&g0.tenantID=2%3A0
   # We will see that part of the data is lost, while in tenant-1, no data is lost
   # http://127.0.0.1:8481/select/1/vmui/#/?g0.expr=node_load5&g0.range_input=15m&g0.end_input=2023-07-20T15%3A22%3A00&g0.relative_time=last_15_minutes&g0.tenantID=1%3A0
   ```
