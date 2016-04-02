# docker-swarm
Create three nodes with your favourite cloud provider e.g. node1, node2 and node3. Please choose docker pre-installed image, if cloud provider offers that. Otherwise install docker manually. 

#node1  : Running the consul ( discovery backend)
	SSH into the host and run the following command, to start the consul service. 
	
	docker run -d -p "8500:8500" -h "consul" progrium/consul -server -bootstrap
	
#node2  : Running the Swarm Manager and client
	SSH into the host and run the following commands. It would do the following, stop the running docker daemon, start the docker daemon which is listening on TCP port to enable swarm communication, run the swarm manager and run the swarm client. 
	
	sudo service docker stop
	docker daemon -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --cluster-store=consul://<<node1-ip>>:8500 --cluster-advertise=<<node2-ip>>:2375 &
	sudo docker run -d -p 3000:2375 swarm manage  consul://<<node1-ip>>:8500
	sudo docker run -d swarm join --addr=<<node2-ip>>:2375  consul://<<node1-ip>>:8500
	
#node3  : Running the Swarm client 
	SSH into the host and run the following command. It would do the following, stop the running docker daemon, start the docker daemon which is listening on TCP port to enable swarm communication and run the swarm client. 
	
	sudo service docker stop
	docker daemon -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --cluster-store=consul://<<node1-ip>>:8500 --cluster-advertise=<<node3-ip>>:2375 &
	sudo docker run -d swarm join --addr=<<node3-ip>>:2375 consul://<<node1-ip>>:8500
	
#Create overlay network : 	
        
        docker -H tcp://<<node2-ip>>:3000 network create --driver overlay --subnet=10.0.9.0/24 my-net	 
        docker -H tcp://<<node2-ip>>:3000 network ls (overlay network should be visible)	

#Running the application : 

 using docker-compose (download docker-compose.yaml) 
	SSH into any machine (node2 or 3) and run the following commands. install docker compose (https://docs.docker.com/compose/install/), if not already in that machine. 

	export DOCKER_HOST="tcp://<<node2-ip>>:3000"
  	docker-compose scale spring=5

Please note docker compose is not mandatory for docker swarm to work, simple docker run would also work.

	docker run -d container1616/gs-scheduling-tasks

To see the created containers run the following command
	
	docker -H tcp://<<node2-ip>>:3000 ps

#References : 
        https://docs.docker.com/engine/userguide/networking/get-started-overlay/
        https://docs.docker.com/swarm/install-manual/
        https://docs.docker.com/swarm/install-w-machine/
 
