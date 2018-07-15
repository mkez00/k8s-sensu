# Requirements

- Deploy Sensu as a service in Kubernetes
- Deploy Postgresql as a service in Kubernetes
- Setup sensu checks to check for memory, disk, and cpu of the 3 Kubernetes nodes, and a check to ensure Postgresql is running/listening.
- Setup Uchiwa as a frontend for Sensu
- Demonstrate taking Postgresql down and Sensu triggering an alert

# Instructions

1. Run `vagrant up` from project source.  The script will provision 3 nodes (`master`,`node20`,`node21`)
2. Access Uchiwa from host by navigating to `http://192.168.56.10:3000`
3. SSH into master using `vagrant ssh master`
4. Scale Postgres instance to 0 using `kubectl scale --replicas=0 deployments/postgres`
5. Check Uchiwa dashboard.  Eventually the `postgrescheck` client will show a warning 
6. Scale Postgres instance to 1 using `kubectl scale --replicas=1 deployments/postgres`

NOTE: The Sensu server will not be accessible until Kubernetes nodes are provisioned and the containers have been created.  Until all pods are up and running, Uchiwa will report the Datacenter as unavailable and will include no registered clients.

# Troubleshooting

If Uchiwa reports that the datacenter is not available even after all pods have been successfully created, restart the API service by executing the following on the master node (access the master node via `vagrant ssh master`):

```
kubectl scale --replicas=0 deployment/api
kubectl scale --replicas=1 deployment/api
```

# Issues

Currently the Postgres plugin is not compiling with the setup due to a Ruby issue:

```
Installing 	 sensu-plugins/postgres:master
ERROR:  Error installing sensu-plugins-postgres-1.4.6.gem:
	ERROR: Failed to build gem native extension.

    current directory: /usr/local/bundle/gems/pg-0.18.3/ext
/opt/sensu/embedded/bin/ruby -r ./siteconf20180715-918-1uc88jb.rb extconf.rb
checking for pg_config... no
No pg_config... trying anyway. If building fails, please try again with
 --with-pg-config=/path/to/pg_config
checking for libpq-fe.h... no
Can't find the 'libpq-fe.h header
*** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.
```