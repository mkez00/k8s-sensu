# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.define "master" do |c|
		c.vm.hostname = 'master'
		c.vm.box = "ubuntu/xenial64"
		c.vm.synced_folder "data", "/data"
		c.vm.network "private_network", ip: "192.168.56.10"
		c.vm.provision "shell", inline: <<-SHELL
			sudo su

			# Need to ensure that hostname -i returns a routable IP address.  See: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
			sed 's/127\.0\.0\.1.*master.*/192\.168\.56\.10 master/' -i /etc/hosts

			apt-get update 
			apt-get install -y apt-transport-https
			curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
			cat <<EOF > /etc/apt/sources.list.d/kubernetes.list  
deb http://apt.kubernetes.io/ kubernetes-xenial main  
EOF
			apt-get update
			apt-get install -y docker.io
			apt-get install -y kubelet kubeadm kubectl kubernetes-cni

			# # need to avertise API server address on interface that is reachable for other nodes.  Storing join command from output in shell script for use by worker nodes
			kubeadm init --apiserver-advertise-address=192.168.56.10 --token=b9e6bb.6746bcc9f8ef8267 --pod-network-cidr=10.244.0.0/16
			kubeadm token create --print-join-command | grep "kubeadm join" > /data/join-cluster.sh

			# setup kubectl for root
      		mkdir -p /root/.kube
      		cp -i /etc/kubernetes/admin.conf /root/.kube/config

      		sysctl net.bridge.bridge-nf-call-iptables=1

      		# using slightly modified flannel config since it requires using enp0s8 as default interface.  See: https://github.com/kubernetes/kubeadm/issues/139#issuecomment-276607463
      		kubectl apply -f /data/kube-flannel.yml

      		# Download default postgres image and run
			kubectl run postgres --image=docker.io/postgres:latest --port=5432
			kubectl expose deployment postgres --name=postgres

			# Download default redis image and expose service to cluster
			kubectl run redis --image=docker.io/redis:latest --port=6379
			kubectl expose deployment redis --name=redis


			kubectl run rabbitmq --image=docker.io/rabbitmq:latest --port=5672
			kubectl expose deployment rabbitmq --name=rabbitmq



			# create new deployment for sensu server
			kubectl run server --image=docker.io/sstarcher/sensu:latest --env="SENSU_SERVICE=server" --env="TRANSPORT_NAME=rabbitmq"
			# create new deployment for sensu api
			kubectl run api --image=docker.io/sstarcher/sensu:latest --env="SENSU_SERVICE=api" --env="TRANSPORT_NAME=rabbitmq" --port=4567
			# create new deployment for a sensu test client
			# kubectl run client --image=docker.io/sstarcher/sensu:latest --env="SENSU_SERVICE=client" --env="CLIENT_NAME=test" --env="CLIENT_SUBSCRIPTIONS=all" --env="CLIENT_ADDRESS=127.0.0.1" --port=3030

			# expose api as service
			kubectl expose deployment api --name=api

			# Install sensu client on master node
			# sensu client
			wget -q https://sensu.global.ssl.fastly.net/apt/pubkey.gpg -O- | sudo apt-key add -
			echo "deb https://sensu.global.ssl.fastly.net/apt xenial main" | sudo tee /etc/apt/sources.list.d/sensu.list
			apt-get update
			apt-get install sensu
			# apt install sensu=1.2.0-1

			# Install uchiwa on master node
			echo "deb https://sensu.global.ssl.fastly.net/apt sensu main" | sudo tee /etc/apt/sources.list.d/uchiwa.list
			wget -O- https://sensu.global.ssl.fastly.net/apt/pubkey.gpg |  sudo apt-key add -
			apt-get update
			apt-get -y install uchiwa

			# configure uchiwa to point at Sensu API and start
			cp /data/uchiwa.json /etc/sensu/uchiwa.json
			apistring=$(kubectl get svc | grep api)
			api=$(echo "$apistring" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')
			sed -i -e 's/API_IP/'"$api"'/g' /etc/sensu/uchiwa.json

			service uchiwa start

			# copy config files for Sensu client
			cp /data/transport.json /etc/sensu/conf.d/transport.json

			# rabbitmq.json will need updating with Cluster IP from service
			cp /data/rabbitmq-sample.json /data/rabbitmq.json
			rabbitmqstring==$(kubectl get svc | grep rabbitmq)
			rabbitmq=$(echo "$rabbitmqstring" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')
			sed -i -e 's/CLUSTER_IP/'"$rabbitmq"'/g' /data/rabbitmq.json
			cp /data/rabbitmq.json /etc/sensu/conf.d/rabbitmq.json

			# copy checks for sensu
			cp /data/check-* /etc/sensu/conf.d/

			cd /opt/sensu/embedded/bin
			sensu-install -p cpu-checks  
			sensu-install -p disk-checks  
			sensu-install -p memory-checks  

			service sensu-client start

		SHELL
	end

	$node_script = <<-SHELL
		sudo su

		# Need to ensure that hostname -i returns a routable IP address.  See: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
		sed "s/127\.0\.0\.1.*node${1}.*/192\.168\.56\.${1} node${1}/" -i /etc/hosts

		apt-get update 
		apt-get install -y apt-transport-https
		curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
		cat <<EOF > /etc/apt/sources.list.d/kubernetes.list  
deb http://apt.kubernetes.io/ kubernetes-xenial main  
EOF
		apt-get update
		apt-get install -y docker.io
		apt-get install -y kubelet kubeadm kubectl kubernetes-cni

		sysctl net.bridge.bridge-nf-call-iptables=1

		# Join node to cluster
		sh /data/join-cluster.sh

		# Install Sensu client on node
		wget -q https://sensu.global.ssl.fastly.net/apt/pubkey.gpg -O- | sudo apt-key add -
		echo "deb https://sensu.global.ssl.fastly.net/apt xenial main" | sudo tee /etc/apt/sources.list.d/sensu.list
		apt-get update
		apt-get install sensu

		# copy config files for Sensu client
		cp /data/transport.json /etc/sensu/conf.d/transport.json

		# redis.json will need updating with Cluster IP from service
		# cp /data/redis.json /etc/sensu/conf.d/redis.json

		# rabbitmq will need updating with Cluster IP from service
		cp /data/rabbitmq.json /etc/sensu/conf.d/rabbitmq.json

		# copy checks for sensu
		cp /data/check-* /etc/sensu/conf.d/

		cd /opt/sensu/embedded/bin
		sensu-install -p cpu-checks  
		sensu-install -p disk-checks  
		sensu-install -p memory-checks 

		service sensu-client start

	SHELL

	$num_node = 2
	$num_node.times do |n|
		node_vm_name = "node#{n+20}"
		hostname = "node#{n+20}"
		private_ip = "192.168.56.#{n+20}"
		node = "#{n+20}"

		config.vm.define node_vm_name do |c|
			c.vm.hostname = hostname
			c.vm.box = "ubuntu/xenial64"
			c.vm.synced_folder "data", "/data"
			c.vm.network "private_network", ip: private_ip
			c.vm.provision "shell", run: "always" do |s|
				s.inline = $node_script
				s.args = node
			end
		end
	end

end