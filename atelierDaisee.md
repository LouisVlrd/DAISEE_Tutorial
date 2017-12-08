Cette page va décrire la mise en place d'une blockchain locale utilisant le client Parity ainsi qu'une application de DAISEE sur un Raspberry Pi 3. 
 
## Configuration de Parity  
Pour des raisons de test ainsi que de ressources, le consensus utilisé au sein de la blockchain locale sera un [Proof-of-Authority](https://github.com/paritytech/parity/wiki/Proof-of-Authority-Chains).  
> \[Consensus\] : le consensus correspond à comment les noeuds de la blockchain vont autoriser les transactions ainsi que se partager le ledger.  
> \[Proof-of-Authority\]  : Contrairement au minage du Bitcoin qui consomme énormément de ressources en faisant résoudre par les noeuds des problèmes mathématiques ayant une difficulté arbitraire, PoA (Proof Of Authority) sélectionne parmi les noeuds participants des validateurs. Ces derniers se mettront d'accord pour accepter les transactions, empêcher les fraudes et sécuriser la blockchain. Pour une chaine privée dont on est sûr de la confiance que l'on peut apporter à ces noeuds, ce mécanisme est le plus simple à utiliser et à vérifier.  
 
La configuration de Parity est très simple car elle ne nécessite que deux fichers de configuration :   
- L'un pour configurer le comportement local du noeud, nommé **config.toml**  
- L'autre pour configurer la chaine à laquelle le noeud se connecte, nommé **demo-spec.json**  
  
> **Le fichier json configurant la chaine, tous les noeuds devront avoir à chaque instant exactement le même pour fonctionner et intéragir correctement.**  
   
### Initialisation des fichiers de configuration  

Tous les fichiers seront créés dans le répertoire `/home/pi/DAISEE`. Pour ce faire, connectez vous en ssh sur le raspberry qui vous est associé via Putty.  
La connexion effectuée, effectuez la commande `cd DAISEE`. Ceci fait, tapez `nano demo-spec.json` puis collez le code suivant :  
```
{      
    "name": "DemoPoA",
    "engine": {
        "authorityRound": {
            "params": {
                "gasLimitBoundDivisor": "0x400",
                "stepDuration": "5",
                "validators" : {
                    "list": []
                }
            }
        }
    },
    "params": {
        "maximumExtraDataSize": "0x20",
        "minGasLimit": "0x1388",
        "networkID" : "0x2323"
    },
    "genesis": {
        "seal": {
            "authorityRound": {
                "step": "0x0",
                "signature": "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
            }
        },
        "difficulty": "0x20000",
        "gasLimit": "0x5B8D80"
    },
    "accounts": {
        "0x0000000000000000000000000000000000000001": { "balance": "1", "builtin": { "name": "ecrecover", "pricing": { "linear": { "base": 3000, "word": 0 } } } },
        "0x0000000000000000000000000000000000000002": { "balance": "1", "builtin": { "name": "sha256", "pricing": { "linear": { "base": 60, "word": 12 } } } },
        "0x0000000000000000000000000000000000000003": { "balance": "1", "builtin": { "name": "ripemd160", "pricing": { "linear": { "base": 600, "word": 120 } } } },
        "0x0000000000000000000000000000000000000004": { "balance": "1", "builtin": { "name": "identity", "pricing": { "linear": { "base": 15, "word": 3 } } } }
    }
}
```
*Pour coller du texte dans un terminal, tapez shift+ctrl+v au lieu de ctrl+v*  
  
Ensuite, enregistrez le fichier avec ctrl+o puis 'Enter', puis quittez nano avec ctrl+x.  
  
Editons alors `config.toml` de la même manière en exécutant `nano config.toml` puis en collant le code suivant :  
```
[parity]
chain = "demo-spec.json"
base_path = "/home/pi/DAISEE/parity"
[network]
port = 30300
[rpc]
port = 8540
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]
[ui]
port = 8180
interface = "0.0.0.0"
[websockets]
port = 8450
[ipc]
disable = true
```
Une fois ceci fait, Parity est (presque) entièrement configuré !  

### Lancement de Parity  

Pour lancer Parity, exécutons la commande `parity -c config.toml --unsafe-expose`. Nous devons obtenir le résultat suivant :  
 
![noderunning](https://framapic.org/kGrNa8b41vVn/io7pi2l3DDMY)
 
Laissez le terminal tel quel puis ouvrez votre navigateur à l'adresse http://**<ip_raspberry>**:8180. Pour trouver l'adresse de votre raspberry, il est indiqué à la fin de la ligne *Public node URL* dans le terminal (192.168.1.xx).
![](https://framapic.org/Fmlke1ALWYh3/aN9S5i0XaLRW)  

A partir d'ici il y a deux cas de figures :
- soit une fenêtre présentant Parity s'est affichée, auquel cas vous serez guidé jusqu'à la création de votre premier compte.
- soit vous êtes sur l'UI sans avoir créé votre premier compte, ce que vous allez faire en allant dans l'onglet 'Accounts' et en cliquant sur "ACCOUNT", "New account".
- soit une fenêtre vous demandant d'entrer un token s'est affiché. Dans ce dernier cas, vous allez établir une nouvelle connexion SSH sur le raspberry puis `cd DAISEE` et `parity signer new-token -c config.toml`. Cette dernière commande affichant sur la dernière ligne le token que vous allez copier (shift+ctrl+v dans un terminal) puis coller dans le champ demandant le token.

> Pour la création du compte, n'oubliez pas de mettre un mot de passe simple ainsi que de garder la phrase de récupération, elle est demandée par la suite.
 
### Add a node 
The goal is to connect with another node.
The blockchain is private/locale, adding a node can be done with the node configuration file.
Get the address of another node and add it in `config.toml` in **[network]** part:
```
bootnodes = ["enode://ff14ae0a273e08ffbbe20b4b398460eb471e23f1b4301ce46b92a86ad420f67b9b470d097f1939fa7b9b2aae7d24e72cf7c63fe67217bdf3fd6cb60bbb7ecc59@192.168.0.47:30300"]
```
*To get that enode, you can read the line you used to get your raspberry IP in the parity launch, or via the UI, click the network status bar (red) and your enode should be displayed. If no info is displayed, simply refresh (f5)*  
![](https://framapic.org/bgZb0PSYhs7m/VkpAgUf4psdO)  
*in the bottom right*
 
Launch the node:
```bash
$ --config config.toml --unsafe-expose
```
A node is connected `1/25 peers`
 
![nodeconnected](https://framapic.org/Gp6UgPgiP2sF/P8NyaK6bbTRH)
 
We can see the same log in the terminal of the other node.
 
Stop the node, use CTRL-C.
 
> **Once the nodes are connected, their `demo-spec.json` must always be identical**
 
### Add authorities
In Proof-of-Authority, we have to define authorities which validate transactions between nodes. An authority is an account on a node of the blockchain.
 
Add one or more account addresses in `demo-spec.json` in `"validators"` part:
```
"list": [
    "0x005d23c129e6866B89E1C73FC3b05014255CEFA2",
    "0x0011067b3a4fE6fd301296AD5bC730F7a1CeCE4f"
]
```
And for each node of each authority account, following parameters have to be add in `config.toml`:
```
[account]
password = ["node.pwds"]
[mining]
engine_signer = "0x002e28950558fbede1a9675cb113f0bd20912019"
reseal_on_txs = "none"
```
> engine_signer is the account address used for authorities.
 
Create an empty file `node.pwds` and write to it the account password.
 
### Deposit Ether on the account
Each account needs Ether to deploy a smart-contract or to execute a transaction.
 
Add the account address in `demo-spec.json` in `"accounts"` part like this:
```
"0x005d23c129e6866B89E1C73FC3b05014255CEFA2": { "balance": "100000000000000000000" }
```
 
Start the node:
```bash
$ parity --config config.toml --unsafe-expose
```
 
Go to http://_raspberry-ip_:8180 and check if the account has been credited.  

If it is the case, you can add to your addressbook the accounts of the other nodes. To do it, go to the **Addressbook** tab and click 'Address' and paste the address of the other account you want to add, and his name.  
![](https://framapic.org/UYgClcmnF97S/bungWbnezu8i) 
*The address of an account is displayed under that account's name in the **Accounts** tab*
 
### Troubleshooting
In the case where a new validator node is added, after some blocks validation, the following message may appear during blocks import:
```
Error: Engine(NotProposer(Mismatch { expected: 00aa39d30f0d20ff03a22ccfc30b7efbfca597c2, found: 00bd138abd70e2f00903268f3db08f2d25677c9e }))
```
The workaround is to modify the chain specifications to revert to the precedent version, until a new error message.  
_Exemple:_ 
![parity client](https://framapic.org/6zUOp85sYvb6/6Jb25HwONahj.png)
_Only 3 blocks are imported with the last version of chain-specs.json (3 validators). In order to import the next blocks, the last validator is removed from chain specs, and blocks are imported until block #55. After this message, using the last version of chain specs (with the 3 validators) allows to import the full blockchain.  
The best solution  would be to add the new validator account only after importing the full blockchain on the node._  
 

## Smart-contracts

2 smart-contracts (in [Solidity](http://solidity.readthedocs.io/en/develop/introduction-to-smart-contracts.html)) are used:
* [token.sol](https://github.com/DAISEE/DApp-v2/blob/master/smartcontracts/token.sol)  
This is the smart-contract described in the tutorial "**[Create your own crypto-currency](https://www.ethereum.org/token)**".  
It allows to create a token, according to the [ERC20 Token Standard](https://github.com/ethereum/EIPs/issues/20).
* [daisee.sol](https://github.com/DAISEE/DApp-v2/blob/master/smartcontracts/daisee.sol)  
The current version allows to :
    * store energy data on blockchain (consumption and production),
    * make transactions between peers.  
For now, the rules that trigger transactions are not implemented in a smart contract.
Note: the smart contract is still under development, the code may change (see [Issues](https://github.com/DAISEE/DApp-v2/issues)).  

Several methods exist for deploying and testing a smart contract.  
In this wiki, the use of the Parity UI is described.  

### Deploying contracts 
In **Contracts** Tab, after clicking on "DEVELOP" button, copy paste the code from token.sol then, at the end of this code, copy/paste the code of daisee.sol:
![](https://framapic.org/bd3PlL7voo1h/Y0rstT9XQlNr)  
  
Then, in the 'select a Solidity version', choose the version 0.4.2 and click "COMPILE". That done, select the contract MyAdvancedToken in the 'select a contract' field and click "DEPLOY":    
![](https://framapic.org/j2Q5T3lleA8c/6UVWBTQgAE8v)  
  
Enter the name of the contract as shown, then the details on the next page and finally confirm the contract creation with your user's password:
![](https://framapic.org/xL3WJP6DyOC3/fmSPKmy0NYAX)  
  
![](https://framapic.org/3EFEwAHH5LLm/GNCVekcH76oc)  
  
![](https://framapic.org/sBVW01VIdool/tvvgpXF9qD1T)  
  
We successfully deployed the DaiseeCoin contract. Now, to deploy the Daisee contract, go back to the **Contracts** tab, "DEVELOP" then select Daisee under the 'select a contract' field:  
![](https://framapic.org/wNAfTD8HJQgU/ux4Q4HRwCMsL)  

Simply name it Daisee and confirm the transaction. The two smart-contract now appear in the **Contracts** tab:
![](https://framapic.org/3kiQhcuf7ECw/zkhGxv6s8Xqt)  

At this point normally, only one node has these two contracts. For letting all nodes use that contract, they have to "WATCH" it. To do that, copy the contract's address (0x----------------) on the node that has it, then transmit it to the others:  
![](https://framapic.org/g85tjH3yB6Wq/bUWJaJJD66Uy)  

The ABI has to be transmitted to for each contract too. To view it, select the contract, click 'Details' in the top-right, and go down until the ABI:  
![](https://framapic.org/ggeEzw2RetGL/ettadL1qIifq)
*To copy the contract, click inside the ABI code then use : ctrl+a to select all the code, then ctrl+c to copy it*  

Then, all the nodes that are on the same network can use that contract by going to the **Contracts** tab, select "WATCH" on the top-right, select 'Custom contract', then pasting the right data in the right field :  
![](https://framapic.org/2ZqtxW66tU0J/gLyV9E25nlKA)  

That done for the two contracts, the other node should see in the **Contracts** tab the contracts with the same address as well :  
![](https://framapic.org/HLM8W77L0qGc/gG1ZV3zQrIfQ)
  
Note: the current version of DAISEE smart contract allows to update consumption and production directly. To do this, select the Daisee contract, click the "EXECUTE" button at the top right, and choose the function 'setProduction' or 'consumeEnergy':
![](https://framapic.org/r3IFUYGIKWYQ/pp71NzT78HIl)  
  
![](https://framapic.org/M2JzuIOl9qbP/8Na2nbGOd3Ds)

> see Issue [DApp-v2/issues/5](https://github.com/DAISEE/DApp-v2/issues/5)  

## Interface

To view data, a Web interface communicates with the local node, through [Web3 JavaScript API](https://github.com/ethereum/wiki/wiki/JavaScript-API), using [web3.js](https://github.com/ethereum/web3.js).  
The microframework [Flask](http://flask.pocoo.org/) allows to display the interface from the Raspberry Pi node.
> _Tutorial used_:  
➡ [A simple smart contract Web UI using web3.js](http://hypernephelist.com/2016/06/21/a-simple-smart-contract-ui-web3.html)  

Install the requirements
```bash
$ sudo aptitude install git
$ sudo pip3 install flask pyyaml requests
```

Clone the repository
```bash
$ git clone https://github.com/DAISEE/DApp-v2.git
```


In `DApp-v2/dapp`, create the configuration file (`config.yml`) from the example and complete it:


```
contracts:
  daisee: '0xbeaE6e2747bD6db798d222E2D2185c484b5f2f9d'
  token: '0x9cf61b2b43f5695D65e633d0CA2dC03908eB6dd1'
user:
  login: 'daisee'
  pwd: '4df74be9792adc7848b15d833748b3affe59fced7e5dd5623831fa3040424761'
  coinbase: '0x005d23c129e6866b89e1c73fc3b05014255cefa2'
  name: 'node1'
  typ: 'C'
  url: 'http://0.0.0.0'
  sensorId:
  sensorLogin: ''
  sensorPassword: ''
  sensorSource: ''
  sensorPort: ''
```

|Type|Field|Description|
| ------------- | ------------- | ------------- |
|**contracts**|||
| |daisee| Daisee.sol address on the blockchain|
| |token| DaiseeCoin smart-contract address on the blockchain|
|**user**|||
| |login | login for UI |
| |pwd | hashed password for the UI* |
| |coinbase | user address |
| |type | type of node ('C' for consumer, 'P' for producer). _Not Used_ |
| |url | url of energy monitoring application** |
| |sensorId | url of energy monitoring application |
| |sensorLogin | login for energy monitoring application |
| |sensorPassword | password for energy monitoring application |
| |sensorSource | energy monitoring application  : `'CW'` for citizenWatt (only app supported for now) |
| |sensorPort | if necessary, port of energy monitoring application, example : `':8080'` |
> still under development, it may change

_\* to obtain the hashed password, use 'raspberry-ip:5000/hash/\<password>' after running the server_  
_\** if the energy monitoring application is (or will be) on the same Raspberry Pi, follow these [instructions](https://github.com/DAISEE/Prototypes/wiki/3.-Energy-monitoring) before running DAISEE app._  
_\*** if sensors are not used, sensor parameters can be empty_  


Run the server
```bash
$ export FLASK_APP=server.py
$ python3 -m flask run --host=0.0.0.0
```

Go to http://_raspberry-ip_:5000 to access to the interface.

![](https://framapic.org/K9JXZbyw9yR4/QuA3uLk6DDNv)
> The current version displays Ethereum transactions and realtime data from the energy monitoring application
