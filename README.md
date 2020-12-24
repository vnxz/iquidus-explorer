Iquidus Explorer - 1.6.2
================

An open source block explorer written in node.js.

### See it in action

*  [Deutsche eMark](http://b.emark.tk/)
*  [Vertcoin](http://explorer.vertcoin.info/)
*  [TheHolyRogerCoin (ROGER) Explorer](https://explorer.theholyroger.com/)
*  [CPUChain (CPU) Explorer](https://explorer.cpuchain.org/)
*  [Omega Blockchain Explorer](http://explorer.omegablockchain.net/)
*  [Sugarchain Explorer](https://1explorer.sugarchain.org/)
*  [Florincoin](https://florincoin.info/info)
*  [Maxcoin Explorer 1](https://explorer.maxcoinproject.net/)


*Note: If you would like your instance mentioned here contact me*

### Requires

*  node.js >= 0.10.28 (8.17.0 is advised for updated dependencies)
*  mongodb 2.6.x
*  *coind

### Create database

Enter MongoDB cli:

    $ mongo

Create databse:

    > use explorerdb

Create user with read/write access:

    > db.createUser( { user: "iquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )

*Note: If you're using mongo shell 2.4.x, use the following to create your user:

    > db.addUser( { user: "username", pwd: "password", roles: [ "readWrite"] })

### Get the source

    git clone https://github.com/iquidus/explorer explorer

### Install node modules

    cd explorer && npm install --production

### Configure

    cp ./settings.json.template ./settings.json

*Make required changes in settings.json*

### Start Explorer

    npm start

*Note: mongod must be running to start the explorer*

As of version 1.4.0 the explorer defaults to cluster mode, forking an instance of its process to each cpu core. This results in increased performance and stability. Load balancing gets automatically taken care of and any instances that for some reason die, will be restarted automatically. For testing/development (or if you just wish to) a single instance can be launched with

    node --stack-size=10000 bin/instance

To stop the cluster you can use

    npm stop

### Syncing databases with the blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorers root directory.

    Usage: node scripts/sync.js [database] [mode]

    database: (required)
    index [mode] Main index: coin info/stats, transactions & addresses
    market       Market data: summaries, orderbooks, trade history & chartdata

    mode: (required for index database only)
    update       Updates index from last sync to current block
    check        checks index for (and adds) any missing transactions/addresses
    reindex      Clears index then resyncs from genesis to current block

    notes:
    * 'current block' is the latest created block when script is executed.
    * The market database only supports (& defaults to) reindex mode.
    * If check mode finds missing data(ignoring new data since last sync),
      index_timeout in settings.json is set too low.


*It is recommended to have this script launched via a cronjob at 1+ min intervals.*

**crontab**

*Example crontab; update index every minute and market data every 2 minutes*

    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1
    */5 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1

### Wallet

Iquidus Explorer is intended to be generic, so it can be used with any wallet following the usual standards. The wallet must be running with atleast the following flags

    -daemon -txindex
    
### Security

Ensure mongodb is not exposed to the outside world via your mongo config or a firewall to prevent outside tampering of the indexed chain data. 

### Known Issues

**script is already running.**

If you receive this message when launching the sync script either a) a sync is currently in progress, or b) a previous sync was killed before it completed. If you are certian a sync is not in progress remove the index.pid and db_index.pid from the tmp folder in the explorer root directory.

    rm tmp/index.pid
    rm tmp/db_index.pid

**exceeding stack size**

    RangeError: Maximum call stack size exceeded

Nodes default stack size may be too small to index addresses with many tx's. If you experience the above error while running sync.js the stack size needs to be increased.

To determine the default setting run

    node --v8-options | grep -B0 -A1 stack_size

To run sync.js with a larger stack size launch with

    node --stack-size=[SIZE] scripts/sync.js index update

Where [SIZE] is an integer higher than the default.

*note: SIZE will depend on which blockchain you are using, you may need to play around a bit to find an optimal setting*

### License

Copyright (c) 2015, Iquidus Technology  
Copyright (c) 2015, Luke Williams  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Iquidus Technology nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.







Use the following tutorial to setup a block explorer for a Scrypt PoW/PoS coin.

Make sure that you have the following requirement.

- A server or VPS with Ubuntu Server 18.04 installed

Update your Ubuntu machine.

sudo apt-get update
sudo apt-get upgrade

Install the required dependencies.

sudo apt-get install libboost-filesystem-dev libboost-program-options-dev libboost-thread-dev libdb-dev libdb++-dev libminiupnpc-dev libkrb5-dev mongodb nodejs npm git nano screen

Go to your home directory.

cd $HOME

Note: replace “examplecoin” with the name of your coin.
Note: replace “6gs39011kick8xmqutpkrvi92xx5kwev4ykanlv1ls0ouuae5x” with the coinID of your coin.

Download the daemon from MyCoin. (Available for a paid coin)

wget "https://dl.walletbuilders.com/download?customer=6gs39011kick8xmqutpkrvi92xx5kwev4ykanlv1ls0ouuae5x&filename=examplecoin-daemon-linux.tar.gz" -O examplecoin-daemon-linux.tar.gz

Extract the tar files.

tar -xzvf examplecoin-daemon-linux.tar.gz

Install the daemon and tools.

sudo mv examplecoind /usr/bin/

Create the config file.

mkdir $HOME/.examplecoin
nano $HOME/.examplecoin/examplecoin.conf

Paste the following lines in examplecoin.conf.

rpcuser=rpc_examplecoin
rpcpassword=CNsahKWiZpMzFxdvb9R7vKuAvYWRXqr24EfCkdjz
rpcallowip=127.0.0.1
listen=1
server=1
txindex=1
daemon=1

Start your daemon with the following command.

examplecoind

Open mongodb.

mongo

Create the database “explorerdb”.

use explorerdb

Note: replace the value “414uq3EhKDNX76f7DZIMszvHrDMytCnzFevRgtAv” with a unique password.
Create the database user.

db.createUser( { user: "iquidus", pwd: "414uq3EhKDNX76f7DZIMszvHrDMytCnzFevRgtAv", roles: [ "readWrite" ] } )

Close mongodb.

exit

Go to your home directory.

cd $HOME

Download iquidus explorer.

git clone https://github.com/walletbuilders/iquidus-explorer explorer

Install iquidus explorer.

cd explorer && npm install --production

Create the settings file.

cp ./settings.json.template ./settings.json

Open the settings file.

nano settings.json

Change the marked values.

title - Change the value “IQUIDUS” with the name of your coin.

address - Change the value “127.0.0.1” with the IP address of your server.

coin - Change the value “Darkcoin” with the name of your coin.

symbol - Change the value “DRK” with the abbreviation of your coin.

password - Change the value “3xp!0reR” with the mongodb password.

user - Change the value “darkcoinrpc” with the RPC username of your coin.

pass - Change the value “123gfjk3R3pCCVjHtbRde2s5kzdf233sa” with the RPC password of your coin.

confirmations - Change the value “40” with the transaction confirmations of your coin.

api - Change the value “true” to “false”.

markets - Change the value “true” to “false”.

twitter - Change the value “true” to “false”.

difficulty - Change the value “POW” to “Hybrid”.


Original settings.json.


/*
  This file must be valid JSON. But comments are allowed

  Please edit settings.json, not settings.json.template
*/
{
  // name your instance!
  "title": "IQUIDUS",

  "address": "127.0.0.1:3001",

  // coin name
  "coin": "Darkcoin",

  // coin symbol
  "symbol": "DRK",

...

  // database settings (MongoDB)
  "dbsettings": {
    "user": "iquidus",
    "password": "3xp!0reR",
    "database": "explorerdb",
    "address": "localhost",
    "port": 27017
  },

  //update script settings
  "update_timeout": 10,
  "check_timeout": 250,

  // wallet settings
  "wallet": {
    "host": "localhost",
    "port": 9332,
    "user": "darkcoinrpc",
    "pass": "123gfjk3R3pCCVjHtbRde2s5kzdf233sa"
  },

  // confirmations
  "confirmations": 40,

  // language settings
  "locale": "locale/en.json",

  // menu settings
  "display": {
    "api": true,
    "markets": true,
    "richlist": true,
    "twitter": true,
    "facebook": false,
    "googleplus": false,
    "youtube": false,
    "search": true,
    "movement": true,
    "network": true
  },

  // index page (valid options for difficulty are POW, POS or Hybrid)
  "index": {
    "show_hashrate": true,
    "difficulty": "POW",
    "last_txs": 100
  },


Example settings.json.


/*
  This file must be valid JSON. But comments are allowed

  Please edit settings.json, not settings.json.template
*/
{
  // name your instance!
  "title": "Examplecoin",

  "address": "198.51.100.1:3001",

  // coin name
  "coin": "Examplecoin",

  // coin symbol
  "symbol": "EXP",

...

  // database settings (MongoDB)
  "dbsettings": {
    "user": "iquidus",
    "password": "414uq3EhKDNX76f7DZIMszvHrDMytCnzFevRgtAv",
    "database": "explorerdb",
    "address": "localhost",
    "port": 27017
  },

  //update script settings
  "update_timeout": 10,
  "check_timeout": 250,

  // wallet settings
  "wallet": {
    "host": "localhost",
    "port": 4763,
    "user": "rpc_examplecoin",
    "pass": "CNsahKWiZpMzFxdvb9R7vKuAvYWRXqr24EfCkdjz"
  },

  // confirmations
  "confirmations": 6,

  // language settings
  "locale": "locale/en.json",

  // menu settings
  "display": {
    "api": false,
    "markets": false,
    "richlist": true,
    "twitter": false,
    "facebook": false,
    "googleplus": false,
    "youtube": false,
    "search": true,
    "movement": true,
    "network": true
  },

  // index page (valid options for difficulty are POW, POS or Hybrid)
  "index": {
    "show_hashrate": true,
    "difficulty": "Hybrid",
    "last_txs": 100
  },

Save the settings file.


----------optional----------

Replace the default logo with your own logo.

Note: The file must be a PNG file with a width and height of 128 px.

Overwrite the file “logo.png” inside the path “explorer/public/images/”.


Replace the default favicon with your own favicon.

Note 1: The file must be an ICO file with a width and height of 16 px.
Note 2: You can convert your PNG file to an ICO file on Online ICO converter.

Overwrite the file “favicon.ico” inside the path “explorer/public/”.

----------optional----------


Get the path to your block explorer.

cd $HOME/explorer
pwd

Example output

/root/explorer

Open crontab

crontab -e

Change the path “/root/explorer” with the path to your block explorer.

Paste the following lines to the bottom of the crontab.

*/1 * * * * cd /root/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
*/5 * * * * cd /root/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1

Save the crontab.

A screen session will remain open when the SSH connection is disconnected.
You can disconnect from a screen session using the keyboard combination ctrl + a + d.

Start a screen session.

screen

Start your block explorer.

cd $HOME/explorer
npm start

You can access the block explorer on the IP of your server on port 3001.

E.G. http://198.51.100.1:3001
