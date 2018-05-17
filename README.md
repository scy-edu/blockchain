# Building Your First Blockchain

You are here because you are excited about Crypto and you want to understand how Blockchains work.

## Terminology Overview

**Blockchain** - A continuously growing list of records. Called blocks, which are linked and secured using cryptography.

Each block typically contains a hash pointer as a link to a previous block, a timestamp and transaction data.

**Peer-to-Peer network** - P2P computing or networking is a distributed application architecture that partitions tasks or workloads between peers.

Peers are equally privileged, equipotent participants in the application. They are said to form a peer-to-peer network of nodes.

**Why secure?**

Security and immutability is important for a couple reasons.

**Immutability** - once data has been written to a blockchain no one, not even a system administrator, can change it.

This provides benefits for audit. As a provider of data you can prove that your data hasn’t been altered, and as a recipient of data you can be sure that the data hasn’t been altered. 

Ethereum expands on blockchains by allowing people to create dapps. 

**Decentralized apps (dapps)** - decentralized apps are a new type of software program designed to exist on the internet in a way that is not controlled by any single entity.

Where bitcoin is a decentralized value exchange, a decentralized application aims to achieve functionality beyond transactions that exchange value.

## Fundamentals:

- A Blockchain is immutable, sequential chain of records called Blocks
- Blocks contain transactions, files, or any kind of data
- Blocks are chained together uses [hashes](https://learncryptography.com/hash-functions/what-are-hash-functions)

## What are Hash Functions?

A function takes an input and from that input derives an output:

**f(x) = y**

**Example:**

```
md5("Hello world") = 5eb63bbbe01eeed093cb22bb8f5acdc3
```

The hash function md5 creates a 32 character hexadecimal output. Hash functions are generally irreversible (one-way), which means you can't figure out the input if you only know the output, unless you brute force attack.

Hash functions are often used for proving that something is the same as something else, without revealing the information beforehand.

## Prerequisites

- Comfortable with basic Python
- Understanding of HTTP requests
- Python 3.6+ 
- pip

## Getting Started

```
pip install Flask==0.12.2 requests==2.18.4
```

## Building a Blockchain

Let's start with a `blockchain.py`

```
class Blockchain(object):
    def __init__(self):
        self.chain = []
        self.current_transactions = []
        
    def new_block(self):
        # Creates a new Block and adds it to the chain
        pass
    
    def new_transaction(self):
        # Adds a new transaction to the list of transactions
        pass
    
    @staticmethod
    def hash(block):
        # Hashes a Block
        pass

    @property
    def last_block(self):
        # Returns the last Block in the chain
        pass
```

The Blockchain class is responsible for managing the chain. It will store the transactions for adding new blocks to the chain

## Goal:

We will build blocks that look like this:

```
block = {
    'index': 1,
    'timestamp': 1506057125.900785,
    'transactions': [
        {
            'sender': "8527147fe1f5426f9dd545de4b27ee00",
            'recipient': "a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount': 5,
        }
    ],
    'proof': 324984774000,
    'previous_hash': "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}
```

Each block contains:

- timestamp
- transactions
- proof
- previous_hash
- index

Blocks are immutable. The previous hash gives blockchains immutability. If a hacker corrupts an earlier Block in the chain, all subsequent blocks will contain incorrect hashes.

## Adding transactions to the Block

We will need a way of adding transactions to the Block.

**new_transaction()**

```
def new_transaction(self, sender, recipient, amount):
        """
        Creates a new transaction to go into the next mined Block
        :param sender: <str> Address of the Sender
        :param recipient: <str> Address of the Recipient
        :param amount: <int> Amount
        :return: <int> The index of the Block that will hold this transaction
        """

        self.current_transactions.append({
            'sender': sender,
            'recipient': recipient,
            'amount': amount,
        })

        return self.last_block['index'] + 1
```

The `new_transaction()` adds a transaction to the list, it returns the index of the block which the transaction will be added to - the next one to be mined.

## Creating new Blocks

A Blockchain starts with a *genesis block* (a block with no predecessors). 

We will need to add a "proof" to our genesis block which is the result of mining. 

**new_block / new_transaction / hash**

```
import hashlib
import json
from time import time


class Blockchain(object):
    def __init__(self):
        self.current_transactions = []
        self.chain = []

        # Create the genesis block
        self.new_block(previous_hash=1, proof=100)

    def new_block(self, proof, previous_hash=None):
        """
        Create a new Block in the Blockchain
        :param proof: <int> The proof given by the Proof of Work algorithm
        :param previous_hash: (Optional) <str> Hash of previous Block
        :return: <dict> New Block
        """

        block = {
            'index': len(self.chain) + 1,
            'timestamp': time(),
            'transactions': self.current_transactions,
            'proof': proof,
            'previous_hash': previous_hash or self.hash(self.chain[-1]),
        }

        # Reset the current list of transactions
        self.current_transactions = []

        self.chain.append(block)
        return block

    def new_transaction(self, sender, recipient, amount):
        """
        Creates a new transaction to go into the next mined Block
        :param sender: <str> Address of the Sender
        :param recipient: <str> Address of the Recipient
        :param amount: <int> Amount
        :return: <int> The index of the Block that will hold this transaction
        """
        self.current_transactions.append({
            'sender': sender,
            'recipient': recipient,
            'amount': amount,
        })

        return self.last_block['index'] + 1

    @property
    def last_block(self):
        return self.chain[-1]

    @staticmethod
    def hash(block):
        """
        Creates a SHA-256 hash of a Block
        :param block: <dict> Block
        :return: <str>
        """

        # We must make sure that the Dictionary is Ordered, or we'll have inconsistent hashes
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()
```

## Understanding Proof of Work

A Proof of Work (PoW) algorithm is how new Blocks are created or mined on the blockchain. The goal of PoW is discover a number which solves a problem. 

**The number must be difficult to find but easy to verify computationally**

Let's decide that the hash of x * y must end in `0`. 

So hash (x * y) = ac23dc...0

Let's fix `x = 5`

Implementing in Python:

```
from hashlib import sha256

x = 0
y = 0 # we don't know what y should be yet

while sha256(f'{x*y}'.encode()).hexdigest()[-1] != "0":
	y +=1
	
print(f'The solution is y = {y}')
```

In this case, we will find that `y = 21` since the produced hash ends in 0.

In Bitcoin, the Proof of Work is called Hashcash. 

It's the algorithm that miners race to solve in order to create a new block.

Generally, the difficulty is determined by the number of character searched for in a string.

In general, the difficulty is determined by the number of characters searched for in a string.

The miners are rewarded for their solution by receiving a coin - in a transaction.

The network is able to *easily* verify their solution

## Implementing basic Proof of Work

> Find a number p when hashed with the previous block's solution a hash with 4 leading 0s is produced

```
import hashlib
import json

from time import time
from uuid import uuid4


class Blockchain(object):
    ...
        
    def proof_of_work(self, last_proof):
        """
        Simple Proof of Work Algorithm:
         - Find a number p' such that hash(pp') contains leading 4 zeroes, where p is the previous p'
         - p is the previous proof, and p' is the new proof
        :param last_proof: <int>
        :return: <int>
        """

        proof = 0
        while self.valid_proof(last_proof, proof) is False:
            proof += 1

        return proof

    @staticmethod
    def valid_proof(last_proof, proof):
        """
        Validates the Proof: Does hash(last_proof, proof) contain 4 leading zeroes?
        :param last_proof: <int> Previous Proof
        :param proof: <int> Current Proof
        :return: <bool> True if correct, False if not.
        """

        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"
```

To adjust the difficulty of the algorithm, we could modify the number of leading zeroes. But 4 is sufficient. You'll find that the addition of a single leading zero makes a mammoth difference to the time required to find a solution

## Our Blockchain as an API

We will create three endpoints:

- `/transactions/new` to add a new transaction to a block
- `/mine` to tell our server to mine a new block
- `/chain` to return the full Blockchain

## Setting up Flask

Our server will form a single node in our blockchain network. Let's create boilerplate code

```
import hashlib
import json
from textwrap import dedent
from time import time
from uuid import uuid4

from flask import Flask


class Blockchain(object):
    ...


# Instantiate our Node
app = Flask(__name__)

# Generate a globally unique address for this node
node_identifier = str(uuid4()).replace('-', '')

# Instantiate the Blockchain
blockchain = Blockchain()


@app.route('/mine', methods=['GET'])
def mine():
    return "We'll mine a new Block"
  
@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    return "We'll add a new transaction"

@app.route('/chain', methods=['GET'])
def full_chain():
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain),
    }
    return jsonify(response), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## Transactions Endpoint

The user sends this payload to the server to add a transactions

```
{
	"sender": "my address",
	"recipient": "someone else's address",
	"amount": 5
}
```

Let's write the function for adding transactions:

```
import hashlib
import json
from textwrap import dedent
from time import time
from uuid import uuid4

from flask import Flask, jsonify, request

...

@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    values = request.get_json()

    # Check that the required fields are in the POST'ed data
    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return 'Missing values', 400

    # Create a new Transaction
    index = blockchain.new_transaction(values['sender'], values['recipient'], values['amount'])

    response = {'message': f'Transaction will be added to Block {index}'}
    return jsonify(response), 201
```

## The Mining Endpoint

Mining endpoint needs to:

- Calculate the Proof of Work (PoW)
- Reward the miner by adding a transaction granting us 1 coin
- Forge the new Block by adding it to the chain

```
import hashlib
import json

from time import time
from uuid import uuid4

from flask import Flask, jsonify, request

...

@app.route('/mine', methods=['GET'])
def mine():
    # We run the proof of work algorithm to get the next proof...
    last_block = blockchain.last_block
    last_proof = last_block['proof']
    proof = blockchain.proof_of_work(last_proof)

    # We must receive a reward for finding the proof.
    # The sender is "0" to signify that this node has mined a new coin.
    blockchain.new_transaction(
        sender="0",
        recipient=node_identifier,
        amount=1,
    )

    # Forge the new Block by adding it to the chain
    previous_hash = blockchain.hash(last_block)
    block = blockchain.new_block(proof, previous_hash)

    response = {
        'message': "New Block Forged",
        'index': block['index'],
        'transactions': block['transactions'],
        'proof': block['proof'],
        'previous_hash': block['previous_hash'],
    }
    return jsonify(response), 200
```

### Interacting with our Blockchain

Start the server:

```
$ python blockchain.py
```

- Mine a block by hitting `http://localhost:5000/mine`
- Create a new transaction by making a POST request to `http://localhost:5000/transactions/new` with a body containing the transaction structure
- Get the full chain back at `http://localhost:5000/chain`

```
{
  "chain": [
    {
      "index": 1,
      "previous_hash": 1,
      "proof": 100,
      "timestamp": 1506280650.770839,
      "transactions": []
    },
    {
      "index": 2,
      "previous_hash": "c099bc...bfb7",
      "proof": 35293,
      "timestamp": 1506280664.717925,
      "transactions": [
        {
          "amount": 1,
          "recipient": "8bbcb347e0634905b0cac7955bae152b",
          "sender": "0"
        }
      ]
    },
    {
      "index": 3,
      "previous_hash": "eff91a...10f2",
      "proof": 35089,
      "timestamp": 1506280666.1086972,
      "transactions": [
        {
          "amount": 1,
          "recipient": "8bbcb347e0634905b0cac7955bae152b",
          "sender": "0"
        }
      ]
    }
  ],
  "length": 3
}
```

The whole point of Blockchain is to be decentralized. We need every node to reflec the same chain. This is called Consensus.

## Registering new Nodes

- `/nodes/register` to accept a list of new nodes in the form of URLs
- `/nodes/resolve` to implement our Consensus Algorithm, which resolves any conflicts. 

```

from urllib.parse import urlparse
...


class Blockchain(object):
    def __init__(self):
        ...
        self.nodes = set()
        ...

    def register_node(self, address):
        """
        Add a new node to the list of nodes
        :param address: <str> Address of node. Eg. 'http://192.168.0.5:5000'
        :return: None
        """

        parsed_url = urlparse(address)
        self.nodes.add(parsed_url.netloc)
```

A conflict is when one node is different compared to another node. To resolve this, we'll make the rule the longest valid chain is authoritative. 

```
...
import requests


class Blockchain(object)
    ...
    
    def valid_chain(self, chain):
        """
        Determine if a given blockchain is valid
        :param chain: <list> A blockchain
        :return: <bool> True if valid, False if not
        """

        last_block = chain[0]
        current_index = 1

        while current_index < len(chain):
            block = chain[current_index]
            print(f'{last_block}')
            print(f'{block}')
            print("\n-----------\n")
            # Check that the hash of the block is correct
            if block['previous_hash'] != self.hash(last_block):
                return False

            # Check that the Proof of Work is correct
            if not self.valid_proof(last_block['proof'], block['proof']):
                return False

            last_block = block
            current_index += 1

        return True

    def resolve_conflicts(self):
        """
        This is our Consensus Algorithm, it resolves conflicts
        by replacing our chain with the longest one in the network.
        :return: <bool> True if our chain was replaced, False if not
        """

        neighbours = self.nodes
        new_chain = None

        # We're only looking for chains longer than ours
        max_length = len(self.chain)

        # Grab and verify the chains from all the nodes in our network
        for node in neighbours:
            response = requests.get(f'http://{node}/chain')

            if response.status_code == 200:
                length = response.json()['length']
                chain = response.json()['chain']

                # Check if the length is longer and the chain is valid
                if length > max_length and self.valid_chain(chain):
                    max_length = length
                    new_chain = chain

        # Replace our chain if we discovered a new, valid chain longer than ours
        if new_chain:
            self.chain = new_chain
            return True

        return False
 ```
 
 The first method `valid_chain()` is responsible for checking if a chain is valid by looping through each block and verifying both the hash and the proof.
 
 `resolve_conflicts()` loops through the neighboring nodes, downloads their chains and verifies them with the above method. If a valid chain is found whose length is longer, we replace ours.
 
```
@app.route('/nodes/register', methods=['POST'])
def register_nodes():
    values = request.get_json()

    nodes = values.get('nodes')
    if nodes is None:
        return "Error: Please supply a valid list of nodes", 400

    for node in nodes:
        blockchain.register_node(node)

    response = {
        'message': 'New nodes have been added',
        'total_nodes': list(blockchain.nodes),
    }
    return jsonify(response), 201


@app.route('/nodes/resolve', methods=['GET'])
def consensus():
    replaced = blockchain.resolve_conflicts()

    if replaced:
        response = {
            'message': 'Our chain was replaced',
            'new_chain': blockchain.chain
        }
    else:
        response = {
            'message': 'Our chain is authoritative',
            'chain': blockchain.chain
        }

    return jsonify(response), 200
``