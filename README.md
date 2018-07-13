# Requirements

- Deploy Sensu as a service in Kubernetes
- Deploy Postgresql as a service in Kubernetes
- Setup sensu checks to check for memory, disk, and cpu of the 3 Kubernetes nodes, and a check to ensure Postgresql is running/listening.
- Setup Uchiwa as a frontend for Sensu
- Demonstrate taking Postgresql down and Sensu triggering an alert

# Instructions

1. Run `vagrant up` from project source.  The script will provision 3 nodes (`master`,`node20`,`node21`)
2. After provisioning is completed, SSH into master node with `vagrant ssh master`
3. Get Cluster IP addresses for Redis and and Api (Sensu) services by entering the command `kubectl get svc`
4. Configure Sensu Client on each node by modifying `/etc/sensu/conf.d/redis.json` and updating the `host` attribute with the Cluster IP address given for the Redis service
5. Start Sensu Client on each node by executing `service sensu-client start`
6. Configure Uchiwa's Sensu connection by modifying `/etc/sensu/uchiwa.json` and updating the `host` attribute in the `sensu` object to be the Cluster IP for the Sensu service
7. Restart Uchiwa by executing `service uchiwa restart` 
8. Access Uchiwa from host by navigating to `http://192.168.56.10:3000`

# Troubleshooting

## No Client Connections

Try registering a client using the Sensu API.  Determine the Cluster IP for the API service by executing the command `kubectl get svc`.  Try executing the following curl command:

```
curl -s -i \
-X POST \
-H 'Content-Type: application/json' \
-d '{"name": "api-example","address": "10.0.2.100","subscriptions":["default"],"environment":"production"}' \
http://<CLUSTER_IP>:4567/clients
```

A 201 HTTP response should be returned.  Refresh the Uchiwa dashboard, the badge for clients will be updated with the newly added client.