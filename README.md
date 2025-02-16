<div align="center">

# **Bittensor** <!-- omit in toc -->
[![Discord Chat](https://img.shields.io/discord/308323056592486420.svg)](https://discord.gg/3rUr6EcvbB)
[![PyPI version](https://badge.fury.io/py/bittensor.svg)](https://badge.fury.io/py/bittensor)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) 

---

### Internet-scale Neural Networks <!-- omit in toc -->

[Discord](https://discord.gg/3rUr6EcvbB) • [Docs](https://app.gitbook.com/@opentensor/s/bittensor/) • [Network](https://www.bittensor.com/metagraph) • [Research](https://uploads-ssl.webflow.com/5cfe9427d35b15fd0afc4687/5fa940aea6a95b870067cf09_bittensor.pdf) • [Code](https://github.com/opentensor/BitTensor)

</div>

At Bittensor, we are creating an open, decentralized, peer-to-peer network that functions as a market system for the development of artificial intelligence. Our purpose is not only to accelerate the development of AI by creating an environment optimally condusive to its evolution, but to democratize the global production and use of this valuable commodity. Our aim is to disrupt the status quo: a system that is centrally controlled, inefficient and unsustainable. In developing the Bittensor API, we are allowing engineers to monetize their work, gain access to machine intelligence and join our community of creative, forward-thinking individuals. For more info, read our [paper](https://uploads-ssl.webflow.com/5cfe9427d35b15fd0afc4687/6021920718efe27873351f68_bittensor.pdf).

- [1. Documentation](#1-documentation)
- [2. Install](#2-install)
- [3. Using Bittensor](#3-using-bittensor)
  - [3.1. Client](#31-client)
  - [3.2. Server](#32-server)
  - [3.3. Validator](#33-validator)
- [4. Features](#4-features)
  - [4.1. Using the CLI](#41-cli)
  - [4.2. Selecting the network to join](#42-selecting-the-network-to-join)
  - [4.3. Running a template miner](#43-running-a-template-miner)
  - [4.4. Running a template server](#44-running-a-template-server)
  - [4.5. Subscription to the network](#45-subscription-to-the-network)
  - [4.6. Syncing with the chain/ Finding the ranks/stake/uids of other nodes](#46-syncing-with-the-chain-finding-the-ranksstakeuids-of-other-nodes)
  - [4.7. Finding and creating the endpoints for other nodes in the network](#47-finding-and-creating-the-endpoints-for-other-nodes-in-the-network)
  - [4.8. Querying others in the network](#48-querying-others-in-the-network)
  - [4.9. Creating a Priority Thread Pool for the axon](#49-creating-a-priority-thread-pool-for-the-axon)
- [5. License](#5-license)
- [6. Acknowledgments](#6-acknowledgments)

## 1. Documentation

https://opentensor.gitbook.io/bittensor/

## 2. Install
Three ways to install Bittensor. 

1. Through the installer:
```
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/opentensor/bittensor/master/scripts/install.sh)"
```

2. With pip:
```bash
$ pip3 install bittensor
```

3. From source:
```
$ git clone https://github.com/opentensor/bittensor.git
$ python3 -m pip install -e bittensor/
```

## 3. Using Bittensor

The following examples showcase how to use the Bittensor API for 3 seperate purposes.

### 3.1. Client 

Querying the network for representations.

```python
import bittensor
import torch
wallet = bittensor.wallet().create().register()
graph = bittensor.metagraph().sync()
representations, _, _ = bittensor.dendrite( wallet = wallet ).forward_text (
    endpoints = graph.endpoints,
    inputs = "The quick brown fox jumped over the lazy dog"
)
representations = // N tensors with shape (1, 9, 1024)
...
// Distill model. 
...
loss.backward() // Accumulate gradients on endpoints.
```

### 3.2. Server

Serving a custom model.

```python
import bittensor
import torch
from transformers import GPT2Model, GPT2Config

model = GPT2Model( GPT2Config(vocab_size = bittensor.__vocab_size__, n_embd = bittensor.__network_dim__ , n_head = 8))
optimizer = torch.optim.SGD( [ {"params": model.parameters()} ], lr = 0.01 )

def forward_text( pubkey, inputs_x ):
    return model( inputs_x )
  
def backward_text( pubkey, inputs_x, grads_dy ):
    with torch.enable_grad():
        outputs_y = model( inputs_x.to(device) ).last_hidden_state
        torch.autograd.backward (
            tensors = [ outputs_y.to(device) ],
            grad_tensors = [ grads_dy.to(device) ]
        )
        optimizer.step()
        optimizer.zero_grad() 

wallet = bittensor.wallet().create().register()
axon = bittensor.axon (
    wallet = wallet,
    forward_text = forward_text,
    backward_text = backward_text
).start().serve()
```

### 3.3. Validator 

Validating models by setting weights.

```python
import bittensor
import torch

graph = bittensor.metagraph().sync()
dataset = bittensor.dataset()
chain_weights = torch.ones( [graph.n.item()], dtype = torch.float32 )

for batch in dataset.dataloader( 10 ):
    ...
    // Train chain_weights.
    ...
bittensor.subtensor().set_weights (
    weights = chain_weights,
    uids = graph.uids,
    wait_for_inclusion = True,
    wallet = bittensor.wallet(),
)
```
## 4. Features

### 4.1. CLI

Creating a new wallet.
```bash
$ btcli new_coldkey
$ btcli new_hotkey
```

Listing your wallets
```bash
$ btcli list
```

Registering a wallet
```bash
$ btcli register
```

Running a miner
```bash
$ btcli run
```

Checking balances
```bash
$ btcli overview
```

Checking the incentive mechanism.
```bash
$ btcli metagraph
```

Transfering funds
```bash
$ btcli transfer
```

Staking/Unstaking from a hotkey
```bash
$ btcli stake
$ btcli unstake
```

### 4.2. Selecting the network to join 
There are two open Bittensor networks: staging (Nobunaga) and main (Nakamoto, Local).

- Nobunaga (staging)
- Nakamoto (main)
- Local (localhost, mirrors nakamoto)

```bash
$ export NETWORK=local 
$ python (..) --subtensor.network $NETWORK
or
>> btcli run --subtensor.network $NETWORK
```

### 4.3. Running a template miner

The following command will run Bittensor's template miner

```bash
$ cd bittensor
$ python ./bittensor/_neuron/text/template_miner/main.py
```
or 
```python3
>> import bittensor
>> bittensor.neurons.text.template_miner.neuron().run()
```

OR with customized settings

```bash
$ cd bittensor
$ python3 ./bittensor/_neuron/text/template_miner/main.py --wallet.name <WALLET NAME> --wallet.hotkey <HOTKEY NAME>
```

For the full list of settings, please run

```bash
$ python3 ~/.bittensor/bittensor/bittensor/_neuron/neurons/text/template_miner/main.py --help
```

### 4.4. Running a template server

The template server follows a similar structure as the template miner. 

```bash
$ cd bittensor
$ python3 ./bittensor/_neuron/text/template_server/main.py --wallet.name <WALLET NAME> --wallet.hotkey <HOTKEY NAME>
```
or 
```python3
>> import bittensor
>> bittensor.neurons.text.template_server.neuron().run()
```

For the full list of settings, please run

```bash
$ cd bittensor
$ python3 ./bittensor/_neuron/text/template_server/main.py --help
```


###  4.5. Serving an endpoint on the network

Endpoints are served to the bittensor network through the axon. The axon is instantiated via a wallet which holds an account on the Bittensor network.

```python
import bittensor

wallet = bittensor.wallet().create().register()
axon = bittensor.axon (
    wallet = wallet,
    forward_text = forward_text,
    backward_text = backward_text
).start().serve()
```

### 4.6. Syncing with the chain/ Finding the ranks/stake/uids of other nodes

Information from the chain is collected/formated by the metagraph.

```bash
btcli metagraph
```
and
```python
import bittensor

meta = bittensor.metagraph()
meta.sync()

# --- uid ---
print(meta.uids)

# --- hotkeys ---
print(meta.hotkeys)

# --- ranks ---
print(meta.R)

# --- stake ---
print(meta.S)

```

### 4.7. Finding and creating the endpoints for other nodes in the network

```python
import bittensor

meta = bittensor.metagraph()
meta.sync()

### Address for the node uid 0
endpoint_as_tensor = meta.endpoints[0]
endpoint_as_object = meta.endpoint_objs[0]
```

### 4.8. Querying others in the network

```python
import bittensor

meta = bittensor.metagraph()
meta.sync()

### Address for the node uid 0
endpoint_0 = meta.endpoints[0]

### Creating the wallet, and dendrite
wallet = bittensor.wallet().create().register()
den = bittensor.dendrite(wallet = wallet)
representations, _, _ = den.forward_text (
    endpoints = endpoint_0,
    inputs = "Hello World"
)

```

## 5. License
The MIT License (MIT)
Copyright © 2021 Yuma Rao

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


## 6. Acknowledgments
**learning-at-home/hivemind**
