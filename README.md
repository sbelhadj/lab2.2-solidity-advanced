# lab2.2-solidity-advanced
Arrays, Modifiers et Gestion des Erreurs (require, revert, assert)
### Lab 2 : Arrays, Modifiers et Gestion des Erreurs (require, revert, assert)

#### Description du Lab

Dans ce lab, vous allez explorer l'utilisation des arrays, des modifiers, et des mécanismes de gestion des erreurs en solidity. Vous apprendrez à manipuler des arrays en storage et memory, à utiliser des modifiers pour restreindre l'accès à certaines fonctions, et à gérer les erreurs avec require, revert, et assert.

Vous allez créer un contrat solidity qui utilise ces éléments et interagir avec ce contrat via Hardhat pour tester son fonctionnement sur le testnet Sepolia.

----------

### Prérequis

Avant de commencer ce lab, assurez-vous d'avoir configuré votre environnement comme suit :

-   Node.js >= 16.x  
      
    
-   Metamask ou un autre wallet Ethereum installé dans votre navigateur  
      
    
-   Compte Ethereum sur Sepolia Testnet  
      
    
-   GitHub Codespaces ou un IDE local avec Hardhat installé  
      
    
-   Infura/Alchemy pour interagir avec Sepolia Testnet  
      
    

----------

### Objectifs du Lab

1.  Comprendre comment manipuler des arrays en solidity dans storage et memory.  
      
    
2.  Créer et utiliser des modifiers pour contrôler l'accès aux fonctions.  
      
    
3.  Utiliser require, revert, et assert pour gérer les erreurs de manière sécurisée.  
      
    
4.  Déployer un contrat sur le testnet Sepolia et interagir avec via Hardhat.  
      
    

----------

### Étapes du Lab

#### 1. Préparer l'environnement  
  

Installer les dépendances en utilisant npm :  
  
```bash  
cd lab-solidity-arrays-modifiers

npm install

```

1.  Vérifier la configuration de Hardhat :  
      
    

-   Assurez-vous que vous avez le fichier hardhat.config.js configuré pour Sepolia comme suit :  
      
    

```javascript  
require('@nomiclabs/hardhat-ethers');

  

module.exports = {

solidity: "0.8.19", // Dernière version de solidity

networks: {

sepolia: {

url: `https://sepolia.infura.io/v3/YOUR_INFURA_KEY`,

accounts: [`0x${YOUR_PRIVATE_KEY}`],

},

},

};

```

2.  Vérification de la configuration Metamask :  
      
    

-   Assurez-vous que Metamask est connecté au testnet Sepolia avec un solde suffisant pour déployer un contrat.  
      
    

----------

#### 2. Créer le contrat solidity

1.  Créez un fichier contracts/ArraysModifiers.sol dans le répertoire contracts/.  
      
    
2.  Implémentez un contrat utilisant des arrays, des modifiers, et des mécanismes de gestion des erreurs :  
      
    

```solidity

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

  

contract ArraysModifiers {

uint[] public numbers; // Array de nombres

mapping(address => uint[]) public userNumbers; // Mapping d'adresses vers des arrays de nombres

  

address public owner;

  

// Modificateur pour restreindre l'accès à certaines fonctions

modifier onlyOwner() {

require(msg.sender == owner, "Only owner can execute this");

_;

}

  

constructor() {

owner = msg.sender; // L'adresse qui déploie le contrat devient le propriétaire

}

  

// Ajouter un nombre à l'array public `numbers`

function addNumber(uint _number) public {

numbers.push(_number);

}

  

// Ajouter un nombre spécifique à l'array d'un utilisateur

function addUserNumber(address _user, uint _number) public {

userNumbers[_user].push(_number);

}

  

// Utilisation de `require` pour valider des conditions avant l'exécution

function transfer(uint _amount) public {

require(_amount > 0, "Amount must be greater than 0");

// Transfert simulé

}

  

// Utilisation de `revert` pour annuler une action en cas d'erreur

function withdraw(uint _amount) public {

if (_amount > address(this).balance) {

revert("Insufficient funds");

}

// Logic for withdrawal

}

  

// Fonction avec `assert` pour vérifier une condition interne

function checkInvariant() public view {

assert(numbers.length <= 100); // Assure qu'il n'y a pas plus de 100 éléments dans l'array

}

  

// Fonction pour obtenir tous les nombres d'un utilisateur

function getUserNumbers(address _user) public view returns (uint[] memory) {

return userNumbers[_user];

}

}

```

----------

#### 3. Déployer le contrat sur Sepolia

1.  Créer un script de déploiement dans scripts/deploy.js :  
      
    

```javascript

async function main() {

const [deployer] = await ethers.getSigners();

console.log("Déployé par : ", deployer.address);

  

const Contract = await ethers.getContractFactory("ArraysModifiers");

const contract = await Contract.deploy();

console.log("Contrat déployé à l'adresse : ", contract.address);

}

  

main()

.then(() => process.exit(0))

.catch((error) => {

console.error(error);

process.exit(1);

});

```

2.  Configurer les informations de testnet Sepolia dans hardhat.config.js comme mentionné précédemment.  
      
    

Exécuter le script de déploiement :  
  
```bash  
npx hardhat run scripts/deploy.js --network sepolia

```

Cela déploiera le contrat sur le testnet Sepolia et vous donnera l'adresse du contrat déployé.  
  

----------

#### 4. Interagir avec le contrat via Hardhat

1.  Après le déploiement, créez un fichier scripts/interact.js pour interagir avec le contrat déployé :  
      
    

```javascript

async function main() {

const [deployer] = await ethers.getSigners();

const contractAddress = "VOTRE_ADRESSE_DE_CONTRAT"; // Remplacez par l'adresse du contrat déployé

const contract = await ethers.getContractAt("ArraysModifiers", contractAddress);

  

// Ajouter des nombres à l'array public

await contract.addNumber(100);

await contract.addNumber(200);

  

console.log("Nombres ajoutés : ", await contract.numbers(0), await contract.numbers(1));

  

// Ajouter un nombre spécifique pour un utilisateur

await contract.addUserNumber(deployer.address, 500);

console.log("User numbers : ", await contract.getUserNumbers(deployer.address));

  

// Vérifier le retrait avec `require` et `revert`

try {

await contract.withdraw(100);

} catch (error) {

console.error(error);

}

  

// Vérifier l'invariant avec `assert`

try {

await contract.checkInvariant();

} catch (error) {

console.error(error);

}

}

  

main()

.then(() => process.exit(0))

.catch((error) => {

console.error(error);

process.exit(1);

});

```

  

Exécuter le script d'interaction :  
  
```bash  
npx hardhat run scripts/interact.js --network sepolia

```

Cela interagira avec le contrat déployé, ajoutera des valeurs dans les arrays et effectuera des vérifications de sécurité avec require, revert, et assert.  
  

----------

#### 5. Vérification des résultats sur Sepolia via Etherscan

-   Une fois que le contrat est déployé et les interactions effectuées, vous pouvez vérifier les résultats sur Sepolia Etherscan.  
      
    
-   Recherchez l'adresse du contrat déployé et consultez les transactions pour vérifier les modifications effectuées dans les arrays et les autres fonctions.  
      
    

----------

### Ressources supplémentaires

-   solidity Documentation : solidity Documentation  
      
    
-   Hardhat Documentation : Hardhat Documentation  
      
    
-   Sepolia Testnet : Sepolia Etherscan  
      
    

----------

### Fichiers du Projet

Voici un aperçu des fichiers et dossiers dans le repo pour ce lab :

```lua

  

lab-solidity-arrays-modifiers/

│

├── contracts/

│ └── ArraysModifiers.sol

│

├── scripts/

│ ├── deploy.js

│ └── interact.js

│

├── test/

│ └── arrays-modifiers.test.js (optionnel pour les tests unitaires)

│

├── hardhat.config.js

├── package.json

├── README.md

```

----------

### Objectifs de l'étudiant après ce lab

-   Avoir une bonne compréhension des arrays en solidity et des différences entre storage et memory.  
      
    
-   Savoir créer et utiliser des modifiers pour contrôler l'accès aux fonctions.  
      
    
-   Comprendre comment gérer les erreurs dans solidity à l'aide de require, revert, et assert.  
      
    
-   Déployer un contrat sur le testnet Sepolia et interagir avec ce contrat via Hardhat.
