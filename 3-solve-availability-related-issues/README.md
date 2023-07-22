So far, we have built a set of monitoring and alert service.
Now we check whether the service has sufficient availability, and then think about the solution.

1. VMStorage is an important service. It needs enough memory to run, but the vmoperator deployed on the same EC2 often takes up too much memory, which reduces the performance of VMStorage. How do we ensure the availability of VMStorage?

2. How do we provide 99.99% service if VMStorage fails every 990 minutes and takes 10 minutes to reboot to restore service? How do we ensure that VMStorage services are still available when any EC2 goes down？

3. [VM supports multiple isolated tenants identified by accountID](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy), but for metrics collection, we don't want any impact between different tenants, what should we do?

4. If all VMStorage services crash for 5 minutes, we lose 5 minutes of availability. But what about after service is recovered? Can we view the data for the 5 minutes of the failure

