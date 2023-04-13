Configuration of a Blockchain Explorer
-

__I thank <a href="https://github.com/samqju">Samqju</a> and his collaborators, <a href="https://github.com/iquidus/explorer">Inquidus</a>, my old StakeShare team, and the <a href="https://github.com/bulwark-crypto/bulwark-explorer">Bulwark</a> team thanks to whom I am building this new guide. Thanks also to all the contributors of the Blockchain community.__

This guide installs the exploration of an independent blockchain.

__The name of your cryptocurrency is replaced by "examplecoin".__

Requirements
-

- To follow this guide you need a server or a VPS with at least 4Gb of RAM..

- It is strongly recommended that your server/VPS only hosts the Blockchain explorer and nothing else.

- Configure it with the Linux OS Ubuntu 18.04.

---

#### 1. Ubuntu system update.

> `sudo sh -c 'apt update && apt upgrade -y'`

#### 2. Installation required.

> `sudo apt install git && htop && unzip`

Installation of Nodejs and npm. First install <a href="https://github.com/nvm-sh/nvm">Node Version Manager (nvm)</a> to be sure to get compatible versions.

> `nvm install node`

(supp) Installation of yarn.

> `sudo apt install yarn`

#### 3. List of packages for the compilation of the blockchain.

_Note. Some of these libraries are not essential, however their usefulness varies depending on your cryptocurrency, so it is wise to install them all._

> `sudo apt install build-essential libdb++-dev libqrencode-dev miniupnpc libminiupnpc-dev autoconf pkg-config libtool autotools-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev libevent-dev libboost-all-dev libgtk2.0-dev bsdmainutils python3 libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-test-dev libboost-thread-dev libboost-program-options-dev libzmq3-dev libkrb5-dev protobuf-compiler software-properties-common libdb4.8-dev libdb4.8++-dev libssl-dev libssl1.0-dev automake -y`

Error : It is possible that the installation of the libraries blocks, because some of them need other installation packages to run. If this happens to you, look at the names of the packages involved in your script, remove them from the command, then run it again.

Install the removed packages just before point 4, or else, just before the `.autogen` script of point 4. Do it individually following this line pattern :

> `sudo apt install <nom du package>`

Berkeley DB.

> `sudo add-apt-repository ppa:bitcoin/bitcoin`

> `sudo apt update`

#### 4. Compilation and Configuration of the daemon.

Download the directory of your cryptocurrency.

> `git clone https://github.com/examplecoin/examplecoin-core`

Stand inside.

> `cd examplecoin-x.x.x`

Run the daemon build script.

> `./autogen.sh && ./configure --with-gui=no --disable-tests --with-unsupported-ssl --with-libressl --with-incompatible-bdb && make`

#### 5. Strip ! And RPC configuration.

Go inside the newly generated `.examplecoin` hidden folder, and create the `examplecoin.conf` file following the outline below.

> `nano examplecoin.conf`

```
rpcuser=examplecoinuser
rpcpassword=examplecoinpassword
listen=1
daemon=1
server=1
staking=0
txindex=1
rpcport=XXXXXX (Use the RPC port of your cryptocurrency here)
```

Save, then move to the src folder : `cd examplecoin-core/src`

Make a strip. This manipulation will allow you to interact with the daemon wherever you are in the architecture.

> `strip stakeshared stakeshare-cli`

Then,

> `mkdir ~/ssx && mv stakeshared stakeshare-cli ~/ssx/`

Start the daemon,

> `~/ssx/stakeshared -daemon`

These are the three important commands. You can now run them at any time. Start, stop the daemon, the getinfo allows you to see if the blockchain is synchronizing correctly.

```
~/ssx/stakeshared -daemon
~/ssx/stakeshare-cli stop
~/ssx/stakeshare-cli getinfo
```

If at this point you notice with the `getinfo` that the number of blocks does not move after a few minutes, then it means that the daemon is not able to index itself. Stop it.

You should, if your blockchain is important, download a bootstrap (as recent as possible) of the chain to your server/VPS and replace the `peer.dat, chainstate and block` files/folder inside the hidden `.examplecoin` folder.

Restart the daemon.

If it still doesn't work, after several manipulations and tests, then you probably need to restart this tutorial. The daemon must be synchronized before continuing.

#### 6. Create the user explorer.

Enter the following lines one by one.

> `adduser explorer`

> `usermod -aG sudo explorer`

> `su explorer`

#### 7. Easy installation of explorer.

> `bash <( curl https://raw.githubusercontent.com/bulwark-crypto/bulwark-explorer/master/script/install.sh )`

#### 8. Configuration of your information.

Make sure `config.js` and `config.server.js` match your cryptocurrency information, similar to the `examplecoin.conf` information in point 5.

Also, `config.js` must have the correct host name specific to your server/VPS. Take note of the port. Recently, CoinMarketCap changed their API, so it is complicated for BlockEx to retrieve data that is no longer on the same model.

Use the CoinGecko API and modify the appropriate line in `config.js`.

```
  coinMarketCap: {
	api: 'https://api.coingecko.com/api/v3/coins/',
	ticker: 'examplecoin'
```

#### 9. Configure MongoDB.

> `mongo`

> `use blockex`

Use the same information as in `config.js`.

> `db.createUser( { user: "blockexuser", pwd: "Explorer!1", roles: [ "readWrite" ] } )`

> `exit`

#### 10. Synchronize all blocks with the Explorer.

> `tail -f /home/explorer/blockex/tmp/block.log`

This can take quite a long time depending on the size of your blockchain.

#### 11. Crontab.

The following commands are necessary for BlockEx to update itself continuously. Indeed, the Crontab allows to schedule automated tasks. We will use it to update the explorer so that it is always in agreement with the data of the blockchain.

First of all, you have to update the `crontab` file, open it following this path : `/home/explorer/crontab`.

Modify it to include the following template.

Check path node or nodejs for `/usr/bin/nodejs`.

Check the correct path for `/home/explorer/blockex`.

``` bash
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

*/1 * * * * cd /home/explorer/blockex && /usr/bin/nodejs ./cron/masternode.js >> ./tmp/masternode.log 2>&1
*/1 * * * * cd /home/explorer/blockex && /usr/bin/nodejs ./cron/peer.js >> ./tmp/peer.log 2>&1
*/1 * * * * cd /home/explorer/blockex && /usr/bin/nodejs ./cron/rich.js >> ./tmp/rich.log 2>&1
*/5 * * * * cd /home/explorer/blockex && /usr/bin/nodejs ./cron/coin.js >> ./tmp/coin.log 2>&1
0 0 * * * cd /home/explorer/blockex && /usr/bin/nodejs ./cron/timeIntervals.js >> ./tmp/timeIntervals.log 2>&1
#
```

#### 12. Synchronization.

Before starting the synchronization, go to `/home/explorer/blockex/cron`.

Open `coin.js` and change the information to match the one below. This allows the CoinGecko API to retrieve the correct existing information since we changed it in point 8.

```
const coin = new Coin({
    cap: market.market_data.total_supply,
    createdAt: date,
    blocks: info.blocks,
    btc: market.market_data.current_price.btc,
    diff: info.difficulty,
    mnsOff: masternodes.total - masternodes.stable,
    mnsOn: masternodes.stable,
    netHash: nethashps,
    peers: info.connections,
    status: 'Online',
    supply: market.market_data.circulating_supply, // TODO: change to actual count from db.
    usd: market.market_data.current_price.usd,
    countCarverAddresses,
    countCarverMovements
  });
```

It is important that you get the correct json information. If you have any doubts or if `yarn run cron:coin` does not work, be sure to check your part information on the coingecko api and follow the json scheme according to the error indicated by the yarn command.

Enter the following commands.

`yarn run cron:coin` : Will retrieve part information like price and supply.

`yarn run cron:masternode` - Updates the list of masternodes in the database with the most recent information deleting the old information before.

`yarn run cron:peer` - Gathers the list of peers and retrieves geographic IP information.

`yarn run cron:block` - Synchronize blocks and transactions by storing them in the database.

`yarn run cron:rich` - Generates the rich list.

#### 13. Running the Explorer.

Create the client's web interface using webpack. Run it,

`yarn run build:web`

API Start up.

> `yarn run start:api`

Start up in the browser.

> `yarn run start:web`

Congratulations ! Your Explorer is now functional and online at : __{IP-of-your-server}:3000__

#### 14. Test.

Performs client-side tests.

> `yarn run start:api`

Performs rpc connection tests, database connection and api endpoints.

> `yarn run start:web`

---

#### 15. Graphic modification.

To modify the graphical aspect of the explorer, you will find the icons in the `public` folder and the interactions to modify from the front in the `components` or `containers`.

#### 16. Support, Help.

If you are looking for answers to some problems, you can find me on Discord ; nickname `Sheitak#6192` or with my GitHub.

Or contact the creators of this template to explore in GitHub ; <a href="https://github.com/bulwark-crypto/bulwark-explorer">Bulwark</a>

---

Thank you very much for following this tutorial and I look forward to seeing your explorer available on the web soon.

Made with love by @Sheitak ♥

#### 17. Useful links.

- <a href="https://gist.github.com/samqju/b9fc6c007f083e6429387051e24da1c3">Samqju Tutorial</a>
- <a href="https://github.com/iquidus/explorer">Inquidus</a>
- <a href="https://www.reddit.com/r/BiblePay/comments/7elm7r/iquidus_block_explorer_guide/">BiblePay Inquidus<a>
- <a href="https://github.com/bulwark-crypto/bulwark-explorer">Bulwark</a>
- <a href="https://bugsdev.blogspot.com/2018/03/node-and-iquidus-explorer-setup-for.html">BugsDevBlog</a>
