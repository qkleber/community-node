This node is starting off in the Linode cloud to alleviate the vulnerability of having too many nodes hosted on AWS, 
eventually migrating to a Rackspace managed OnMetal server. No additional VM software or containerization overhead needs 
to be added since this is a dedicated server.

Next build out will split off http and add exim mail & dns services to a small slice nanode VM, and also provide a testable node for community members.


# CURRENT CONFIGURATION
Debian GNU/Linux 9.1 (stretch)
4.15.12-x86_64-linode105

oracle-java8
Version: 8u161-1~webupd8~0

PORTS OPEN:
80/tcp
8088/tcp
18888/tcp
18888/udp
50051/tcp
50051/udp
*****/tcp (ssh)

The node will be configured according to the instructions at my [Witness Node for Newbies guide](https://www.reddit.com/r/tronsupport/comments/8beglx/witness_node_for_newbies)



I will start hosting the project at [my private server](http://beans.bondibox.com) and upon token creation buy an appropriately named domain. The website will begin as a node.js API because this will have the smallest footprint, it's fast, and will be the easiest to turn into a TRON main net dapp. I'd like to redirect ERROR level log warnings to the website to provide a System Status page.
