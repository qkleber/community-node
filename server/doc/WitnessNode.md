To the best of my knowledge, this is the way to set up a TRON witness node. This is a living document and it is updated frequently.

Most of us will be setting up our nodes in the cloud, on a VM slice. There's a video for getting started on [Google Cloud](https://www.youtube.com/watch?v=AN9YwX7PqgY) or you can use an indie host like Rackspace or Linode. Rackspace is rock but be careful about over the limit bandwidth charges. 

**STEP #1 Register an address**

Go to [tronscan.org](https://tronscan.org/#/) and register. Clicking the red bar at the top of the page will create a new test wallet address, password and private key. Copy and save them to a txt file. You can then use the password to login and request 1,000,000 test TRX to be sent to your wallet address. If you want to run the wallet-web app it will now show you the same balance, but you can only request tokens from the tronscan.org server. The test net has a good amount of functionality now, you can use those test TRX to vote for delegates, to create tokens, and to send to other TRX test addresses.

**STEP #2 Install Java 8**

Even though there is a Java 9 and Java 10 available, use JDK 8. You'll need several GB of free space to unpack java, and even more free space so the cache buildup doesn't crash the server every day. 

For a **Windows** install you can follow [these instructions](https://docs.google.com/document/d/1RsRx_NrWsySP6EthUzYtdEGkAWg7j29UTHOxRhFgfVY) to run the INTELLIJ app which will take you up to the "this is just a node" line in this document.

For **Mac O/S**, see if you have java installed. Go to Applications/Utilities/Terminal. You'll be needing it later but for now learn a neat Mac trick. Type:
    
    open /Library/Java/JavaVirtualMachines
In case you didn't notice, it opened that folder in the finder. If you have anything other than a jdk1.8.0_172.jdk folder there, either delete it (them) or move to the desktop until you're done with this project. Get and install the [jdk-8u172-macosx-x64.dmg](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

For **Debian or Ubuntu**, After the initial o/s install, you want to update with any new patches

    sudo apt-get update
    sudo apt-get upgrade
Add the java repository to the apt-cache

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update

We'll also install needed packages & helpful tools

    apt-get -y -V install nodejs build-essential git git-core locate curl libcurl4-openssl-dev wget javascript-common libjs-jquery unzip zip unattended-upgrades libcap2-bin oracle-java8-installer

Then, for the love of everything holy, set security updates to automatically install.

    echo 'APT::Periodic::Update-Package-Lists "1";' >> /etc/apt/apt.conf.d/20auto-upgrades
	echo 'APT::Periodic::Unattended-Upgrade "1";' >> /etc/apt/apt.conf.d/20auto-upgrades

The node.js that comes with the standard debian package is insufficient. Remove it, and then download the full package.

	sudo apt-get remove nodejs
	sudo apt-get remove npm	
	cd /tmp
	curl -sL https://deb.nodesource.com/setup_6.x | sudo bash -
	sudo apt-get install -y nodejs
	sudo apt-get install -y npm

Install the grpc because I think it's needed for protobuf

	sudo npm install grpc


**THE REST OF THIS DOC IS THE SAME FOR ALL O/S** Although it's written for linux users. If you're on a Mac you ought to be able to figure out how to copy, edit and delete files.

**STEP #3 Install Java-Tron**

From this point on, you should not need to run any commands as 'sudo'.

Clone the tron repository to the local machine, then change into the tron directory and build the machine

    git clone https://github.com/tronprotocol/java-tron.git
    cd java-tron
    ./gradlew build
This will create a folder named 'build'. There are 3 ways to launch the java-tron machine. The first and second are for all intents and purposes identical. The third uses a GUI program. I like #1 best:

    ./gradlew run

This is just a node. The next step is to set up a witness node using the information you got when you registered at tronscan in step 1. You can choose to copy the original config.conf file in java-tron/src/main/resources to the root of the application and edit that. This way you don't end up changing any of the repository files and you don't have to do any git trickery to do another pull.

    cp src/main/resources/config.conf .
    nano config.conf    
There is only one key piece of information to enter here - the private key in the localwitnesses block. However if you don't specify your IP address it will have to probe for it on startup and that takes a few seconds, so I like to hardcode the IP information. Google cloud users will have two IP addresses. It may work best by configuring separate IP's like this. Use actual IP addresses. I have both of those configured for my one public IP even though I also have an internal subnet IP. 

**config.conf changes**

    node.discovery = {
      enable = true
      persist = true
      bind.ip = "INTERNAL.IP.ADDRESS"
      external.ip = "EXTERNAL.IP.ADDRESS"
    }

--------------------------

	localwitness = [
	ABCDEF1234567890ABCDEF1234567890ABCDEF1234567890ABCDEF1234567890
	]




O.K. now when the tron machine starts up it's not just running a node, it's running *your* node.

If you've made any changes when you restart ~/java-tron, you'll need to throw away the database directory. While you're at it, start the logs fresh.

    cd ~/java-tron
    rm -Rf output-directory/database
    rm -Rf logs/*
    ./gradlew run -Pwitness=true

The TRON Developers recommend starting up a different way. They say don't put your private key in the config file, call it as a parameter when you're starting the application, like this:

    cd build/libs
    java -jar java-tron.jar -p private_key --witness -c config_path

**Example:**

    java -jar java-tron.jar -p 650950B193DDDDB35B6E48912DD28F7AB0E7140C1BFDEFD493348F02295BD812 --witness -c /data/java-tron/config.conf

.  


**SKIP TO THE END OF THIS DOCUMENT TO SEE EXAMPLE LOG FILES**

.  

**STEP 4 Install Command Line Wallet**

Technically the wallet doesn't work yet, but we're one step closer with this latest build. Unless you installed screen as per the Google Cloud video, you'll need to open a second terminal window to work while the java-tron is running.

    git clone https://github.com/tronprotocol/wallet-cli.git
    cd wallet-cli
    ./gradlew build       
    cp src/main/resources/config.conf .
	nano config.conf
Point your wallet-cli to the new witness node if you want by changing the first block of the wallet-cli/config.conf file

	fullnode = {
	  ip.list = [
	    "12.34.56.78:50051"
	  ]
	}
12.34.56.78 is just an example. Use your IP address of the java-tron witness node. N.B. even though the server may be running on port 18888 you should set this port to 50051. This is the proper location for the config file and it's one less error generated when your start up.

    ./gradlew run -Pcmd
After it gets to 90% you will have an active cursor. 

At this point it's a bit of a best guess, since the test net isn't fully connected. When I run the blockchain explorer I don't see any nodes or any blocks, so it's not surprising that the login doesn't work. Even still, I'm not 100% sure how it's supposed to work. I believe you ought to be able to just login with the password you got when you registered at tronscan.org 

    login YOUR_PASSWORD_HERE
But that has yet to work for me. It shouldn't need a network connection to login. The address is generated from the password when the wallet was originally registered. TRON Docs say this about the RegisterWallet Command:

> *RegisterWallet* Register a wallet in local. Generate a pair of ecc keys. Derive a AES Key by password and then use the AES algorithm to encrypt and save the private key. The account address is calculated by the public key sha3-256, and taking the last 20 bytes. All subsequent operations that require the use of a private key must enter the password.

I kind of got lost in this explanation but does this mean the password will generate a different address on every machine running the wallet? Because that's the behavior right now. That can't be right. You have to be able to print up this stuff to make a cold wallet transferable to any computer. If all I have is my password, address and my private key printed out and my wallet gets destroyed I should be able to use that password to log in with a different wallet and still be connected to the same address.

TRON devs posted an update:

> "The way addresses are generated changed a few times and it is gonna be changed (again) to base58 format soon."

That explains why my wallet isn't sharing info with the blockchain, but it doesn't explain why two different computers running the same version of the wallet come up with different addresses.

Logically this should be the same address and private key but like the devs said things are changing fast. It doesn't make sense either that you need to register this wallet. But I think once my blockchain explorer can see the nodes and blocks my wallet will be able to log in without the need to register the wallet. It's either that, or the wallet is creating an **actual wallet** for cold storage. 

    RegisterWallet YOUR_PASSWORD_HERE
    login YOUR_PASSWORD_HERE
    getAddress
    BackupWallet YOUR_PASSWORD_HERE
You'll want to back up everything (None of any of this is going to be valid on main.net, but get in the habit). This doesn't actually back anything up, just prints your address and private key that you need to save. Ideally this should be the same as what you saw at tronscan.org. I know that it's not sharing the same information as the rest of the blockchain, because when logged in to Tronscan.org I can send TRX to the address created in wallet-cli, and it says success but my wallet-cli balance doesn't change.

    getbalance
    listAccounts
    listNodes
I usually get about 190 nodes. ListAccounts only shows the ones listed in my java-tron config file, the tronscan.org blockchain explorer shows new ones coming online often.

When the wallet starts up, it knows where to get its blockchain data.  

In the wallet-cli terminal there are a lot of useful commands. Type ? for a list. There's no point in walking you through most of them since we don't have a balance sync'd up. The command everyone wants to use is assetIssue, but unfortunately it's got a bug. If you don't enter 10 elements in the array, it throws an error. But the underlying code is expecting 11 elements before it can create the object. I tried haxoring the code to make it expect 11 elements, but no go. They forgot VoteScore in their example doc.

    AssetIssue Password AssetName TotalSupply TrxNum AssetNum StartDate EndDate DecayRatio VoteScore Description Url

Eventually the command will look like this:

    assetIssue YOUR_PASSWORD_HERE Test1 10000000000000000 1 100 2018-4-1 2018-4-30 1 5 just-test https://github.com/tronprotocol/wallet-cli/

**BLOCKCHAIN EXPLORER**
If you go to https://tronscan.io/ you can access a web wallet.  An identical page has been set up at https://tronscan.org/#/

There are two ways for you to run one. The first is

    cd ~/wallet-cli/build/libs
    java -jar wallet-1.0-SNAPSHOT.jar
then directing your browser to http://serveripaddress:8088

The standalone version of this app is wallet-web. In order to install wallet-web you need Yarn. I needed to upgrade to yarn 1.5.1 because 1.3 didn't work.  Yarn 1.3 is actually a symlink from cmdtest, so this is what my workflow looked like

    sudo apt-get remove cmdtest libuv1
    sudo apt-get remove yarn
    sudo npm install -g yarn
    git clone https://github.com/tronprotocol/wallet-web
    cd wallet-web
    sudo yarn install
    sudo yarn start
then directing your browser to http://serveripaddress:3000
If you try to install wallet-web without the right version of Yarn installed you will bork the install because the installer will already have staged the files incorrectly. To fix this you have to delete the contents of node-modules

    sudo rm -Rf ~/wallet-web/node-modules/*
then run 'yarn install' again

However, when I run the explorer it doesn't show any blocks like the other sites do.


**LOG FILE EXAMPLES**

The first lines of logs/tron.log should be 

    Full node running.
    Refreshing org.springframework.context.annotation.AnnotationConfigApplicationCote [Thu Apr 26 18:47:17 EDT 2018]; root of context hierarchy
    Transaction create succeeded！

    Unknown channel option 'SO_KEEPALIVE' for channel '[id: 0x9ec67546]' 
    Reading Node statistics from PeersStore: 0 nodes. 
    Add new node: NodeHandler[state: Discovered, node:  
    Change node: old NodeHandler[state: Discovered, node:  

Then it will scroll through adding nodes from the network

    Add new node: NodeHandler[state: Discovered, node: 93.35.144.199:3484, id=609cd88e], size=19
    Add new node: NodeHandler[state: Discovered, node: 109.27.36.86:50196, id=61c5b1bc], size=20

Finally it looks like you've got something, only to be disappointed in the details:

    Peer stats:
    Active peers
    Other connected peers
    
    LocalNode stats:
    MyHeadBlockNum: 0
    advToSpreadNum: 0
    advObjectToFetchNum: 0
    advObjWeRequestedNum: 0
    unSyncNum: 0
    blockWaitToProcess: 0
    blockJustReceived: 0
    syncBlockIdWeRequested: 0
    badAdvObj: 0

Eventually you'll connect to 47.91.246.252:18888 which seems to be the most trusted peer.
    
    Other connected peers
    Peer 47.91.246.252:18888: [           aed3688f, ping    215 ms]
    connect time: 1969-12-31 19:00:00.0
    last know block num: 0
    needSyncFromPeer:true
    needSyncFromUs:false
    syncToFetchSize:0
    syncToFetchSizePeekNum:-1
    syncBlockRequestedSize:0
    unFetchSynNum:0
    syncChainRequested:2018-04-26 18:49:10.483
    blockInPorc:0
    NodeStat[reput: 390(0), discover: 1/1 0/0 5/5 3/3 229ms, p2p: 1/1/1 , tron: 3/2   
    
    Handle Message: INVENTORY: First hash: 3d0f8c2e68d89082f2d53222c6debdeb3254eba7dd5f6ab96946e60bdb39c588 End hash: 3d0f8c2e68d89082f2d53222c6debdeb3254eba7dd5f6ab96946e60bdb39c588 from 
    Peer: aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888
    channel active, /47.91.246.252:47240
    Finish handshake with /47.91.246.252:47240.
    rcv BLOCK_CHAIN_INVENTORY from /47.91.246.252:18888
    Handle Message: BLOCK_CHAIN_INVENTORY : first blockId: Num:0,ID:00000000000000006aaae11afeca9b84f031ef5e45c311ac302eacd4c6cbb73b end blockId: Num:500,ID:00000000000001f40ae2b2aae1b96655e4d614fdd3e927cee636968b06651300 size: 501 from 
    Peer: aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888
    update peer 47.91.246.252 block both we have, Num:0,ID:00000000000000006aaae11afeca9b84f031ef5e45c311ac30
    2eacd4c6cbb73b
    headNumber:0
    syncBeginNumber:0
    solidBlockNumber:0
    headNumber:0
    syncBeginNumber:0
    solidBlockNumber:0
    Send Message:SYNC_BLOCK_CHAIN: 
    Num:0,ID:00000000000000006aaae11afeca9b84f031ef5e45c311ac302eacd4c6cbb73b
    Num:251,ID:00000000000000fbd7da930cb3b20ba7235cda24f51b48d73cb27534ec8300ee
    Num:376,ID:0000000000000178201326b872ec527841e9a2b9d64837800eb2179a77be691b
    Num:439,ID:00000000000001b7ccda2628b8f2f7841be1251be901d8b7465be28ff21ef861
    Num:470,ID:00000000000001d6dca3061341dbd59f6a84074691dc9d88e452a55e4254dba4
    Num:486,ID:00000000000001e62adc9c52ad1281dbc862fa54e1770589fb7d49d227f03e1d
    Num:494,ID:00000000000001eee197065f8628548a675e80ac7762118eb1a506af21f336c4
    Num:498,ID:00000000000001f2182f8ff99eddf21df25968a11ca2c4f77bf3986e38ffd612
    Num:500,ID:00000000000001f40ae2b2aae1b96655e4d614fdd3e927cee636968b06651300 to
    aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888
    18:49:10.963 INFO  [MessageQueueTimer-1] [MessageQueue](MessageQueue.java:186) send SYNC_BLOCK_CHAIN to /47.91.246.252:18888

After some kind of handshake the trusted node goes from 'Other connected peers' to 'Active peers'

    Active peers
    Peer 47.91.246.252:18888: [           aed3688f, ping    271 ms]
    connect time: 2018-04-26 18:49:20.459
    last know block num: 0
    needSyncFromPeer:true
    needSyncFromUs:false
    syncToFetchSize:0
    syncToFetchSizePeekNum:-1
    syncBlockRequestedSize:0
    unFetchSynNum:0
    syncChainRequested:2018-04-26 18:49:20.462
    blockInPorc:0
    NodeStat[reput: 180(0), discover: 1/1 0/0 5/5 3/3 229ms, p2p: 2/2/2 , tron: 8/4 X 1<=DUPLICATE_PEER 

Finally we start to sync blocks

    There are 31922 blocks we need to sync.
    
    handle Block number is 1
    Handle Message: [Message Type: BLOCK, Message Hash:0000000000000002ecb773a55f1f0311372e34e47749af06ed973e37299b1159] from Peer: aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888

Once you have caught up to the blockchain you'll mostly just see transactions:

    1:TransactionCapsule 
    [ hash=51d9db7a72aa66886b8886d2352f600c67acf14c8a39fa9639491342d268b802
    contract list:{ [0] type: TransferContract
    from address=[B@d6a90e1
    to address=[B@62622a52
    transfer amount=8000000
    sign=G16nepaM8mBg//Yp06AZocYlYxSz8XIc6+d9hsn12r3GIVTWiP7mOPGhz8VziZ/4+107JQCZTY6N29ICH6IIj+4=
    }
    ]


Finally you'll receive notice of your place in the queue

    save block num:33716
    update peer 47.91.246.252 block both we have Num:33716,ID:00000000000083b46d4a63d6cbda07f0fd5ce620230545c629b
    scheduledWitness:a00a9309758508413039e4bc5a3d113f3ecc55031d, currentSlot:304959039

Here's a command to extract the display of discovered nodes.

    more ~/java-tron/logs/tron.log | grep nodes.
	
> Write Node statistics to PeersStore: 2375 nodes.



Next article - Adding a Solidity Node
