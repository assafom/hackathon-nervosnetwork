# Gitcoin: 7) Port An Existing Ethereum DApp To Polyjuice

## Screenshots or video of your application running on Godwoken.
This is an election app - the user can choose who to vote to, and can vote only once.
Before voting:
![Before voting](7.1.1.png)

After voting:
![After voting](7.1.2.png)

## Link to the GitHub repository with your application which has been ported to Godwoken. This must be a different application than the one covered in this guide.
[https://github.com/assafom/dapp-nervos-integration](https://github.com/assafom/dapp-nervos-integration)

## If you deployed any smart contracts as part of this tutorial, please provide the transaction hash of the deployment transaction, the deployed contract address, and the ABI of the deployed smart contract. (Provide all in text format.)
Deployment transaction hash:
```
0x62287a4109ea927ae03e43f722492b9f44a95a7bcd1d5102424d963fdde4560d
```

Deployed contract address:
```
0xd5A32a5c1d213bb19702DAacac642899746bEa1D
```

ABI:
```
[
	{
		"inputs": [],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "_candidateId",
				"type": "uint256"
			}
		],
		"name": "votedEvent",
		"type": "event"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "candidates",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "id",
				"type": "uint256"
			},
			{
				"internalType": "string",
				"name": "name",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "voteCount",
				"type": "uint256"
			},
			{
				"internalType": "address",
				"name": "donations",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "candidatesCount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"name": "hasVoted",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "_candidateId",
				"type": "uint256"
			}
		],
		"name": "vote",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	}
]
```
