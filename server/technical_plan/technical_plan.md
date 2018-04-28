This node is starting off in the Linode cloud to alleviate the vulnerability of having too many nodes hosted on AWS, possibly migrating to a Rackspace managed OnMetal server. No additional VM software or containerization overhead needs to be added since this is a dedicated server.

Next build out will split off a slave server to handle solidity, http, https, and add exim to a small slice nanode VM, and also provide a testable node for community members.

For Production mode a private network will sit behind a public IP running an NGINX server and load balancer will serve as the network gateway. It will pass through p2p requests on port 18888 to the full node which will produce blocks. It will also proxy grpc wallet requests to several smaller slaves which will run solidity seed nodes. 

The website will begin as a node.js API because this will have the smallest footprint, it's fast, and will be the easiest to turn into a TRON main net dapp. It may turn into a puma-served Ruby on Rails website for its ease of development.

Please refer to individual build out subdirectories for specific server and network configuration information.

