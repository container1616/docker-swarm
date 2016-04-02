# docker-swarm
Create three nodes with your favourite cloud provider e.g. node1, node2 and node3. 

node1  : Running the consul ( discovery backend) 
	
	docker run -d -p "8500:8500" -h "consul" progrium/consul -server -bootstrap
	
node2  : Running the Swarm Manager and client
	
	sudo service docker stop
	docker daemon -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --cluster-store=consul://<<node1-ip>>:8500 --cluster-advertise=<<node2-ip>>:2375 &
	sudo docker run -d -p 3000:2375 swarm manage  consul://<<node1-ip>>:8500
	sudo docker run -d swarm join --addr=<<node2-ip>>:2375  consul://<<node1-ip>>:8500
	
node3  : Running the Swarm client 
	
	sudo service docker stop
	docker daemon -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --cluster-store=consul://<<node1-ip>>:8500 --cluster-advertise=<<node3-ip>>:2375 &
	sudo docker run -d swarm join --addr=<<node3-ip>>:2375 consul://<<node1-ip>>:8500
	
Create overlay network : 	
        
        docker -H tcp://<<node2-ip>>:3000 network create --driver overlay --subnet=10.0.9.0/24 my-net	 
        docker -H tcp://<<node2-ip>>:3000 network ls (overlay network should be visible)	

Running the application (download docker-compose.yaml) :

	export DOCKER_HOST="tcp://<<node2-ip>>:3000"
  	docker-compose scale spring=5

References : 
        https://docs.docker.com/engine/userguide/networking/get-started-overlay/
        https://docs.docker.com/swarm/install-manual/
        https://docs.docker.com/swarm/install-w-machine/
 
