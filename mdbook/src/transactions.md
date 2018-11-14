[[tx_chapter]]
== Transactions

Transactions are signed messages originated by an externally owned account, transmitted by the Ethereum network, and recorded (mined) on the Ethereum blockchain. Behind that basic definition, there are a lot of surprising and fascinating details. Another way to look at transactions is that they are the only thing that can trigger a change of state or cause a contract to execute in the EVM. Ethereum is a global singleton state machine, and transactions are the only thing that can make that state machine "tick", changing its state. Contracts don't run on their own. Ethereum doesn't run "in the background". Everything starts with a transaction.

In this section, we will dissect transactions, show how they work, and understand the details.

[[tx_struct]]
=== Structure of Transaction

First let's take a look at the basic structure of a transaction, as it is serialized and transmitted on the Ethereum network. Each client and application that receives a serialized transaction will store it in-memory using its own internal data structure, perhaps embellished with metadata that doesn't exist in the network serialized transaction itself. The network serialization of a transaction is, therefore, the only common standard of a transaction's structure.

A transaction is a serialized binary message that contains the following data:

nonce:: A sequence number, issued by the originating EOA, used to prevent message replay.

gas price:: The price of gas (in wei) the originator is willing to pay.

start gas:: The maximum amount of gas the originator is willing to pay.

to:: Destination Ethereum address.

value:: Amount of ether to send to the destination.

data:: Variable length binary data payload.

v,r,s:: The three components of an ECDSA signature of the originating EOA.

The transaction message's structure is serialized using the Recursive Length Prefix (RLP) encoding scheme (see <<rlp>>), which was created specifically for accurate and byte-perfect data serialization in Ethereum. All numbers in Ethereum are encoded as big-endian integers, of lengths that are multiples of 8 bits.

Note that the field labels ("to", "start gas", etc.) are shown here for clarity, but are not part of the transaction serialized data, which contains the field values RLP-encoded. In general, RLP does not contain any field delimiters or labels. RLP's length prefix is used to identify the length of each field. Anything beyond the defined length, therefore, belongs to the next field in the structure.

While this is the actual transaction structure transmitted, most internal representations and user interface visualizations embellish this with additional information, derived from the transaction or from the blockchain.

For example, you may notice there is no "+from+" data in the address identifying the originator EOA. That EOA's public key can easily be derived from the +v,r,s+ components of the ECDSA signature. The address can, in turn, be easily derived from the public key. When you see a transaction showing a "from" field, that was added by the software used to visualize the transaction. Other metadata frequently added to the transaction by client software include the block number (once it is mined) and a transaction ID (calculated hash). Again, this data is derived from the transaction and not part of the transaction message itself.

[[tx_nonce]]
==== The transaction nonce

The nonce is one of the most important and least understood components of a transaction. The definition in the Yellow Paper (see <<yellow_paper>>) reads:

 nonce: A scalar value equal to the number of transactions sent from this address or, in the case of accounts with associated code, the number of contract-creations made by this account.

Strictly speaking, the nonce is an attribute of the originating address (it only has meaning in the context of the sending address). However, the nonce is not stored explicitly as part of an account's state on the blockchain. Instead it is calculated dynamically, by counting the number of confirmed transactions that have originated from an address. 

The nonce value is also used to prevent erroneous calculations of an account balance. For example, let's say an account has a balance of 10 ether and signs two transactions spending 6 ether each, with nonce 1 and nonce 2 respectively. Which of these two transaction is valid? In a distributed system like Ethereum, nodes may receive the transactions out of sequence. The nonce forces transactions from any address to be processed sequentially, without gaps, regardless of the sequence in which they were received by a node. That way, all nodes calculate the same balance. The transaction paying 6 ether with nonce 1 will be processed successfully, reducing the balance of the account to 4 ether. The transaction paying 6 ether with the nonce 2, will be seen as invalid by all nodes, regardless of when it was received. If the  nonce 2 transaction is received first, a node will hold it and not validate it until it has seen, and processed, the nonce 1 transaction. 

The use of the nonce to ensure that all nodes calculate the same balance, and sequence transactions properly, is equivalent to the mechanism used to prevent "double spending" in Bitcoin. However, because Ethereum tracks account balances and does not track coins individually (called UTXO in Bitcoin), "double spending" can only happen if the account balance is calculated erroneously. The nonce mechanism prevents that from happening. 

[[tracking_nonce]]
===== Keeping track of nonces

In practical terms, the nonce is an up-to-date count of the number of _confirmed_ (mined) transactions that have originated from an account. To find out what the nonce is, you can interrogate the blockchain, for example via the web3 interface:

[[nonce_getTransactionCount]]
.Retrieving the transaction count of our example address
----
web3.eth.getTransactionCount("0x9e713963a92c02317a681b9bb3065a8249de124f")
40
----

[TIP]
====
The nonce is a zero-based counter, meaning the first transaction has nonce 0. In <<nonce_getTransactionCount>>, we have a transaction count of 40, meaning nonces 0 through 39 have been seen. The next transaction's nonce will be 40.
====

Your wallet will keep track of nonces for each address it manages. It's fairly simple to do that, as long as you are only originating transactions from a single point. Let's say you are writing your own wallet software or some other application that originates transactions. How do you track nonces?

When you create a new transaction, you assign the next nonce in the sequence. But until it is confirmed, it will not count towards the +getTransactionCount+ total.

[[get_tx_count_bug]]
Unfortunately, the +getTransactionCount+ function will run into some problems if we send a few transactions in a row. There is a known bug where +getTransactionCount+ does not count pending transactions correctly. Let's look at an example:

----
web3.eth.getTransactionCount("0x9e713963a92c02317a681b9bb3065a8249de124f", "pending")
40
web3.eth.sendTransaction({from: web3.eth.accounts[0], to: "0xB0920c523d582040f2BCB1bD7FB1c7C1ECEbdB34", value: web3.toWei(0.01, "ether")});
web3.eth.getTransactionCount("0x9e713963a92c02317a681b9bb3065a8249de124f", "pending")
41
web3.eth.sendTransaction({from: web3.eth.accounts[0], to: "0xB0920c523d582040f2BCB1bD7FB1c7C1ECEbdB34", value: web3.toWei(0.01, "ether")});
web3.eth.getTransactionCount("0x9e713963a92c02317a681b9bb3065a8249de124f", "pending")
41
web3.eth.sendTransaction({from: web3.eth.accounts[0], to: "0xB0920c523d582040f2BCB1bD7FB1c7C1ECEbdB34", value: web3.toWei(0.01, "ether")});
web3.eth.getTransactionCount("0x9e713963a92c02317a681b9bb3065a8249de124f", "pending")
41
----

As you can see, the first transaction we sent increased the transaction count to 41, showing the pending transaction. But when we sent 3 more transactions in quick succession, the +getTransactionCount+ call didn't count them correctly. It only counted one, even though there are 3 pending in the mempool. If we wait a few seconds, once a block is mined, the +getTransactionCount+ call will return the correct number. But in the interim, while there are more than one transactions pending, it does not help us.

When you build an application that constructs transactions, it cannot rely on +getTransactionCount+ for pending transactions. Only when pending and confirmed are equal (all outstanding transactions are confirmed) can you trust the output of +getTransactionCount+ to start your nonce counter. Thereafter, keep track of the nonce in your application until each transaction confirms.

Parity's JSON RPC interface offers the +parity_nextNonce+ function, that returns the next nonce that should be used in a transaction. The +parity_nextNonce+ function counts nonces correctly, even if you construct several transactions in rapid succession, without confirming them.

[[parity_curl]]
Parity has a web console for accessing the JSON RPC interface, but here we are using a command line HTTP client to access it:

----
curl --data '{"method":"parity_nextNonce","params":["0x9e713963a92c02317a681b9bb3065a8249de124f"],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545

{"jsonrpc":"2.0","result":"0x32","id":1}
----

[[gaps_nonce]]
===== Gaps in nonces, duplicate nonces, and confirmation

It is important to keep track of nonces if you are creating transactions programmatically, especially if you are doing so from multiple independent processes simultaneously.

The Ethereum network processes transactions sequentially, based on the nonce. That means that if you transmit a transaction with nonce +0+ and then transmit a transaction with nonce +2+, the second transaction will not be mined. It will be stored in the mempool, while the Ethereum network waits for the missing nonce to appear. All nodes will assume that the missing nonce has simply been delayed and that the transaction with nonce +2+ was received out-of-sequence.

If you then transmit a transaction with the missing nonce +1+, both transactions (nonces +1+ and +2+) will be mined. Once you fill the gap, the network can mine the out-of-sequence transaction that it held in the mempool.

What this means is that if you create several transactions in sequence and one of them does not get mined, all the subsequent transactions will be "stuck", waiting for the missing nonce. A transaction can create an inadvertent "gap" in the nonce sequence because it is invalid or has insufficient gas. To get things moving again, you have to transmit a valid transaction with the missing nonce.

If on the other hand you accidentally duplicate a nonce, for example by transmitting two transactions with the same nonce, but different recipients or values, then one of them will be confirmed and one will be rejected. Which one is confirmed will be determined by the sequence in which they arrive at the first validating node that receives them.

As you can see, keeping track of nonces is necessary and if your application doesn't manage that process correctly, you will run into problems. Unfortunately, things get even more difficult if you are trying to do this concurrently, as we will see in the next section.

[[concurrency]]
===== Concurrency, transaction origination, and nonces

Concurrency is a complex aspect of computer science, and it crops up unexpectedly sometimes, especially in decentralized/distributed real-time systems like Ethereum.

In simple terms, concurrency is when you have simultaneous computation by multiple independent systems. These can be in the same program (e.g. threading), on the same CPU (e.g. multi-processing), or on different computers (i.e. distributed systems). Ethereum, by definition, is a system that allows concurrency of operations (nodes, clients, DApps), but enforces a singleton state (e.g. there is only one common/shared state of the system at each mined block).

Now, imagine that we have multiple independent wallet applications that are generating transactions from the same address or addresses. One example of such a situation would be an exchange processing withdrawals for a hot wallet. Ideally, you'd want to have more than one computer processing withdrawals, so that it doesn't become a bottleneck or single point of failure. However, this quickly becomes problematic, as having more than one computer producing withdrawals will result in some thorny concurrency problems, not least of which is the selection of nonces. How do multiple computers generating, signing and broadcasting transactions from the same hot wallet account coordinate?

You could use a single computer to assign nonces, on a first-come first-served basis to computers signing transactions. However, this computer is now a single-point of failure. Worse, if several nonces are assigned and one of them never gets used (because of a failure in the computer processing the transaction with that nonce), all of the subsequent ones get stuck.

You could generate the transactions, but don't sign them or assign a nonce to them. Then queue them to a single node that signs them and also keeps track of nonces. Again, you have a single point of failure. The signing and tracking of nonces is the part of your operation that is likely to become congested under load, whereas the generation of the unsigned transaction is the part you don't really need to parallelize. You have concurrency, but you don't have it in any useful part of the process.

In the end, these concurrency problems, on top of the difficulty of tracking account balances and transaction confirmation in independent processes, force most implementations towards avoiding concurrency and creating bottlenecks such as a single process handling all withdrawal transactions in an exchange.

[[tx_gas]]
=== Transaction gas

We discuss _gas_ in detail in <<gas>>. However, let's cover some basics about the role of the +gasPrice+ and +startGas+ components of a transaction.

Gas is the fuel of Ethereum. Gas is not ether - it's a separate virtual currency with an exchange rate vis-a-vis ether. Ethereum uses gas to control the amount of resources that a transaction can spend, since it will be processed on thousands of computers around the world. The open-ended (Turing complete) computation model requires some form of metering in order to avoid denial of service attacks or inadvertent resource-devouring transactions.

Gas is separate from ether in order to protect the system from the volatility that might arise along with rapid changes in the value of ether.

The +gasPrice+ field in a transaction allows the transaction originator to set the exchange rate of each unit of gas. Gas price is measured in +wei+ per gas unit. For example, in a transaction we recently created for an example in this book, our wallet had set the +gasPrice+ to +3 Gwei+ (3 Giga-wei, 3 billion wei).

The popular site +ethgasstation.info+ provides information on the current prices of gas, and other relevant gas metrics for the Ethereum main network:

https://ethgasstation.info/

Wallets can adjust the +gasPrice+ in transactions they originate, to achieve faster confirmation (mining) of transactions. The higher the +gasPrice+, the faster the transaction is likely to confirm. Conversely, lower priority transactions can carry a reduced price they are willing to pay for gas, resulting in slower confirmation. The minimum +gasPrice+ that can be set is zero, which means a fee-free transaction. During periods of low demand for space in a block, such transactions will get mined.

[TIP]
====
The minimum acceptable gasPrice is zero. That means that wallets can generate completely free transactions. Depending on capacity, these may never be mined, but there is nothing in the protocol that prohibits free transactions. You can find several examples of such transactions successfully mined in the Ethereum blockchain.
====

[[gas_price_suggestion]]
The web3 interface offers a gasPrice suggestion, by calculating a median price across several blocks:

----
truffle(mainnet)> web3.eth.getGasPrice(console.log)
truffle(mainnet)> null BigNumber { s: 1, e: 10, c: [ 10000000000 ] }
----

[[calc_gas_price]]
The second important field related to gas, is +startGas+. This is explained in more detail in <<gas>>. In simple terms, +startGas+ defines how many units of gas the transaction originator is willing to spend to complete the transaction. For simple payments, meaning transactions that transfer ether from one EOA to another EOA, the gas amount needed is fixed at 21,000 gas units. To calculate how much ether that will cost, you multiply 21,000 with the +gasPrice+ you're willing to pay:

----
truffle(mainnet)> web3.eth.getGasPrice(function(err, res) {console.log(res*21000)} )
truffle(mainnet)> 210000000000000
----

If your transaction's destination address is a contract, then the amount of gas needed can be estimated but cannot be determined with accuracy. That's because a contract can evaluate different conditions that lead to different execution paths, with different gas costs. That means that the contract may execute only a simple computation or a more complex one depending on conditions that are outside of your control and cannot be predicted. To demonstrate this let's use a rather contrived example: each time a contract is called it increments a counter and on the 100th time (only) it computes something complex. If you call the contract 99 times one thing happens, but on the 100th something completely different happens. The amount of gas you would pay for that depends on how many other transactions have called that function before your transaction is mined. Perhaps your estimate is based on being the 99th transaction and just before your transaction is mined, someone else calls the contract for the 99th time. Now you're the 100th transaction to call and the computation effort (and gas cost) is much higher.

To borrow a common analogy used in Ethereum, you can think of startGas as the fuel tank in your car (your car is the transaction). You fill the tank with as much gas as you think it will need for the journey (the computation needed to validate your transaction). You can estimate the amount to some degree, but there might be unexpected changes to your journey such as a diversion (a more complex execution path), which increase fuel consumption.

The analogy to a fuel tank is somewhat misleading, however. It's more like a credit account for a gas station company, where you pay after the trip is completed, based on how much gas you actually used. When you transmit your transaction, one of the first validation steps is to check that the account it originated from has enough ether to pay the +gasPrice * startGas+ fee. But the amount is not actually deducted from your account until the end of the transaction execution. You are only billed for gas actually consumed by your transaction at the end, but you have to have enough balance for the maximum amount you are willing to pay before you send your transaction.

[[tx_recipient]]
=== Transaction recipient

The recipient of a transaction is specified in the +to+ field. This contains a 20-byte Ethereum address. The address can be an EOA or a contract address.

Ethereum does no further validation of this field. Any 20-byte value is considered valid. If the 20-byte value corresponds to an address without a corresponding private key, or without a corresponding contract, the transaction is still valid. Ethereum has no way of knowing whether an address was correctly derived from a public key (and therefore from a private key).

[WARNING]
====
Ethereum cannot and does not validate recipient addresses in a transaction. You can send to an address that has no corresponding private key or contract, thereby "burning" the ether, rendering it forever unspendable. Validation should be done at the user interface level.
====

Sending a transaction to an invalid address will _burn_ the ether sent, rendering it forever inaccessible (unspendable), since no signature can be generated to spend it. It is assumed that validation of the address happens at the user interface level (see <<eip-55>> or <<icap>>). In fact, there are a number of valid reasons for burning ether, including as a game-theory disincentive to cheating in payment channels and other smart contracts.

[[tx_value_data]]
=== Transaction value and data

The main "payload" of a transaction is contained in two fields: +value+ and +data+. Transactions can have both value and data, only value, only data, or neither value nor data. All four combinations are valid.

A transaction with only value is a _payment_. A transaction with only data is an _invocation_. A transaction with neither value nor data, well that's probably just a waste of gas! But it is still possible.

Let's try all of the above combinations:

[[src_dest_address]]
First, we set the source and destination addresses from our wallet, just to make the demo easier to read:

.Set the source and destination addresses
[source,javascript]
----
src = web3.eth.accounts[0];
dst = web3.eth.accounts[1];
----

[[tx_value_nodata]]
===== Transaction with value (payment), and no data payload

[[tx_value_nodata_src]]
.Value, no data
[source,javascript]
----
web3.eth.sendTransaction({from: src, to: dst, value: web3.toWei(0.01, "ether"), data: ""});
----

Our wallet shows a confirmation screen, indicating the value to send, and no data payload:

[[parity_txdemo_value_nodata]]
.Parity wallet showing a transaction with value, but no data
image::images/parity_txdemo_value_nodata.png["Parity wallet showing a transaction with value, but no data"]


[[tx_value_data]]
===== Transaction with value (payment), and a data payload

[[tx_value_data_src]]
.Value and data
[source,javascript]
----
web3.eth.sendTransaction({from: src, to: dst, value: web3.toWei(0.01, "ether"), data: "0x1234"});
----

Our wallet shows a confirmation screen, indicating the value to send and a data payload:

[[parity_txdemo_value_data]]
.Parity wallet showing a transaction with value and data
image::images/parity_txdemo_value_data.png["Parity wallet showing a transaction with value and data"]


[[tx_novalue_nodata]]
===== Transaction with 0 value, only a data payload

[[tx_novalue_nodata_src]]
.No value, only data
[source,javascript]
----
web3.eth.sendTransaction({from: src, to: dst, value: 0, data: "0x1234"});
----

Our wallet shows a confirmation screen, indicating the value as 0 and a data payload:

[[parity_txdemo_novalue_data]]
.Parity wallet showing a transaction with no value, only data
image::images/parity_txdemo_novalue_data.png["Parity wallet showing a transaction with no value, only data"]


[[tx_novalue_data]]
===== Transaction with neither value (payment), nor data payload

[[tx_novalue_nodata_src]]
.No value, no data
[source,javascript]
----
web3.eth.sendTransaction({from: src, to: dst, value: 0, data: ""}));
----

Our wallet shows a confirmation screen, indicating 0 value and no data:

[[parity_txdemo_novalue_nodata]]
.Parity wallet showing a transaction with no value, and no data
image::images/parity_txdemo_novalue_nodata.png["Parity wallet showing a transaction with no value, and no data"]

[[value_EOA_contracts]]
=== Transmitting value to EOAs and contracts

When you construct an Ethereum transaction that contains +value+, it is the equivalent of a _payment_. These transactions will behave differently depending on whether the destination address is a contract or not.

For EOA addresses, or rather for any address that isn't registered as a contract on the blockchain, Ethereum will record a state change, adding the value you sent to the balance of the address. If the address has not been seen before, it will be created and its balance initialized to the +value+ of your payment.

If the destination address (+to+) is a contract, then the EVM will execute the contract and attempt to call the function named in the +data+ payload of your transaction (see <<invocation>>). If there is no +data+ payload in your transaction, the EVM will call the destination contract's _fallback_ function and, if that function is payable, will execute it to determine what to do next.

A contract can reject incoming payments by throwing an exception immediately when the payable function is called, or as determined by conditions coded in the payable function. If the payable function terminated successfully (without an exception), then the contract's state is updated to reflect an increase in the contract's ether balance.

[[data_EOA]]
=== Transmitting a data payload to an EOA or contract

When your transaction contains a +data+ payload, it is most likely addressed to a contract address. That doesn't mean you cannot send a +data+ payload to an EOA. In fact, you can do that. However, in that case, the interpretation of the +data+ payload is up to the wallet you use to access the EOA. Most wallets ignore any +data+ payload received in a transaction to an EOA they control. In the future, it is possible that standards may emerge that allow wallets to interpret +data+ payload encodings the way contracts do, thereby allowing transactions to invoke functions running inside user wallets. The critical difference is that any interpretation of the data payload by an EOA, is not subject to Ethereum's consensus rules, unlike a contract execution.

For now, let's assume your transaction is delivering a +data+ payload to a contract address. In that case, the +data+ payload will be interpreted by the EVM as _function invocation_, calling the named function and passing any encoded arguments to the function.

The +data+ payload sent to a contract is a hex-serialized encoding of:

A function selector:: The first 4 bytes of the Keccak256 hash of the function's _prototype_. This allows the EVM to unambiguously identify which function you wish to invoke.

The function arguments:: The function's arguments, encoded according to the rules for the various elementary types defined by the EVM.

[[withdraw_function_src]]
Let's look at a simple example, drawn from our <<solidity_faucet_example>>. In +Faucet.sol+, we defined a single function for withdrawals:

----
function withdraw(uint withdraw_amount) public {
----

The _prototype_ of the withdraw function is defined as the string containing the name of the function, followed by the data type of each of its arguments enclosed in parenthesis and separated by a single comma. The function name is +withdraw+ and it takes a single argument that is a uint (which is an alias for uint256). So the prototype of +withdraw+ would be:

----
withdraw(uint256)
----

Let's calculate the Keccak256 hash of this string (we can use the truffle console or any JavaScript web3 console to do that):

[source,javascript]
----
web3.sha3("withdraw(uint256)");
'0x2e1a7d4d13322e7b96f9a57413e1525c250fb7a9021cf91d1540d5b69f16a49f'
----

The first 4 bytes of the hash are +0x2e1a7d4d+. That's our "function selector" value, which will tell the EVM which function we want to call.

Next, let's calculate a value to pass as the argument +withdraw_amount+. We want to withdraw 0.01 ether. Let's encode that to a hex-serialized big-endian unsigned 256-bit integer, denominated in wei:

[source,javascript]
----
withdraw_amount = web3.toWei(0.01, "ether");
'10000000000000000'
withdraw_amount_hex = web3.toHex(withdraw_amount);
'0x2386f26fc10000'
----

Now, we add the function selector to the amount (padded to 32 bytes):

----
2e1a7d4d000000000000000000000000000000000000000000000000002386f26fc10000
----

That's the +data+ payload for our transaction, invoking the +withdraw+ function and requesting 0.01 ether as the +withdraw_amount+.

[[contract_reg]]
=== Special transaction: Contract registration

There is one special case of a transaction with a data payload and no value. This is a transaction that _registers_ a new contract. Contract registration transactions are sent to a special destination address, the zero address. In simple terms, the +to+ field in a contract registration transaction contains the address +0x0+. This address represents neither an EOA (there is no corresponding private/public key pair) nor a contract. It can never spend ether or initiate a transaction. It is only used as a destination, with the special meaning "register this contract".

While the zero address is only intended for contract registration, it sometimes receives payments from various addresses. There are two explanations for this: either it is by accident, resulting in the loss of ether, or it is an intentional _ether burn_ (see <<burning_ether>>). If you want to do an intentional ether burn, you should make your intention clear to the network and use the specially designated burn address instead:

[[burn_address]]
----
0x000000000000000000000000000000000000dEaD
----

[WARNING]
====
Any ether sent to the contract registration address +0x0+ or the designated burn address +0x0...dEaD+ above will become unspendable and lost forever.
====

A contract registration transaction should contain no ether value, only a data payload that contains the compiled bytecode of the contract. The only effect of this transaction is to register the contract.

As an example, we can publish +Faucet.sol+ used in <<intro>>. The contract needs to be compiled into a binary hexadecimal representation. This can be done with the Solidiy compiler.
----
> solc --bin Faucet.sol
======= Faucet.sol:Faucet =======
Binary:
6060604052341561000f57600080fd5b60e58061001d6000396000f300606060405260043610603f576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680632e1a7d4d146041575b005b3415604b57600080fd5b605f60048080359060200190919050506061565b005b67016345785d8a00008111151515607757600080fd5b3373ffffffffffffffffffffffffffffffffffffffff166108fc829081150290604051600060405180830381858888f19350505050151560b657600080fd5b505600a165627a7a72305820d276ddd56041f7dc2d2eab69f01dd0a0146446562e25236cf4ba5095d2ee802f0029
----
The same information can also be obtained from the Remix online compiler.
Now we can create the transaction.
[source,javascript]
----
> src = web3.eth.accounts[0];
> faucet_code = "0x6060604052341561000f57600080fd5b60e58061001d6000396000f300606060405260043610603f576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680632e1a7d4d146041575b005b3415604b57600080fd5b605f60048080359060200190919050506061565b005b67016345785d8a00008111151515607757600080fd5b3373ffffffffffffffffffffffffffffffffffffffff166108fc829081150290604051600060405180830381858888f19350505050151560b657600080fd5b505600a165627a7a72305820d276ddd56041f7dc2d2eab69f01dd0a0146446562e25236cf4ba5095d2ee802f0029"

> web3.eth.sendTransaction({from: src, data: faucet_code, gas: 113558, gasPrice: 200000000000})

"0x7bcc327ae5d369f75b98c0d59037eec41d44dfae75447fd753d9f2db9439124b"
----
There is no need to specify a +to+ parameter, the default zero address will be used. You can specify +gasPrice+ and +gas+ limit.
Once, the contract is mined we can see it on etherscan block explorer

[[publish_contract_from_web3]]
.Etherscan showing the contract successully minded
image::images/contract_published.png["Etherscan showing the contract successully mined"]

You can look at the receipt of transaction to get information about the contract.

[source,javascript]
----
> eth.getTransactionReceipt("0x7bcc327ae5d369f75b98c0d59037eec41d44dfae75447fd753d9f2db9439124b");

{
  blockHash: "0x6fa7d8bf982490de6246875deb2c21e5f3665b4422089c060138fc3907a95bb2",
  blockNumber: 3105256,
  contractAddress: "0xb226270965b43373e98ffc6e2c7693c17e2cf40b",
  cumulativeGasUsed: 113558,
  from: "0x2a966a87db5913c1b22a59b0d8a11cc51c167a89",
  gasUsed: 113558,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x7bcc327ae5d369f75b98c0d59037eec41d44dfae75447fd753d9f2db9439124b",
  transactionIndex: 0
}
----
Here we can see the address of the contract. We can send and receive funds from the contract as shown in <<data_EOA>>.

[source,javascript]
----
> contract_address = "0xb226270965b43373e98ffc6e2c7693c17e2cf40b"
> web3.eth.sendTransaction({from: src, to: contract_address, value: web3.toWei(0.1, "ether"), data: ""});

"0x6ebf2e1fe95cc9c1fe2e1a0dc45678ccd127d374fdf145c5c8e6cd4ea2e6ca9f"

> web3.eth.sendTransaction({from: src, to: contract_address, value: 0, data: "0x2e1a7d4d000000000000000000000000000000000000000000000000002386f26fc10000"});

"0x59836029e7ce43e92daf84313816ca31420a76a9a571b69e31ec4bf4b37cd16e"
----

After a while, both transactions are visible on ethescan
[[publish_contract_transactions]]
.Etherscan showing the transactions for sending and receiving funds
image::images/published_contract_transactions.png["Etherscan showing the transactions for sending and receiving funds"]


[[digital_sign]]
=== Digital signatures

((("transactions", "digital signatures and")))So far, we have not delved into any detail about "digital signatures." In this section, we look at how digital signatures work and how they can present proof of ownership of a private key without revealing that private key.

[[ecdsa]]
==== Elliptic Curve Digital Signature Algorithm (ECDSA)

((("digital signatures", "algorithm used")))((("Elliptic Curve Digital Signature Algorithm (ECDSA)")))The digital signature algorithm used in Ethereum is the _Elliptic Curve Digital Signature Algorithm_, or _ECDSA_. ECDSA is the algorithm used for digital signatures based on elliptic curve private/public key pairs, as described in <<elliptic_curve>>.

((("digital signatures", "purposes of")))A digital signature serves three purposes in Ethereum (see the following sidebar). First, the signature proves that the owner of the private key, who is by implication the owner of an Ethereum account, has _authorized_ the spending of ether, or execution of a contract. Secondly, the proof of authorization is _undeniable_ (non-repudiation). Thirdly, the signature proves that the transaction data have not and _cannot be modified_ by anyone after the transaction has been signed.

[[digital_signature_definition]]
.Wikipedia's Definition of a "Digital Signature"
****
((("digital signatures", "defined")))A digital signature is a mathematical scheme for demonstrating the authenticity of a digital message or documents. A valid digital signature gives a recipient reason to believe that the message was created by a known sender (authentication), that the sender cannot deny having sent the message (non-repudiation), and that the message was not altered in transit (integrity).

_Source: https://en.wikipedia.org/wiki/Digital_signature_
****

[[digital_sign_work]]
==== How Digital Signatures Work

((("digital signatures", "how they work")))A digital signature is a _mathematical scheme_ that consists of two parts. The first part is an algorithm for creating a signature, using a private key (the signing key) from a message (the transaction). The second part is an algorithm that allows anyone to verify the signature by only using the message and a public key.

[[digital_sign_create]]
===== Creating a digital signature

In Ethereum's implementation of ECDSA, the "message" being signed is the transaction, or more accurately, the Keccak256 hash of the RLP-encoded data from the transaction. The signing key is the EOA's private key. The result is the signature:

latexmath:[\(Sig = F_{sig}(F_{keccak256}(m), k)\)]

where:

* _k_ is the signing private key
* _m_ is the RLP-encoded transaction
* _F_~_keccak256_~ is the Keccak256 hash function
* _F_~_sig_~ is the signing algorithm
* _Sig_ is the resulting signature

More details on the mathematics of ECDSA can be found in <<ecdsa_math>>.

[[sign_function]]
The function _F_~_sig_~ produces a signature +Sig+ that is composed of two values, commonly referred to as +R+ and +S+:

----
Sig = (R, S)
----

[[verify_sign]]
==== Verifying the Signature

((("digital signatures", "verifying")))To verify the signature, one must have the signature (+R+ and +S+), the serialized transaction, and the public key (that corresponds to the private key used to create the signature). Essentially, verification of a signature means "Only the owner of the private key that generated this public key could have produced this signature on this transaction."

The signature verification algorithm takes the message (a hash of the transaction or parts of it), the signer's public key and the signature (+R+ and +S+ values), and returns TRUE if the signature is valid for this message and public key.

[[ecdsa_math]]
==== ECDSA Math

((("Elliptic Curve Digital Signature Algorithm (ECDSA)")))As mentioned previously, signatures are created by a mathematical function _F_~_sig_~ that produces a signature composed of two values _R_ and _S_. In this section we look at the function _F_~_sig_~ in more detail.

((("public and private keys", "key pairs", "ephemeral")))The signature algorithm first generates an _ephemeral_ (temporary) private/public key pair. This temporary key pair is used in the calculation of the _R_ and _S_ values, after a transformation involving the signing private key and the transaction hash.

The temporary key pair is generated by two input values:

1. A random number _q_, which is used as the temporary private key
1. and the elliptic curve generator point _G_

From _q_ and _G_, we generate the corresponding temporary public key _Q_ (calculated as _Q = q*G_, in the same way Ethereum public keys are derived; see <<pubkey>>). The _R_ value of the digital signature is then the x coordinate of the ephemeral public key _Q_.

From there, the algorithm calculates the _S_ value of the signature, such that:

_S_ &#8801; __q__^-1^ (__Keccak256__(__m__) + __k__ * __R__)  {nbsp} {nbsp} (_mod p_)

where:

* _q_ is the ephemeral private key
* _R_ is the x coordinate of the ephemeral public key
* _k_ is the signing (EOA owner's) private key
* _m_ is the transaction data
* _p_ is the prime order of the elliptic curve

Verification is the inverse of the signature generation function, using the _R_, _S_ values and the public key to calculate a value _Q_, which is a point on the elliptic curve (the ephemeral public key used in signature creation):

_Q_ &#8801; __S__^-1^ * __Keccak256__(__m__) * _G_ + __S__^-1^ * _R_ * _K_  {nbsp} {nbsp} (_mod p_)

where:

* _R_ and _S_ are the signature values
* _K_ is the signer's (EOA owner's) public key
* _m_ is the transaction data that was signed
* _G_ is the elliptic curve generator point
* _p_ is the prime order of the elliptic curve

If the x coordinate of the calculated point _Q_ is equal to _R_, then the verifier can conclude that the signature is valid.

Note that in verifying the signature, the private key is neither known nor revealed.

[TIP]
====
ECDSA is necessarily a fairly complicated piece of math; a full explanation is beyond the scope of this book. A number of great guides online take you through it step by step: search for "ECDSA explained" or try this one: http://bit.ly/2r0HhGB[].
====

[[tx_sign]]
==== Transaction signing in practice

To produce a valid transaction, the originator must apply a digital signature to the message, using the Elliptic Curve Digital Signature algorithm. When we say "sign the transaction", we actually mean "sign the Keccak256 hash of the RLP serialized transaction data". The signature is applied to the hash of the transaction data, not the transaction itself.

[TIP]
====
At block # 2,675,000, Ethereum implemented the "Spurious Dragon" hard fork that, among other changes, introduced a new signing scheme that includes transaction replay protection. This new signing scheme is specified in EIP-155 (see <<eip155>>). This change affects the first step of the signing process, adding three fields (v, r, s) to the transaction before signing.
====

To sign a transaction in Ethereum, the originator must:

1. Create a transaction data structure, containing the nine fields: nonce, gasPrice, startGas, to, value, data, v, r, s
1. Produce an RLP-encoded serialized message of the transaction
1. Compute the Keccak256 hash of this serialized message
1. Compute the ECDSA signature, signing the hash with the originating EOA's private key
1. Insert the ECDSA signature's computed +r+ and +s+ values in the transaction

[[raw_tx]]
==== Raw transaction creation and signing

Let's create a raw transaction and sign it, using the +ethereumjs-tx+ library. The source code for this example is in +raw_tx_demo.js+ in the GitHub repository:

[[raw_tx_demo_source]]
.raw_tx_demo.js: Creating and signing a raw transaction in JavaScript
----
include::code/web3js/raw_tx/raw_tx_demo.js[]
----

Download it here:
https://github.com/ethereumbook/ethereumbook/blob/develop/code/web3js/raw_tx/raw_tx_demo.js

[[raw_tx_demo_run]]
Run the example code:
----
$ node raw_tx_demo.js
RLP-Encoded Tx: 0xe6808609184e72a0008303000094b0920c523d582040f2bcb1bd7fb1c7c1ecebdb348080
Tx Hash: 0xaa7f03f9f4e52fcf69f836a6d2bbc7706580adce0a068ff6525ba337218e6992
Signed Raw Transaction: 0xf866808609184e72a0008303000094b0920c523d582040f2bcb1bd7fb1c7c1ecebdb3480801ca0ae236e42bd8de1be3e62fea2fafac7ec6a0ac3d699c6156ac4f28356a4c034fda0422e3e6466347ef6e9796df8a3b6b05bed913476dc84bbfca90043e3f65d5224
----

[[raw_tx_eip155]]
==== Raw transaction creation with EIP-155

The EIP-155 "Simple Replay Attack Protection" standard specifies a replay-attack-protected transaction encoding, which includes a _chain identifier_ inside the transaction data, prior to signing. This ensures that transactions created for one blockchain (e.g. Ethereum main network) are invalid on another blockchain (e.g. Ethereum Classic or Ropsten test network). Therefore, transactions broadcast on one network cannot be _replayed_ on another, hence the "replay attack protection" name of the standard.

EIP-155 adds three fields to the transaction data structure, +v+, +r+, and +s+. The +r+ and +s+ fields are initialized to zero. These three fields are added to the transaction data _before it is encoded and hashed_. The three additional fields therefore change the transaction's hash, to which the signature is later applied. By including the chain identifier in the data being signed, the transaction signature prevents any changes, as the signature is invalidated if the chain identifier is modified. Therefore, EIP-155 makes it impossible for a transaction to be replayed on another chain, because the signature's validity depends on the chain identifier.

[[sign_prefix_table]]
The +v+ signature prefix field is initialized to the chain identifier, with values:

|======
| Chain | Chain ID |
| Ethereum main net | 1 |
| Morden (obsolete), Expanse | 2 |
| Ropsten | 3 |
| Rinkeby | 4 |
| Rootstock main net | 30 |
| Rootstock test net | 31 |
| Kovan | 42 |
| Ethereum Classic main net | 61 |
| Ethereum Classic test net | 62 |
| Geth private testnets | 1337 |
|======

The resulting transaction structure is RLP-encoded, hashed and signed. The signature algorithm is modified slightly to encode the chainID in the +v+ prefix, too.

For more details, see the EIP-155 specification:
https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md

[[sign_prefix]]
=== The signature prefix value (v) and public key recovery

As mentioned in <<tx_struct>>, the transaction message doesn't include any "from" field. That's because the originator's public key can be computed directly from the ECDSA signature. Once you have the public key, you can compute the address easily. The process of recovering the signer's public key is called a _Public Key Recovery_.

Given the values +r+ and +s+, that were computed in <<ecdsa_math>>, we can compute two possible public keys.

First, we compute two elliptic curve points R and R^'^, from the x coordinate +r+ value that is in the signature. There are two points, because the elliptic curve is symmetric across the x-axis, so that for any value +x+, there are two possible values that fit the curve, on either side of the x-axis.

From +r+, we also calculate r^-1^ which is the multiplicative inverse of +r+.

Finally we calculate +z+, which is the n-lowest bits of the message hash, where n is the order of the elliptic curve.

The two possible public keys are then:

K~1~ = r^-1^ (sR - zG)

and

K~2~ = r^-1^ (sR^'^ - zG)

where:

* K~1~ and K~2~ are the two possibilities for the signer's public key
* r^-1^ is the multiplicative inverse of signature's +r+ value
* s is the signature's +s+ value
* R and R^'^ are the two possibilities for the ephemeral public key _Q_
* z are the n-lowest bits of the message hash
* G is the elliptic curve generator point

To make things more efficient, the transaction signature includes a prefix value +v+, which tells us which of the two possible R values are the ephemeral public key. If +v+ is even, then R is the correct value. If +v+ is odd, then R^'^. That way, we need to calculate only one value for R and only one value for K.

[[offline_sign]]
=== Separating signing and transmission (offline signing)

Once a transaction is signed, it is ready to transmit to the Ethereum network. The three steps of creating, signing, and broadcasting a transaction normally happen in a single function, for example using +web3.eth.sendTransaction+. However, as we saw in <<raw_tx>>, you can create and sign the transaction in two separate steps. Once you have a signed transaction, you can then transmit it using +web3.eth.sendSignedTransaction+ which takes a hex-encoded and signed transaction message and transmits it on the Ethereum network.

Why would you want to separate the signing and transmission of transactions? The most common reason is security: the computer that signs a transaction must have unlocked private keys loaded in memory. The computer that transmits must be connected to the internet and be running an Ethereum client. If these two functions are on one computer, then you have private keys on an online system, which is quite dangerous. Separating the functions of signing and transmitting is called _offline signing_ and is a common security practice.

Depending on the level of security you need, your "offline signing" computer can have varying degrees of separation from the online computer, ranging from an isolated and firewalled subnet (online but segregated) to a completely offline system known as an _air-gapped_ system. In an air-gapped system there is no network connectivity at all - the computer is separated from the online environment by a gap of "air". To sign transactions you transfer them to and from the air-gapped computer using data storage media or (better) a webcam and QR code. Of course, this means you must manually transfer every transaction you want signed, and this doesn't scale.

While not many environments can utilize a fully air-gapped system, even a small degree of isolation has significant security benefits. For example, an isolated subnet with a firewall that only allows a message-queue protocol through, can offer a much-reduced attack surface and much higher security than signing on the online system. Many companies use a protocol such as ZeroMQ (0MQ), as it offers a reduced attack surface for the signing computer. With a setup like that, transactions are serialized and queued for signing. The queuing protocol transmits the serialized message, in a way similar to a TCP socket, to the signing computer. The signing computer reads the serialized transactions from the queue (carefully), applies a signature with the appropriate key, and places them on an outgoing queue. The outgoing queue transmits the signed transactions to a computer with an Ethereum client that de-queues them and transmits them.

[[tx_propagation]]
=== Transaction propagation

The Ethereum network uses a "flood" routing protocol. Each Ethereum client, acts as a _node_ in a _Peer-to-Peer (P2P)_, which (ideally) forms a _mesh_ network. No network node is "special", they all act as equal peers. We will use the term "node" to refer to an Ethereum client that is connected to and participates in the P2P network.

Transaction propagation starts with the originating Ethereum node creating (or receiving from offline) a signed transaction. The transaction is validated and then transmitted to all the other Ethereum nodes that are _directly_ connected to the originating node. On average, each Ethereum node maintains connections to at least 13 other nodes, called its _neighbors_. Each neighbor node validates the transaction as soon as they receive it. If they agree that it is valid, they store a copy and propagate it to all their neighbors (except the one it came from). As a result, the transaction ripples outwards from the originating node, _flooding_ across the network, until all nodes in the network have a copy of the transaction.

Within just a few seconds, an Ethereum transaction propagates to all the Ethereum nodes around the globe. From the perspective of each node, it is not possible to discern the origin of the transaction. The neighbor that sent it to our node may be the originator of the transaction or may have received it from one of its neighbors. To be able to track the origin of transactions, or interfere with propagation, an attacker would have to control a significant percentage of all nodes. This is part of the security and privacy design of P2P networks, especially as applied to blockchains.

[[chain_record]]
=== Recording in the chain

While all the nodes in Ethereum are equal peers, some of them are operated by _miners_ and are feeding transactions and blocks to _mining farms_, which are computers with high-performance Graphical Processing Units (GPUs). The mining computers add transactions to a candidate block and attempt to find a _Proof-of-Work_ that makes the candidate block valid. We will discuss this in more detail in <<consensus>>.

Without going into too much detail, valid transactions will eventually be included in a block of transactions and, thus, recorded in the Ethereum blockchain. Once mined into a block, transactions also modify the state of the Ethereum singleton, either by modifying the balance of an account (in the case of a simple payment), or by invoking contracts that change their internal state. These changes are recorded alongside the transaction, in the form of a transaction _receipt_, which may also include _events_. We will examine all this in much more detail in <<evm>>

Our transaction has completed its journey from creation to signing by an EOA, propagation, and finally mining. It has changed the state of the singleton and left an indelible mark on the blockchain.

=== Multiple signatures (multisig) transactions

If you are familiar with Bitcoin's scripting capabilities, you know that it is possible to create a Bitcoin multisig account which can only spend funds when multiple parties sign the transaction (e.g. 2 of 2 or 3 of 4 signatures). Ethereum's value transactions have no provisions for multiple signatures, although arbitrary contracts with any conditions may be deployed to handle the transfer of ether and tokens alike. 

To protect your ether under a multisig condition, transfer them to a multisig contract. Whenever you want to send funds to another account, all the required users will need to send transactions to the contract using a regular wallet software, effectively authorizing the contract to perform the final transaction.

These contracts can also be designed to require multiple signatures before executing local code or to trigger other contracts. The security of the scheme is ultimately determined by the multisig contract code.

Discussion and Grid+ reference implementation: +
https://blog.gridplus.io/toward-an-ethereum-multisig-standard-c566c7b7a3f6