# Requirements

- Deploy Sensu as a service in Kubernetes
- Deploy Postgresql as a service in Kubernetes
- Setup sensu checks to check for memory, disk, and cpu of the 3 Kubernetes nodes, and a check to ensure Postgresql is running/listening.
- Setup Uchiwa as a frontend for Sensu
- Demonstrate taking Postgresql down and Sensu triggering an alert

# Instructions

1. Run `vagrant up` from project source.  The script will provision 3 nodes (`master`,`node20`,`node21`)
2. Access Uchiwa from host by navigating to `http://192.168.56.10:3000`

NOTE: The Sensu server will not be accessible until Kubernetes nodes are provisioned and the containers have been created.  Until all pods are up and running, Uchiwa will report the Datacenter as unavailable and will include no registered clients.

# Troubleshooting

If Uchiwa reports that the datacenter is not available even after all pods have been successfully created, restart the API service by executing the following on the master node `vagrant ssh master`:

```
kubectl scale --replicas=0 deployment/api
kubectl scale --replicas=1 deployment/api
```