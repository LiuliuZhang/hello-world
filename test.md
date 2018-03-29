# Schönhofer Document Tracking POC

## Technical requirements
**************
- For Hyperledger:
	- Please refer [Hyperledger Dokumentation](https://hyperledger-fabric.readthedocs.io/en/release/prereqs.html), especially Docker and the Go environment
	
- latest Hyperledger Binaires and Docker images, loud [source](https://hyperledger-fabric.readthedocs.io/en/release/samples.html#binaries) to refer to 
	> curl -sSL https://goo.gl/byy2Qj | bash -s 1.0.5
	
- the files from the [Repository](https://github.com/multimedial/Hyperledger)

- mysql-Docker-Image, available per 
	> docker pull mysql/mysql-server
	
- node.js and npm for the [Blockchain Explorer](https://github.com/hyperledger/blockchain-explorer#requirements) (The Explorer is already part of the local repository, it does not need to be reloaded)


****************************************
## Steps to build the demo network: 
The process is:
- Raising the infrastructure
- Preparation and execution of the chain code
- Visualization by the blockchain viewer (optional)
- Execution of calls*

**step 1: Raising the infrastructure**

Change to the directory "Hyperledger/Network/Schoenhofer" of the repository and start the Docker container via shell script:

	cd Hyperledger/Network/Schoenhofer
	
	./start.sh
	
This starts the required Docker container and thus creates the required infrastructure. It will be created: 

* three peer nodes (each representing an office or local office)
* three Certification Authorities (CA) (one for each office),
* an orderer (distributes and orders the requests in the network), 
* a CouchDB container to manage the WorldState for the peer nodes.
* a MySQL container for the optional blockchain viewer (see "Insert").

To the controller via docker ps notice that all containers were started.

	Inset: MySQL container for the optional blockchain viewer

	The MySql container for the optional blockchain viewer has to start manually at the moment and the required database with the [fabricexplorer.sql](https://github.com/multimedial/Hyperledger/blob/master/Network/db/fabricexplorer.sql) File to be configured. This creates the required tables. Starting the MySQL Server Docker image with root password "123456" and allowing the DB root user to log in to the database from external hosts as well:
		
			docker run -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_ROOT_HOST=% -p 3306:3306 --name mysql mysql/mysql-server

	In another terminal then log into the mysql client:
		
			docker exec -it mysql mysql -u root -p

	You will be prompted to enter the previously defined root password "123456", then paste (copy-paste) the contents of["Hyperledger/Network/db/fabricexplorer.sql"](https://github.com/multimedial/Hyperledger/blob/master/Network/db/fabricexplorer.sql), so that the required database tables are created.
	
	TODO: 
	Automating or initializing the Docker container with the SQL file.
	

**step 2: Preparation and execution of the chain code**

Switch to the CLI container of the blockchain infrastructure:

	docker exec -it cli bash

Make sure to be in the directory "/opt/gopath/src/docutracker" :
	
	cd /opt/gopath/src/docutracker
	
Then execute:

	./buildandinstall.sh
	
The chain code is then started, the last line in the terminal should be 

	" [...] starting up ... "

Leave this terminal open for inspection. Log in to the CLI container in a new terminal:

	docker exec -it cli bash
	
instantiate the chaincode via shell script and start with:

	cd /opt/gopath/src/docutracker
	
	./startcode.sh
	
Result should be without error, and in the previous, left open terminal should now be:

	...
	"#### Smartcontract struct initialized #####"

**step 3: Visualization by the blockchain viewer**

**ACHTUNG**: since this is a separate project, it must be built once before the first call with:

	cd Hyperledger/Network
	npm install

Once the blockchain viewer has been created, it can be started by shell script:

	./monitor.sh
	
Now it should be possible to call http://localhost:8080 in the browser. There should then be the blockchain viewer with the peers and at least one block in the chain.


**Demo of the chaincode**

Switch to the CLI container:

	docker exec -it cli bash

and then execute:

	./demo.sh
	
This fills the blockchain with transactions and objects (workplaces, users, documents). They are created in the following order: 

- three places of work ("workplace1", "workplace2", "workplace3")
- 9 User
- 9 documents with different security levels

The 9 users are assigned to the workplaces, three users per workplace. These operations should all be seen as transactions in the browser (block numbers and transactions):

![Blockchain-Viewer](https://raw.githubusercontent.com/multimedial/Hyperledger/master/Network/BlockchainViewer.jpg "Blockchain-Viewer")
