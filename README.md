Configuration d'un Explorer Blockchain
-

__Je remercie <a href="https://github.com/samqju">Samqju</a> et ses collaborateurs, <a href="https://github.com/iquidus/explorer">Inquidus</a>, mon équipe du <a href="https://github.com/StakeShare">StakeShare</a>, ainsi que l'équipe du <a href="https://github.com/bulwark-crypto/bulwark-explorer">Bulwark</a> grâce à qui je construis ce nouveau guide. Merci également à tous les contributeurs du milieu Blockchain.__

Ce guide installe l'explorer d'un blockchain indépendante.

__le nom de votre cryptomonnaie est içi remplacé par "examplecoin".__

Prérequis
-

- Pour suivre ce guide vous avez besoin d'un serveur ou bien d'un VPS possédant minimum 4Gb de RAM.

- Il est fortement préférable que votre serveur/VPS n'héberge QUE l'explorer Blockchain et rien d'autre.

- Configurez avec OS Linux Ubuntu 18.04.

---

1. Mise à jours du système Ubuntu.

> `sudo sh -c 'apt update && apt upgrade -y'`

2. Installation nécéssaire.

> `sudo apt install git && htop && unzip`

Installation de Nodejs et npm. Installer d'abord <a href="https://github.com/nvm-sh/nvm">Node Version Manager (nvm)</a> pour être certain d'obtenir des versions compatibles.

> `nvm install node`

(supp)Installation de yarn

> `sudo apt install yarn`

3. Liste des packages pour la compilation de la blockchain.

_Note. Certaines de ces librairies ne sont pas indispensables, cependant leurs utilitées varies en fonction de votre cryptomonnaie, il est donc judicieux de toutes les installers._

> `sudo apt install build-essential libdb++-dev libqrencode-dev miniupnpc libminiupnpc-dev autoconf pkg-config libtool autotools-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev libevent-dev libboost-all-dev libgtk2.0-dev bsdmainutils python3 libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-test-dev libboost-thread-dev libboost-program-options-dev libzmq3-dev libkrb5-dev protobuf-compiler software-properties-common libdb4.8-dev libdb4.8++-dev libssl-dev libssl1.0-dev automake -y`

Erreur : Il est possible que l'installation des librairies bloque, car certaines d'entre-elles ont besoin d'autres packages d'installation pour s'éxécuter. Si cela vous arrive, observez le nom des packages en causes dans votre script, retirez-les de la commande, puis exécuter de nouveau.

Installer les packages retirés juste avant le point 4, ou bien, juste avant le script `.autogen` du point 4. Faite le de façon individuel en suivant ce schéma de lignes :

> `sudo apt install <nom du package>`

Berkeley DB 

> `sudo add-apt-repository ppa:bitcoin/bitcoin`

> `sudo apt update`

4. Compilation et Configuration du daemon

Télécharger le répertoire de votre cryptomonnaie.

> `git clone https://github.com/examplecoin/examplecoin-core`

Placez-vous à l'intérieur.

> `cd examplecoin-x.x.x`

Lancez le script de compilation du daemon.

> `./autogen.sh && ./configure --with-gui=no --disable-tests --with-unsupported-ssl --with-libressl --with-incompatible-bdb && make`

5. Strip ! Et configuration RPC.

Placez-vous à l'intérieur du dossier caché `.examplecoin` nouvellement généré, puis créer le fichier examplecoin.conf en suivant le schéma ci-dessous.

> `nano examplecoin.conf`

```
rpcuser=examplecoinuser
rpcpassword=examplecoinpassword
listen=1
daemon=1
server=1
staking=0
txindex=1
rpcport=XXXXXX (Utilisez ici le port RPC de votre cryptomonnaie)
```

Enregistrer, puis placez-vous dans le dossier src : `cd examplecoin-core/src`

Effectuez un strip. Cette manipulation vous permettras d'intéragir avec le daemon où que vous soyez dans l'architecture.

> `strip stakeshared stakeshare-cli`

Puis,

> `mkdir ~/ssx && mv stakeshared stakeshare-cli ~/ssx/`

Lancer le daemon,

> `~/ssx/stakeshared -daemon`

Voici les trois commandes importantes. Vous pouvez à présent les lancers à n'importe quel moments. Lancer, arréter le daemon, le getinfo vous permet de voir si la blockchain se synchronise correctement.

```
~/ssx/stakeshared -daemon
~/ssx/stakeshare-cli stop
~/ssx/stakeshare-cli getinfo
```

Si à ce stade, vous constatez à l'aide du `getinfo` que le nombre de block ne bouge pas au bout de quelques minutes, alors cela signifie que le daemon ne parviens pas à s'indexer. Stopper-le.

Vous devez, si votre blockchain est importante, télécharger un bootstrap (le plus récent possible) de la chaîne sur votre serveur/VPS et remplacer les fichier/dossier `peer.dat , chainstate et block`, se trouvant à l'intérieur du dossier caché `.examplecoin`

Relancer le daemon.

Si cela ne fonctionne toujours pas, après plusieurs manipulation et test, alors vous devez probablement recommencer ce tutoriel. Le daemon doit-être synchroniser avant de poursuivre.

6. Créer l'utilisateur Explorer.

Entrer les lignes suivantes une par une.

> `adduser explorer`

> `usermod -aG sudo explorer`

> `su explorer`

7. Installation facile de l'explorer

> `bash <( curl https://raw.githubusercontent.com/bulwark-crypto/bulwark-explorer/master/script/install.sh )`

8. Configuration de vos informations

Assurez-vous que `config.js` et `config.server.js` correspondent bien aux informations de votre cryptomonnaie, similaire aux informations de `examplecoin.conf` du point 5.

Egalement, `config.js` doit avoir le nom de host correct propre à votre serveur/VPS. Prenez note du port. Récemment, CoinMarketCap a modifié son API, ainsi il est compliqué pour BlockEx de récupérer les données qui ne sont plus sur le même modèle.

Utilisez l'API de CoinGecko et modifier la ligne approprié dans `config.js`

```
  coinMarketCap: {
	api: 'https://api.coingecko.com/api/v3/coins/',
	ticker: 'examplecoin'
```

9. Configurer MongoDB

> `mongo`

> `use blockex`

Utilisez les même informations que dans `config.js`

> `db.createUser( { user: "blockexuser", pwd: "Explorer!1", roles: [ "readWrite" ] } )`

> `exit`

10. Synchroniser tous les block avec l'Explorer.

> `tail -f /home/explorer/blockex/tmp/block.log`

Cela peut prendre assez longtemps en fonction de la taille de votre blockchain.

11. Crontab

Les commandes suivantes sont nécéssaire pour que BlockEx se mette à jours perpetuellement. En effet, la Crontab permet de planifier des tâches automatisées. Nous allons donc l'utiliser pour mettre à jours l'explorer afin qu'il soit toujours en accords avec les données de la blockchain.

Avant toute choses, vous devez mettre à jours le fichier `crontab`, ouvrez-le en suivant ce chemin : `/home/explorer/crontab`

Modifier pour y intégrer le modèle suivant.

Vérifiez path `node` ou `nodejs` pour `/usr/bin/nodejs`

Vérifiez le path correcte pour `/home/explorer/blockex`

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

12. Synchronisation

Avant de commencer la synchronisation, rendez-vous dans `/home/explorer/blockex/cron`

Ouvrez `coin.js` et modifier les informations pour correspondre à celles ci-dessous. Cela permet à l'API de CoinGecko de récupérer les bonnes informations existantes étant donné que nous l'avons changé au point 8.

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
Il est important que vous récupériez les bonnes informations json. Si vous avez des doutes ou si `yarn run cron:coin` ne fonctionne pas, veillez à vérifier les informatiosn de votre pièce sur l'api coingecko et suivez le schéma du json en fonction de l'érreur indiqué par la commande yarn.

Entrer les commandes suivantes.

`yarn run cron:coin` : Récupérera les informations relatives aux pièces comme le prix et l'approvisionnement.

`yarn run cron:masternode` - Met à jour la liste des masternodes dans la base de données avec les informations les plus récentes effaçant les anciennes informations auparavant.

`yarn run cron:peer` - Rassemble la liste des pairs et récupére les informations géographiques IP.

`yarn run cron:block` - Synchronisera les blocs et les transactions en les stockants dans la base de données.

`yarn run cron:rich` - Génére la Rich list.

13. Exécution de l'Explorer

Créez l'interface Web du client à l'aide de webpack. Exécutez,

`yarn run build:web`

Démarrage API

> `yarn run start:api`

Démarrage dans le navigateur

> `yarn run start:web`

Félicitation ! Votre Explorer est à présent fonctionnel et en ligne à l'adresse : {IP-de-votre-serveur}:3000


14. Tester

Execute des tests côté client.

> `yarn run start:api`

Execute des tests de connexion rpc, la connexion à la base de données et les points finaux api. 

> `yarn run start:web`

---

15. Modification Graphique.

Pour modifier l'aspect graphique de l'explorer, vous trouverez les icons dans le dossier `public` et les intéractions à modifier du front dans les `components` ou les `containers`

16. Soutien, Help.

Si vous cherchez des réponses à certains problèmes, vous pouvez me retrouver sur le <a href="https://discord.gg/sjhpWsS">Discord de StakeShare</a> ; pseudo `Sheitak#3420`

Ou contacter les créateurs de ce modèle d'explorer ici ; <a href="https://discord.gg/EjhT4Ju">Bulwark Discord</a>

---

Merci beaucoup d'avoir suivis ce tutoriel et au plaisir de voir votre explorer disponible sur le web prochainement,

Réalisé avec amour par StakeShare Team.

17. Liens utiles.

- <a href="https://gist.github.com/samqju/b9fc6c007f083e6429387051e24da1c3">Samqju Tutorial</a>
- <a href="https://github.com/iquidus/explorer">Inquidus</a>
- <a href="https://www.reddit.com/r/BiblePay/comments/7elm7r/iquidus_block_explorer_guide/">BiblePay Inquidus<a>
- <a href="https://github.com/bulwark-crypto/bulwark-explorer">Bulwark</a>
- <a href="https://bugsdev.blogspot.com/2018/03/node-and-iquidus-explorer-setup-for.html">BugsDevBlog</a>