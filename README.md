# Requirements

- Deploy Sensu as a service in Kubernetes
- Deploy Nginx as a service in Kubernetes
- Setup sensu checks to check for memory, disk, and cpu of the 3 Kubernetes nodes, and a check to ensure Nginx is online.
- Setup Uchiwa as a frontend for Sensu
- Demonstrate taking Nginx down and Sensu triggering an alert

# Instructions

1. Run `vagrant up` from project source.  The script will provision 3 nodes (`master`,`node20`,`node21`)
2. Access Uchiwa from host by navigating to `http://192.168.56.10:3000`
3. SSH into master using `vagrant ssh master`
4. Scale Nginx instance to 0 using `kubectl scale --replicas=0 deployments/nginx`
5. Check Uchiwa dashboard.  Eventually the `nginxcheck` client will show a warning 
6. Scale Nginx instance to 1 using `kubectl scale --replicas=1 deployments/nginx`

NOTE: The Sensu server will not be accessible until Kubernetes nodes are provisioned and the containers have been created.  Until all pods are up and running, Uchiwa will report the Datacenter as unavailable and will include no registered clients.

# Troubleshooting

If Uchiwa reports that the datacenter is not available even after all pods have been successfully created, restart the API service and Server by executing the following on the master node (access the master node via `vagrant ssh master`):

```
kubectl scale --replicas=0 deployment/api
kubectl scale --replicas=1 deployment/api
kubectl scale --replicas=0 deployment/server
kubectl scale --replicas=1 deployment/server
```