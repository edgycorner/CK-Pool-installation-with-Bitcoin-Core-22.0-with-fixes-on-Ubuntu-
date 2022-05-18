**Step 1:**

`sudo apt-get update`

**Step 2:**

Installing git, build-essentials and yasm

`sudo apt install git build-essential yasm sudo libtool`
 
Enter y/yes whenever prompted for response

**Step 3:**

The next few steps will focus on the installation of bitcoin node

Get download link for linux from bitcoin.org website(https://bitcoin.org/en/download)

Which is https://bitcoin.org/bin/bitcoin-core-22.0/bitcoin-22.0-x86_64-linux-gnu.tar.gz , in my case. There might be an updated version in future, it shouldn't be a problem.

then download this file by the followuing command:

`sudo wget https://bitcoin.org/bin/bitcoin-core-22.0/bitcoin-22.0-x86_64-linux-gnu.tar.gz`

**step 4:**

Once it's downloaded, we will extract the file with following command:

`sudo tar xzf bitcoin-22.0-x86_64-linux-gnu.tar.gz`


**Step 5:**

Move into the extracted directory
`cd bitcoin-22.0`


**Step 6:**

Move into binary folder and install bitcoin core

`cd bin;sudo install -m 0755 -o root -g root -t /usr/local/bin * `

**Step 7:**

We will start bitcoin in daemon mode and let it sync

`bitcoind -daemon --prune=550 --dbcache=1000`

Your node should start running in prune mode(to save space this mode keeps on removing raw data of old blocks which are frivolous), and it will sync all previous blocks.

You can track the progress by using `bitcoin-cli getblockcount`


Once it reaches the latest block(check https://www.blockchain.com/explorer for latest block height), continue to step 8 


**Step 8:**

We will proceed with the installation of CK pool


Clone the ckpool repo with this command:

`git clone https://bitbucket.org/ckolivas/ckpool/src/master/`

*if you get an error saying git not found or installed, run `sudo apt-get install git` and then run the above command


**Step 9:**

move into ckpool directory with following command

`cd master`


**Step 10:**

We will make a few amends to make it work with bitcoin core 22.0, since it's depreciated

`nano src/bitcoin.c`

Search for "JSON failed to decode GBT", you can search with ctrl+w in nano editor

We just have to replace three lines of code, which are


```
if (unlikely(!previousblockhash || !target || !version || !curtime || !bits || !coinbase_aux || !flags)) {

 		LOGERR("JSON failed to decode GBT %s %s %d %d %s %s", previousblockhash, target, version, curtime, bits, flags);
		
 		goto out;
		
 	}
```
	
	
	
with the following:

```
if(!flags) {

flags = calloc(1, 1);

}

if (unlikely(!previousblockhash || !target || !version || !curtime || !bits || !coinbase_aux)) {

 		LOGERR("JSON failed to decode GBT %s %s %d %d %s %s", previousblockhash, target, version, curtime, bits, flags);
		
 		goto out;
		
 	}
	
```
	

ctlr+x then y then enter
	

**Step 11:**

Configue the package

`autoreconf -i`


**Step 12:**

Time to build

`sudo  ./autogen.sh; sudo ./configure; sudo make`


**Step 13:**

We will install ckpool so that we can call it from anywhere

`sudo make install`



**Step 14:**

Configure bitcoind rpc and set notifier

We will begin with stopping already running bitcoind

`bitcoin-cli stop`


**Step 15:**

Edit your bitcoin.conf

`cd $HOME/.bitcoin/; sudo rm bitcoin.conf; nano bitcoin.conf`

copy paste this:

`server=1
rpcuser=user
rpcpassword=pass
rpcallowip=127.0.0.1`


ctlr+x then y then enter


**Step 16:**

we will set up a notifier script, as required for ckpool

`sudo nano /usr/bin/notify.sh`

Copy paste this
`
#!/bin/bash
/usr/bin/notifier -s /opt
`

ctrl+x then press y then enter


**Step 17:**

Start bitcoind again

`bitcoind -daemon -blocknotify=/usr/bin/notify.sh --prune=550 --dbcache=1000`



**Step 18:**

Edit ckpool.conf to replace serverurl, nodeserver, trusted with "localhost:3336"

it should look like this


"serverurl" : [

        "localhost:3333"
],

"nodeserver" : [

        "localhost:3335"
	
],
"trusted" : [

        "localhost:3336"
	
]



Edit upstream url too, with localhost:3336

"upstream" : "localhost:3336"


**Step 19:**

Time to turn on the pool

`sudo ckpool --btcaddress 1edgym1sR5wLyhiziT8nk5spF1MUWAJcA --name new -L`
