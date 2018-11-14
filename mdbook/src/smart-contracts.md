[[smart_contracts_chapter]]
== Smart contracts

As we discovered in <<intro>>, there are two different types of account in Ethereum: Externally Owned Accounts (EOAs) and contract accounts. EOAs are controlled by software, such as a wallet application, that is external to Ethereum. Contract accounts are controlled by software that runs within the Ethereum Virtual Machine (EVM). Both types of accounts are identified by an Ethereum address. In this section, we will discuss the second type, contract accounts, and the software that controls them: smart contracts.

[[smart_contracts_definition]]
=== What is a smart contract?

The term _smart contract_ has been used to describe a wide variety of different things. In the 1990’s, cryptographer Nick Szabo coined the term and defined it as  “a set of promises, specified in digital form, including protocols within which the parties perform on the other promises.” Since then, the concept of smart contracts has evolved, especially after the introduction of decentralized blockchains with the invention of Bitcoin in 2009. In this book, we use the term “smart contract” to refer to immutable computer programs that run deterministically in the context of an Ethereum Virtual Machine, which operates as a decentralized world computer.

Let’s unpack that definition:

Computer programs: Smart contracts are simply computer programs. The word contract has no legal meaning in this context.
Immutable: Once deployed, the code of a smart contract cannot change. Unlike traditional software, the only way to modify a smart contract is to deploy a new instance.
Deterministically:  The outcome of a smart contract is the same for everyone who runs it, in the context of the transaction that called it and the state of the Ethereum blockchain at the moment of execution.
The EVM context: Smart contracts operate with a very limited execution context. They can access their own state, the context of the transaction that called them and some information about the most recent block.
Decentralized world computer: The EVM runs as a local instance on every Ethereum node, but because all instances of the EVM operate on the same initial state and produce the same final state, the system as a whole operates as a single world computer.

[[smart_contract_lifecycle]]
=== Lifecycle of a smart contract

Smart contracts are typically written in a high-level language, such as Solidity. But in order to run, they must be compiled to the low-level bytecode that runs in the EVM (see <<evm>>). Once compiled, they are deployed on the Ethereum blockchain with a transaction to a special contract-creation address. Each contract is identified by an Ethereum address, which is derived from the contract-creation transaction as a function of the originating account and nonce. The Ethereum address of a contract can be used in a transaction as the recipient, sending funds to the contract or calling one of the contract’s functions.

Importantly, contracts _only run if they are called by a transaction_. All smart contracts in Ethereum are executed by a transaction initiated from an Externally Owned Account. A contract can call another contract that can call another contract etc., but the first contract in such a chain of execution must always be called by a transaction from an EOA. Contracts never run “on their own”, or “run in the background”. Contracts effectively lie “dormant” on the blockchain until a transaction triggers execution, either directly or indirectly as part of a chain of contract calls.

Transactions are _atomic_, regardless of how many contracts they call or what those contracts do when called. Transactions execute in their entirety, with any changes in the global state (contracts, accounts, etc.) recorded only if a transaction terminates successfully. Successful termination means that the program executed without an error and reached the end of execution. If a transaction fails due to an error, all of its effects (changes in state) are “rolled back” as if the transaction never ran. A failed transaction is still stored on the blockchain and deducts the gas cost from the originating account, but has no other effects on contract or account state.

A contract’s code cannot be changed. However a contract can be “deleted”, removing the code and it’s internal state (variables) from the blockchain. To delete a contract, you execute an EVM opcode called +SELFDESTRUCT+ (previously called +SUICIDE+), which removes the contract from the blockchain. That operation costs “negative gas”, thereby incentivizing the release of stored state. Deleting a contract in this way does not remove the transaction history (past) of the contract, since the blockchain itself if immutable. But it does remove the contract state from all future blocks.

[[high_level_languages]]
=== Introduction to Ethereum high-level languages

The EVM is an emulated computer that runs a special form of _machine code_ called _EVM bytecode_, just like your computer's CPU, which runs machine code such as x86_64. We will examine the operation and language of the EVM in much more detail in <<evm>>. In this section we will look at how smart contracts are written to run on the EVM.

While it is possible to program smart contracts directly in bytecode.  EVM bytecode is unwieldy and very difficult for programmers to read and understand. Instead, most Ethereum developers use a high-level symbolic language to write programs and a compiler to convert them into bytecode.

While any high-level language could be adapted to write smart contracts, that is quite a cumbersome exercise. Smart contracts operate in a highly constrained and minimalistic execution environment (the EVM), where almost all of the usual user interfaces, operating system interfaces and hardware interfaces are missing. It is easier to build a minimalistic smart-contract language, from scratch, than it is to constrain a general-purpose language and make it suitable for writing smart contracts. As a result, a number of special-purpose languages have emerged for programming smart contracts. Ethereum has several such languages, together with the compilers needed to produce EVM-executable bytecode.

In general, programming languages can be classified in two broad programming paradigms: declarative and imperative, also known as “functional” and “procedural”, respectively. In declarative programming, we write functions that express the _logic_ of a program, but not its _flow_. Declarative programming is used to create programs where there are no _side effects_, meaning that there are no changes to state outside of a function. Declarative programming languages include, for example, Haskell, SQL and HTML. Imperative programming, by contrast, is where a programmer writes a set of procedures that combine the logic and flow of a program. Imperative programming languages include, for example, BASIC, C, C++, and Java. Some languages are “hybrid”, meaning that they encourage declarative programming but can also be used to express an imperative programming paradigm. Such hybrids include Lisp, Erlang, Prolog, JavaScript, and Python. In general, any imperative language can be used to write in a declarative paradigm, but it often results in inelegant code. By comparison, pure declarative languages cannot be used to write in an imperative paradigm. In purely declarative languages, _there are no “variables”_.

While imperative programming is easier to write and read, and is more commonly used by programmers, it can be very difficult to write programs that execute _exactly as expected_. The ability of any part of the program to change the state makes it difficult to reason about a program’s execution and introduces many opportunities for unintended side effects and bugs. Declarative programming by comparison is harder to write, but avoids side effects, making it easier to understand how a program will behave.

Smart contracts create a very high burden for programmers: bugs cost money. As a result, it is critically important to write smart contracts without unintended effects. To do that, you must be able to clearly reason about the expected behavior of the program. So, declarative languages play a much bigger role in smart contracts than they do in general purpose software. Nevertheless, as you will see below, the most prolific language for smart contracts (Solidity) is imperative.

High-level programming languages for smart contracts include (ordered by approximate age):

LLL:: A functional (declarative) programming language, with Lisp-like syntax. It was the first high-level language for Ethereum smart contracts, but it is rarely used today.

Serpent:: A procedural (imperative) programming language with a syntax similar to Python. Can also be used to write functional (declarative) code, though it is not entirely free of side effects. Used sparsely. First created by Vitalik Buterin.

Solidity:: A procedural (imperative) programming language with a syntax similar to JavaScript, C++ or Java. The most popular and most frequently used language for Ethereum smart contracts. First created by Gavin Wood (co-author of this book).

Vyper:: A more recently developed language, similar to Serpent and with Python-like syntax. Intended to get closer to a pure-functional Python-like language than Serpent, but not to replace Serpent. First created by Vitalik Buterin.

Bamboo:: A newly developed language, influenced by Erlang with explicit state transitions and without iterative flows (loops). Intended to reduce side effects and increase auditability. Very new and rarely used.

As you can see, there are many languages to choose from. However, of all these Solidity is by far the most popular, to the point of being the de-facto high-level language of Ethereum and even other EVM-like blockchains. We will spend most of our time using Solidity, but will also explore some of the examples in other high-level languages, to gain an understanding of their different philosophies.

[[building_a_smart_contract_sec]]
=== Building a smart contract with Solidity

From Wikipedia:

[quote, "Wikipedia entry for Solidity"]
Solidity is a "contract-oriented" programming language for writing smart contracts. It is used for implementing smart contracts on various blockchain platforms. It was developed by Gavin Wood, Christian Reitwiessner, Alex Beregszaszi, Liana Husikyan, Yoichi Hirai and several former Ethereum core contributors to enable writing smart contracts on blockchain platforms such as Ethereum.

Solidity is developed and currently maintained by a team of developers as the Solidity project on GitHub:

https://github.com/ethereum/solidity

The main "product" of the Solidity project is the _Solidity Compiler (solc)_ which converts programs written in the Solidity language to EVM bytecode, as well as producing other artefacts such as an Application Binary Interface (ABI). Each version of the Solidity compiler corresponds to and compiles a specific version of the Solidity language.

To get started, we will download a binary executable of the Solidity compiler. Then we will write and compile a simple contract.

==== Selecting a version of Solidity

Solidity follows a versioning model called _semantic versioning_ (https://semver.org/), which specifies version numbers structured as three numbers separated by dots: +MAJOR.MINOR.PATCH+. The "major" number is incremented for major and _backwards incompatible_ changes, the "minor" number is incremented as backwards compatible features are added in between major releases, and the "patch" number is incremented for bug-fix and security related changes.

Currently, Solidity is at version +0.4.21+, where +0.4+ is the major version, 21 is the minor version and anything specified after that is a patch release.  The 0.5 major version release of Solidity is anticipated imminently.

As we saw in <<intro>>, your Solidity programs can contain a +pragma+ directive that specifies the minimum and maximum version of Solidity that is compatible with, and can be used to compile your contract.

Since Solidity is rapidly evolving it is best to always use the latest release.

==== Download/install

There are a number of methods you can use to download and install Solidity, either as a binary release or to compile from source code. You can find detailed instruction in the Solidity documentation at:

https://solidity.readthedocs.io/en/latest/installing-solidity.html

In <<install_solidity_ubuntu>>, we will install the latest binary release of Solidity on an Ubuntu/Debian operating system, using the +apt+ package manager:

[[install_solidity_ubuntu]]
.Installing solc on Ubuntu/Debian with apt package manager
[source, bash]
----
$ sudo add-apt-repository ppa:ethereum/ethereum
$ sudo apt update
$ sudo apt install solc
----

Once you have +solc+ installed, check the version by running:

----
$ solc --version
solc, the solidity compiler commandline interface
Version: 0.4.21+commit.dfe3193c.Linux.g++
----

There are a number of other ways to install Solidity, depending on your Operating System and requirements, including compiling from the source code directly. For more information see:

https://github.com/ethereum/solidity

==== Development environment

To develop in Solidity, you can use any text editor and +solc+ on the command-line. However, you might find that some text editors designed for development, such as Atom, offer additional features such as syntax highlighting and macros that make Solidity development easier.

There are also web-based development environments, such as Remix IDE (https://remix.ethereum.org/), and EthFiddle (https://ethfiddle.com/).

Use the tools that make you productive. In the end, Solidity programs are just plain-text files. While fancy editors and development environments can make things a bit easier, you don't need anything more than a simple text editor, such as vim (Linux/Unix), TextEdit (MacOS) or even NotePad (Windows). Simply save your program source code with a +.sol+ extension and it will be recognized by the Solidity compiler as a Solidity program.

==== Writing a simple Solidity program

In <<intro>> we wrote our first Solidity program, called +Faucet+.  When we first built the +Faucet+, we used the Remix IDE to compile and deploy the contract. In this section, we will revisit, improve, and embellish +Faucet+.

Our first attempt looked like this:

.Faucet.sol : A Solidity contract implementing a faucet
[source,solidity,linenums]
----
include::code/Solidity/Faucet.sol[]
----

We will build on this first example, starting in <<make_it_better>>.

==== Compiling with the Solidity compiler (solc)

Now, we will use the Solidity compiler on the command-line to compile our contract directly. The Solidity compiler +solc+ offers a variety of options, which you can see by passing the +--help+ argument.

We use the +--bin+ and +--optimize+ arguments of +solc+ to produce an optimized binary of our example contract:

[[compile_Faucet_with_solc]]
.Compiling Faucet.sol with solc
----
$ solc --optimize --bin Faucet.sol
======= Faucet.sol:Faucet =======
Binary:
6060604052341561000f57600080fd5b60cf8061001d6000396000f300606060405260043610603e5763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416632e1a7d4d81146040575b005b3415604a57600080fd5b603e60043567016345785d8a0000811115606357600080fd5b73ffffffffffffffffffffffffffffffffffffffff331681156108fc0282604051600060405180830381858888f19350505050151560a057600080fd5b505600a165627a7a723058203556d79355f2da19e773a9551e95f1ca7457f2b5fbbf4eacf7748ab59d2532130029
----

The result that +solc+ produces is a hex serialized binary that can be submitted to the Ethereum blockchain.

[[eth_contract_abi_sec]]
=== Ethereum contract Application Binary Interface (ABI)

In computer software, an Application Binary Interface (ABI) is an interface between two program modules; often, one at the level of machine code, and the other at the level of a program run by a user. An ABI defines how data structures and functions are accessed in *machine code*; this is not to be confused with an API, which defines this access in high-level, often human-readable format as *source code*. The ABI is thus the primary way of encoding and decoding data into and out of machine code.

In Ethereum, the ABI is used to encode contract calls for the EVM and to read data out of transactions. The purpose of an ABI is to define which functions in the contract can be invoked and describe how the function will accept arguments and return data.

The JSON format for a contract's ABI is given by an array of descriptions of functions (see <<solidity_functions>>) and events (see <<solidity_events>>). A function description is a JSON object with fields for `type`, `name`, `inputs`, `outputs`, `constant`, and `payable`. An event description object has fields for `type`, `name`, `inputs`, and `anonymous`.

We use the +solc+ command-line Solidity compiler to produce the ABI for our +Faucet.sol+ example contract:

----
solc --abi Faucet.sol
======= Faucet.sol:Faucet =======
Contract JSON ABI
[{"constant":false,"inputs":[{"name":"withdraw_amount","type":"uint256"}],"name":"withdraw","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"payable":true,"stateMutability":"payable","type":"fallback"}]
----

As you can see, the compiler produces a JSON object describing the two functions that are defined by +Faucet.sol+. This JSON object can be used by any application that wants to access the +Faucet+ contract once it is deployed. Using the ABI, an application such as a wallet or DApp browser, can construct transactions that call the functions in +Faucet+, with the correct arguments and argument types. For example, a wallet would know that to call the function +withdraw+ it would have to provide a +uint256+ argument named +withdraw_amount+. The wallet could prompt the user to provide that value, then create a transaction that encodes it and executes the +withdraw+ function.

All that is needed for an application to interact with a contract is an ABI and the address where the contract has been deployed.

[[solidity_pragma]]
==== Selecting Solidity compiler and language version

As we see in <<compile_Faucet_with_solc>> our +Faucet+ contract compiles successfully with Solidity version 0.4.21. But what if we had used a different version of the Solidity compiler? The language is still in constant flux and things may change in unexpected ways. Our contract is fairly simple, but what if our program used a feature that was only added in Solidity version +0.4.19+ and we tried to compile it with +0.4.18+?

To resolve such issues, Solidity offers a _compiler directive_ known as a _version pragma_ that instructs the compiler that the program expects a specific compiler (and language) version. Let’s look at an example:

[[compiler_version]]
----
pragma solidity ^0.4.19;
----

The Solidity compiler reads the version pragma and will produce an error if the compiler version is incompatible with the version pragma. In this case, our version pragma says that this program can be compiled by a Solidity compiler with a minimum version +0.4.19+. The symbol ^ states, however, that we allow compilation of any _minor revisions_ above that +0.4.19+, e.g., +0.4.20+, but not +0.5.0+ (which is a major revision, not a minor revision). Pragma directives are not compiled into EVM bytecode. They are only used by the compiler to check compatibility.

Let’s add a pragma directive to our +Faucet+ contract. We will name the new file +Faucet2.sol+, to keep track of our changes as we proceed through these examples:

.Faucet2.sol : Adding the version pragma to Faucet
[source,solidity,linenums]
----
include::code/Solidity/Faucet2.sol[]
----

Adding a version pragma is a best practice, as it avoids problems with mismatched compiler and language versions. We will explore other best practices and continue to improve the +Faucet+ contract throughout this chapter.

=== Programming with Solidity

In this section, we will look at some of the capabilities of the Solidity language. As we mentioned in <<intro>> our first contract example was very simple and also flawed in many different ways. We'll gradually improve that example, while learning how to use Solidity. This won't be a comprehensive Solidity tutorial however, as Solidity is quite complex and rapidly evolving. We'll cover the basics and give you enough of a foundation to be able to explore the rest on your own. The complete documentation for Solidity can be found at:

https://solidity.readthedocs.io/en/latest/

==== Data types

First, let's look at some of the basic data types offered in Solidity:

boolean (bool):: Boolean value, +true+ or +false+, with logical operators pass:[! (not)], pass:[&& (and)], pass:[|| (or)], pass:[== (equal)], pass:[!= (not equal)].

integer (int/uint):: Signed (int) and unsigned (uint) integers, defined in increments of 8 bits from u/int8 to u/int256. Without a size suffix, they are set to 256 bits.

fixed point (fixed/ufixed):: Fixed point numbers, defined as u/fixedMxN where M is the size in bits (increments of 8) and N is the number of decimals after the point.

address:: A 20-byte Ethereum address. The +address+ object has members +balance+ (returns the account balance) and +transfer+ (transfer ether to the account).

byte array (fixed):: Fixed size arrays of bytes, defined as +bytes1+ through +bytes32+

byte array (dynamic):: Dynamic size arrays of bytes, defined as +bytes+ or +string+

enum:: User-defined type for enumerating discrete values.

struct:: User-defines data containers for grouping variables.

mapping:: Hash lookup tables for +key => value+ pairs.

In addition to the data types above, Solidity also offers a variety of value literals that can be used to calculate different units:

time units:: The units +seconds+, +minutes+, +hours+, and +days+ can be used as a suffix, converting to multiples of the base unit +seconds+.

ether units:: The units +wei+, +finney+, +szabo+, and +ether+ can be used as a suffix, converting to multiples of the base unit +wei+.

So far, in our +Faucet+ contract example, we used +uint+ (which is an alias for +uint256+), for the +withdraw_amount+ variable. We also indirectly used an +address+ variable, which is +msg.sender+. We will use some more of these data types in our examples, throughout this chapter.

Let's use one of the unit multipliers, to improve the readability of our example contract +Faucet+. In the +withdraw+ function we limit the maximum withdrawal, expressing the amount limit as +wei+, the base unit of ether:

----
require(withdraw_amount <= 100000000000000000);
----

That's not very easy to read, so we can improve our code by using the unit multiplier +ether+, to express the value in ether instead of wei:

----
require(withdraw_amount <= 0.1 ether);
----

==== Predefined global variables and functions

When a contract is executed in the EVM, it has access to a narrow set of global objects. These include the +block+, +msg+ and +tx+ objects. In addition, Solidity exposes a number of EVM opcodes as predefined Solidity functions. In this section we will examine the variables and functions you can access from within a smart contract in Solidity.

===== Calling transaction/message context

msg:: The +msg+ object is the transaction (EOA originated) or message (contract originated) that launched the contract execution. It contains a number of useful attributes:

msg.sender:: We've already used this one. It represents the address that originated the message. If our contracts were called by an EOA transaction, then this is the address that signed the transaction.

msg.value:: The value of ether sent with the message.

msg.gas:: The amount of gas left in the message that called our contract. It has been deprecated and will be replaced with the gasleft() function as of Solidity v0.4.21.

msg.data:: The data payload of the message that called our contract.

msg.sig:: The first four bytes of the data payload, which is the function selector.

[NOTE]
====
Whenever a contract calls another contract, the values of all the attributes of +msg+ change, to reflect the new caller's information. The only exception to this is the +delegatecall+ function which runs the code of another contract/library within the original +msg+ context.
====

===== Transaction context

tx.gasprice:: The gas price in the calling Transaction

tx.origin:: The full call stack from the originating (EOA) transaction.

===== Block context

block:: The block object contains information about the current block.

block.blockhash(blockNumber):: The block hash of the specified block number, up to the last 256 blocks. Deprecated and replaced with the +blockhash()+ function in Solidity v.0.4.22.

block.coinbase:: The address of the miner of the current block.

block.difficulty:: The difficulty (Proof-of-Work) of the current block.

block.gaslimit:: The total block gas limit for the current block.

block.number:: The current block number (height).

block.timestamp:: The timestamp placed in the current block by the miner, since Unix epoch (seconds).

[[solidity_address_object]]
===== Address object

Any address, either passed as an input, or cast from a contract object, has a number of attributes and methods:

address.balance:: The balance of the address, in wei. For example, the current contract balance is +address(this).balance+.

address.transfer(amount):: Transfer the amount (wei) to this address, throwing an exception on any error. We used this function in our +Faucet+ example as a method on the +msg.sender+ address, as +msg.sender.transfer()+.

address.send(amount):: Similar to +transfer+ above, only instead of throwing an exception, it returns +false+ on error.

address.call():: Low-level call function, can construct an arbitrary message with +value+, +data+ payloads. Returns +false+ on error.

address.delegatecall():: Low-level call function, keeps the +msg+ context of the calling contract, returns +false+ on error.

===== Built-in functions

addmod, mulmod:: Modulo addition and multiplication. For example, +addmod(x,y,k)+ calculates +pass:[(x + y) % k]+.

keccak256, sha256, sha3, ripemd160:: Functions to calculate hashes with various standard hash algorithms.

ecrecover:: Recover the address used to sign a message, from the signature.

==== Contract definition

Solidity's main data type is the _contract_ object, which is defined at the top of our +Faucet+ example. Similar to any object in an object-oriented language, the contract is a container that includes data and methods.

Solidity offers two other objects that are similar to a contract:

interface:: An interface definition is structured exactly like a contract, except none of the functions are defined, they are only declared. This type of function declaration is often called a _stub_, as it tells you about the arguments and returns all types of functions without any implementation. It serves to specify a contract interface and if inherited, each of the functions must be specified in the child.

library:: A library contract is one that is meant to be deployed only once and used by other contracts, using the +delegatecall+ method (see <<solidity_address_object>>).

==== Functions

Within a contract, we define functions that can be called by an EOA transaction or another contract. In our +Faucet+ example, we have two functions: +withdraw+ and the (unnamed) _fallback_ function.

Functions are defined with the following syntax:

*function*{nbsp}FunctionName([_parameters_])
{*public*|*private*|*internal*|*external*}
[*pure*|*constant*|*view*|*payable*]
pass:[[]_modifiers_]
[*returns*{nbsp}(pass:[<]_return types_pass:[>])]

Let's look at each of these components:

FunctionName:: Defines the name of the function, which is used to call the function from a transaction (EOA), other contract, or from within the same contract. One function in each contract may be defined without a name, in which case it is the _fallback_ function, which is called when no other function is named. The fallback function cannot have any arguments or return anything.

parameters:: Following the name, we specify the arguments that must be passed to the function, with their names and types. In our +Faucet+ example we defined +uint withdraw_amount+ as the only argument to the +withdraw+ function.

The next set of keywords (public, private, internal, external) specify the function's _visibility_:

public:: Public is the default and such functions can be called by other contracts, EOA transactions or from within the contract. In our +Faucet+ example, both functions are defined as public.

external:: External functions are like public, except they cannot be called from within the contract, unless they are prefixed with the keyword +this+.

internal:: Internal functions are only "visible" within the contract, cannot be called by another contract or EOA transaction. They can be called by derived contracts (those that inherit them).

private:: Private functions are like internal functions but they cannot be called by derived contracts (that inherit them).

Keep in mind, the terms +internal+ and +private+ are somewhat misleading. Any function or data inside a contract is always _visible_ on the public blockchain, meaning that anyone can see the code or data. The keywords above only affect how and when a function can be _called_.

The next set of keywords (pure, constant, view, payable) affect the behavior of the function:

constant/view:: A function marked as a _view_, promises not to modify any state. The term _constant_ is an alias for _view_ that will be deprecated. At this time, the compiler does not enforce the _view_ modifier, only producing a warning, but this is expected to become an enforced keyword in v0.5 of Solidity.

pure:: A pure function is one that neither reads nor write any variables. It can only operate on arguments and return data, without reference to any stored data. Pure functions are intended to encourage declarative-style programming without side-effects or state.

payable:: A payable function is one that can accept incoming payments. Functions without payable will reject incoming payments, unless they originate in a coinbase (mining income) or as the destination of +SELFDESTRUCT+ (contract termination). In those cases, a contract cannot prevent incoming payments, due to design decisions in the EVM.

As you can see in our +Faucet+ example, we have one payable function (the fallback function), which is the only function that can receive incoming payments.

==== Contract constructor and selfdestruct

There is a special function that is only used once. When a contract is created, it also runs the _constructor function_ if one exists, to initialize the state of the contract. The constructor is run in the same transaction as the contract creation. The constructor function is optional. In fact, our +Faucet+ example has no constructor function.

Constructors can be specified in two ways. Up to Solidity v.0.4.21, the constructor is a function whose name matches the name of the contract:

[[old_constructor_style]]
.Constructor function prior to Solidity v0.4.22
----
contract MEContract {
	function MEContract() {
		// This is the constructor
	}
}
----

The difficulty with this format is that if the contract name is changed and the constructor function name is not changed, it is no longer a constructor. This can cause some pretty nasty, unexpected and difficult to notice bugs. Imagine for example if the constructor is setting the "owner" of the contract for purposes of control. Not only will it not set the owner at the time of contract creation, but may also be "callable" like a normal function, allowing any third party to hijack the contract and become the "owner" after contract creation.

To address the potential problems with constructor functions being based on having an identical name as the contract, Solidity v0.4.22 introduces a +constructor+ keyword that operates like a constructor function but does not have a name. Renaming the contract does not affect the constructor at all. Also, it is easier to identify which function is the constructor. It looks like this:

----
pragma ^0.4.22
contract MEContract {
	constructor () {
		// This is the constructor
	}
}
----

So, to summarize, a contract's lifecycle starts with a creation transaction from an EOA or other contract. If there is a constructor, it is called from the same creation transaction and can initialize the state of the contract as it is being created.

The other end of the contract's lifecycle is _contract destruction_. contracts are destroyed by a special EVM opcode called +SEFDESTRUCT+. It used to be name +SUICIDE+, but that name was deprecated due to the negative associations of the word. In Solidity, this opcode is exposed as a high level built-in function called +selfdestruct+, which takes one argument: the address to receive any balance remaining in the contract account. It looks like this:

----
selfdestruct(address recipient);
----

==== Adding a constructor and selfdestruct to our +Faucet+ example

The +Faucet+ example contract we introduced in <<intro>> does not have any constructor or selfdestruct functions. It is an eternal contract that cannot be removed from the blockchain. Let's change that, by adding a constructor and selfdestruct function. We probably want the selfdestruct to be callable _only_ by the EOA that originally created the contract. By convention, this is usually stored in an address variable, called +owner+. Our constructor sets the owner variable and the selfdestruct function will first check that the owner called it.

First our constructor:

----
// Version of Solidity compiler this program was written for
pragma solidity ^0.4.22;

// Our first contract is a faucet!
contract Faucet {

	address owner;

	// Initialize Faucet contract: set owner
	constructor() {
		owner = msg.sender;
	}

[...]
----

We've changed the pragma directive to specify v0.4.22 as the minimum version for this example, as we are using the new constructor keyword that only exists as of v.0.4.22 of Solidity. Our contract now has an +address+ type variable named +owner+. The name "owner" is not special in any way. We could call this address variable "potato" and still use it the same way. The name +owner+ simply makes the intention and purpose clear.

Then, our constructor, which runs as part of the contract creation transaction, assigns the address from +msg.sender+ to the +owner+ variable. We've used the +msg.sender+ in the +withdraw+ function to identify the origin of the withdrawal request. In the constructor however, the +msg.sender+ is the EOA or contract address that signed the contract creation transaction. We know this is the case _because_ this is a constructor function: it only runs once and only as a result of the contract creation transaction.

Ok, now we can add a function to destroy the contract. We need to make sure that only the owner can run this function, so we will use a +require+ statement to control access. Here's how it will look:

----
// Contract destructor
function destroy() public {
	require(msg.sender == owner);
	selfdestruct(owner);
}
----

If anyone other calls this +destroy+ function from an address other than +owner+, it will fail. But if the same address stored in +owner+ by the constructor calls it, the contract will selfdestruct and send any remaining balance to the +owner+ address.

==== Function modifiers

Solidity offers a special type of function which is called a _function modifier_. You apply modifiers to functions by adding the modifier name in the function declaration. Modifier functions are most often used to create conditions that apply to many functions within a contract. We have an access control statement already, in our +destroy+ function. Let's create a function modifier that expresses that condition:

[[function_modifier_onlyowner]]
.onlyOwner function modifier
----
modifier onlyOwner {
    require(msg.sender == owner);
    _;
}
----

In <<function_modifier_onlyowner>> we see the declaration of a function modifier, named +onlyOwner+. This function modifier sets a condition on any function that it modifies, requiring that the address stored as the +owner+ of the contract is the same as the address of the transaction's +msg.sender+. This is the basic design pattern for access control, allowing only the owner of a contract to execute any function that has the +onlyOwner+ modifier.

You may have noticed that our function modifier has a peculiar syntactic "placeholder" in it, an underscore followed by a semicolon (+pass:[_;]+). This placeholder is replaced by the code of the function that is being modified. Essentially, the modifier is "wrapped around" the modified function, placing its code in the location identified by the underscore character.

To apply a modifier, you add its name to the function declaration. More than one modifier can be applied to a function, as a comma-separated list, applied in the sequence they are declared.

Let's re-write our +destroy+ function to use the +onlyOwner+ modifier:

----
function destroy() public onlyOwner {
    selfdestruct(owner);
}
----

The function modifier's name (+onlyOwner+) is after the keyword +public+ and tells us that the +destroy+ function is modified by the +onlyOwner+ modifier. Essentially you can read this as "Only the owner can destroy this contract". In practice, the resulting code is equivalent to "wrapping" the code from +onlyOwner+ around +destroy+.

Function modifiers are an extremely useful tool because they allow us to write preconditions for functions and apply them consistently, making the code easier to read and, as a result, easier to audit for security problems. They are most often used for access control, as in the example <<function_modifier_onlyowner>>, but are quite versatile and can be used for a variety of other purposes.

Inside a modifier, you can access all the symbols (variables and arguments) visible to the modified function. In this case, we can access the +owner+ variable, which is declared within the contract. However, the inverse is not true: you cannot access any of the modifier's variables inside the modified function.

==== Contract inheritance

Solidity's contract object supports _inheritance_, which is a mechanism for extending a base contract with additional functionality. To use inheritance, specify a parent contract with the keyword +is+:

----
contract Child is Parent {
}
----

With this construct, the +Child+ contract inherits all the methods, functionality, and variables of +Parent+. Solidity also supports multiple inheritance, which can be specified by comma-separated contract names after the keyword +is+:

----
contract Child is Parent1, Parent2 {
}
----

Contract inheritance allows us to write our contracts in such as way as to achieve modularity, extensibility and reuse. We start with contracts that are simple and implement the most generic capabilities, then extend them by inheriting those capabilities in more specialized contracts.

In our +Faucet+ contract, we introduced the constructor and destructor, together with access control for an owner, assigned on construction. Those capabilities are quite generic: many contracts will have them. We can defined them as generic contracts, then use inheritance to extend them to the +Faucet+ contract.

We start by defining a base contract +owned+, which has an +owner+ variable, setting it in the contract's constructor:

----
contract owned {
	address owner;

	// Contract constructor: set owner
	constructor() {
		owner = msg.sender;
	}

	// Access control modifier
	modifier onlyOwner {
	    require(msg.sender == owner);
	    _;
	}
}
----

Next, we define a base contract +mortal+, which inherits +owned+:

----
contract mortal is owned {
	// Contract destructor
	function destroy() public onlyOwner {
		selfdestruct(owner);
	}
}
----

As you can see, the +mortal+ contract can utilize the +onlyOwner+ function modifier, defined in +owned+. It indirectly also uses the +owner+ address variable and the constructor defined in +owned+. Inheritance makes each contract simpler and focused on the specific functionality of its class, allowing us to manage the details in a modular way.

Now we can further extend the +owned+ contract, inheriting its capabilities in +Faucet+:

----
contract Faucet is mortal {
    // Give out ether to anyone who asks
    function withdraw(uint withdraw_amount) public {
        // Limit withdrawal amount
        require(withdraw_amount <= 100000000000000000);
        // Send the amount to the address that requested it
        msg.sender.transfer(withdraw_amount);
    }
    // Accept any incoming amount
    function () public payable {}
}
----

By inheriting +mortal+, which in turn inherits +owned+, the +Faucet+ contract now has the constructor and destroy functions, and a defined owner. The functionality is the same as when those functions were within +Faucet+, but now we can reuse those functions in other contracts without writing them again. Code re-use and modularity make our code cleaner, easier to read, and easier to audit.

==== Error handling (assert, require, revert)

A contract call can terminate and return an error. Error handling in Solidity is handled by four functions: +assert+, +require+, +revert+, and +throw+ (now deprecated).

When a contract terminates with an error, all the state changes (changes to variables, balances, etc.) are reverted, all the way up the chain of contract calls, if more than one contract were called. This ensures that transactions are atomic, meaning they either complete successfully or have no effect on state and are reverted entirely.

The +assert+ and +require+ functions operate in the same way, evaluating a condition and stopping execution with an error if the condition is false. By convention, +assert+ is used when the outcome is expected to be true, meaning that we use +assert+ to test internal conditions. By comparison, +require+ is used when testing inputs (such as function arguments or transaction fields), setting our expectations for those conditions.

We've used +require+ in our function modifier +onlyOwner+, to test that the message sender is the owner of the contract:

----
require(msg.sender == owner);
----

The +require+ function acts as a _gate condition_, preventing execution of the rest of the function and producing an error if it is not satisfied.

As of Solidity v.0.4.22, +require+ can also include a helpful text message, that can be used to show the reason for the error. The error message is recorded in the transaction log. So we can improve our code, by adding an error message in our +require+ function:

----
require(msg.sender == owner, "Only the contract owner can call this function");
----

The +revert+ and +throw+ functions, stops the execution of the contract and revert any state changes. The  +throw+ function is obsolete and will be removed in future versions of Solidity - you should use +revert+ instead. The +revert+ function can also take an error message as the only argument, which is recorded in the transaction log.

Certain conditions in a contract will generate errors regardless of whether we explicitly check for them. For example, in our +Faucet+ contract, we don't check whether there is enough ether to satisfy a withdrawal request. That's because the +transfer+ function will fail with an error and revert the transaction if there is insufficient balance to make the transfer:

.The transfer function will fail if there is an insufficient balance
----
msg.sender.transfer(withdraw_amount);
----

However, it might be better to check explicitly and provide a clear error message on failure. We can do that by adding a require statement before the transfer:

----
require(this.balance >= withdraw_amount,
	"Insufficient balance in faucet for withdrawal request");
msg.sender.transfer(withdraw_amount);
----

Additional error checking code like this will increase gas consumption slightly, but it offers better error reporting than if omitted. Striking the right balance between gas consumption and verbose error checking is something you will need to decide based on the expected use of your contract. In the case of a +Faucet+ intended for a testnet, we'd probably err on the side of extra reporting even if it costs more gas. Perhaps for a mainnet contract we'd choose to be frugal with our gas usage instead.


==== Events

Events are Solidity constructs that facilitate the production of transaction logs. When a transaction completes (successfully or not), it produces a _transaction receipt_, as we will see in <<evm>>. The transaction receipt contains _log_ entries that provide information about the actions that occurred during the execution of the transaction. Events are the Solidity high-level objects that are used to construct these logs.

Events are especially useful in lightweight clients and DApps, which can "watch" for specific events and report them to the user-interface, or make a change in the state of the application to reflect an event in an underlying contract.

Event objects take arguments that are serialized and recorded in the transaction logs, in the blockchain. You can supply the keyword +indexed+, before an argument, to make the value part of an indexed table (hash table) that can be searched or filtered by an application.

We have not added any events in our +Faucet+ example, so far, so let's do that. We will add two events, one to log any withdrawals and one to log any deposits. We will call these events +Withdrawal+ and +Deposit+ respectively. First, we define the events, in the +Faucet+ contract:

----
contract Faucet is mortal {
	event Withdrawal(address indexed to, uint amount);
	event Deposit(address indexed from, uint amount);

	[...]
}
----

We've chosen to make the addresses +indexed+, to allow searching and filtering in any user interface built to access our +Faucet+.

Next, we use the +emit+ keyword to incorporate the event data in the transaction logs:

----
// Give out ether to anyone who asks
function withdraw(uint withdraw_amount) public {
    [...]
    msg.sender.transfer(withdraw_amount);
    emit Withdrawal(msg.sender, withdraw_amount);
}
// Accept any incoming amount
function () public payable {
    emit Deposit(msg.sender, msg.value);
}
----

The resulting +Faucet.sol+ contract looks like this:

[[Faucet8.sol]]
.Faucet8.sol: Revised Faucet contract, with events
[source,solidity,linenums]
----
include::code/Solidity/Faucet8.sol['code/Solidity/Faucet8.sol']
----


===== Catching Events

Ok, so we've setup our contract to emit events. How do we see the results of a transaction and "catch" the events? The +web3.js+ library provides a data structure as the result of a transaction that contains the transaction logs. Within those we can see the events generated by the transaction.

Let's use +truffle+ to run a test transaction on the revised +Faucet+ contract. Follow the instructions in <<truffle>> to setup a project directory and compile the +Faucet+ code. The source code can be found in the book's GitHub repository under:

----
code/Solidity/FaucetEvents
----


[[testing_events_prep]]
----
$ truffle develop
truffle(develop)> compile
truffle(develop)> migrate
Using network 'develop'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0xb77ceae7c3f5afb7fbe3a6c5974d352aa844f53f955ee7d707ef6f3f8e6b4e61
  Migrations: 0x8cdaf0cd259887258bc13a92c0a6da92698644c0
Saving successful migration to network...
  ... 0xd7bc86d31bee32fa3988f1c1eabce403a1b5d570340a3a9cdba53a472ee8c956
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying Faucet...
  ... 0xfa850d754314c3fb83f43ca1fa6ee20bc9652d891c00a2f63fd43ab5bfb0d781
  Faucet: 0x345ca3e014aaf5dca488057592ee47305d9b3e10
Saving successful migration to network...
  ... 0xf36163615f41ef7ed8f4a8f192149a0bf633fe1a2398ce001bf44c43dc7bdda0
Saving artifacts...

truffle(develop)> Faucet.deployed().then(i => {FaucetDeployed = i})
truffle(develop)> FaucetDeployed.send(web3.toWei(1, "ether")).then(res => { console.log(res.logs[0].event, res.logs[0].args) })
Deposit { from: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  amount: BigNumber { s: 1, e: 18, c: [ 10000 ] } }
truffle(develop)> FaucetDeployed.withdraw(web3.toWei(0.1, "ether")).then(res => { console.log(res.logs[0].event, res.logs[0].args) })
Withdrawal { to: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  amount: BigNumber { s: 1, e: 17, c: [ 1000 ] } }

----

After getting the deployed contract, using the +deployed()+ function, we execute two transactions. The first transaction is a deposit (using +send+), which emits a +Deposit+ event in the transaction logs:

----
Deposit { from: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  amount: BigNumber { s: 1, e: 18, c: [ 10000 ] } }
----

Next, we use the +withdraw+ function to make a withdrawal. This emits a +Withdrawal+ event:

----
Withdrawal { to: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  amount: BigNumber { s: 1, e: 17, c: [ 1000 ] } }
----

To get these events, we looked at the +logs+ array returned as a result (+res+) of the transactions. The first log entry (+logs[0]+) contains an event name in +logs[0].event+ and the event arguments in +logs[0].args+. By showing these on the console, we can see the emitted event name and the event arguments.

Events are a very useful mechanism, not only for intra-contract communication, but also for debugging during development.

==== Calling other contracts (call, send, delegatecall, callcode)

Calling other contracts from within your contract is a very useful but potentially dangerous operation. We'll examine the various ways you can achieve this and evaluate the risks of each method.

===== Creating a new instance

The safest way to call another contract is if you create that other contract yourself. That way, you are certain of its interfaces and behavior. To do this, you can simply instantiate it, using the keyword +new+, as with any object-oriented language. In Solidity, the keyword +new+ will create the contract on the blockchain and return an object that you can use to reference it. Let's say you want to create and call a +Faucet+ contract, from within another contract called +Token+:


----
contract Token is mortal {
	Faucet _faucet;

	constructor() {
		_faucet = new Faucet();
	}
}
----

This mechanism for contract construction ensures that you know the exact type of contract and its interface. The contract +Faucet+ must be defined within the scope of +Token+, which you can do with an +import+ statement, if the definition is in another file:

----
import "Faucet.sol"

contract Token is mortal {
	Faucet _faucet;

	constructor() {
		_faucet = new Faucet();
	}
}
----

The +new+ keyword can also accept optional parameters to specify the +value+ of ether transfer on creation, and arguments passed to the new contract's constructor, if any:

----
import "Faucet.sol"

contract Token is mortal {
	Faucet _faucet;

	constructor() {
		_faucet = (new Faucet).value(0.5 ether)();
	}
}
----

If we endow the created +Faucet+ with some ether, we can also then call the +Faucet+ functions, which operate just like a method call. In this example, we call the +destroy+ function of +Faucet+, from within the +destroy+ function of +Token+:

----
import "Faucet.sol"

contract Token is mortal {
	Faucet _faucet;

	constructor() {
		_faucet = (new Faucet).value(0.5 ether)();
	}

	function destroy() ownerOnly {
		_faucet.destroy();
	}
}
----

===== Addressing an existing instance

Another way we can use to call a contract, is to cast the address of an existing instance of the contract. With this method, we apply a known interface to an existing instance. It is therefore critically important that we know, for sure, that the instance we are addressing is in fact of the same type as we assume. Let's look at an example:

----
import "Faucet.sol"

contract Token is mortal {

	Faucet _faucet;

	constructor(address _f) {
		_faucet = Faucet(_f);
		_faucet.withdraw(0.1 ether)
	}
}
----

Here, we take an address provided as an argument to the constructor and we cast it as a +Faucet+ object. This is much riskier than the previous mechanism, because we don't in fact know whether that address is in fact a +Faucet+ object. When we call +withdraw+, we are assuming that it accepts the same arguments and executes the same code as our +Faucet+ declaration, but we can't be sure. For all we know, the +withdraw+ function at this address could execute something completely different from what we expect, even if it is named the same. Using addresses passed as input and casting them into specific objects is therefore much more dangerous than creating the contract ourselves.

===== Raw call, delegatecall

Solidity offers some even more "low-level" functions for calling other contracts. These correspond directly to EVM opcodes of the same name and allow us to construct a contract-to-contract call manually. As such, they represent the most flexible *and* the most dangerous mechanisms for calling other contracts.

Here's the same example, using a +call+ method:

----
contract Token is mortal {
	constructor(address _faucet) {
		_faucet.call("withdraw", 0.1 ether);
	}
}
----

As you can see, this type of +call+, is a _blind_ call into a function, very much like constructing a raw transaction, only from within a contract's context. It can expose our contract to a number of security risks, most importantly _reentrancy_, which we will talk about in more detail in <<reentrancy>>. The +call+ function will return false if there is a problem, so we can evaluate the return value, for error handling:

----
contract Token is mortal {
	constructor(address _faucet) {
		if !(_faucet.call("withdraw", 0.1 ether)) {
			revert("Withdrawal from faucet failed");
		}
	}
}
----

Another variant of +call+ is +delegatecall+, which replaced the more dangerous +callcode+. The +callcode+ method will be deprecated soon, so it should not be used.

As mentioned in <<solidity_address_object>>, a +delegatecall+ is different from a +call+, in that the +msg+ context does not change. For example, whereas a +call+ changes the value of +msg.sender+ to be the calling contract, a +delegatecall+ keeps the same +msg.sender+ as it is in the calling contract. Essentially, +delegatecall+ runs the code of another contract inside the context of the current contract. It is most often used to invoke code from a +library+.

The +delegate+ call should be used with great caution. It can have some unexpected effects, especially if the contract you call was not designed as a library.

Let's use an example contract, to demonstrate the various call semantics used by +call+ and +delegatecall+ for calling libraries and contracts. We use an event to log the origin of each call and see how the calling context changes depending on the call type:

[[call_examples_code]]
.CallExamples.sol: An example of different call semantics.
[source,solidity,linenums]
----
include::code/truffle/CallExamples/contracts/CallExamples.sol["code/truffle/CallExamples/contracts/CallExamples.sol"]
----

Our main contract is +caller+, which calls a library +calledLibrary+ and a contract +calledContract+. Both the called library and contract have identical functions +calledFunction+, which emit an event +calledEvent+. The event +calledEvent+ logs three pieces of data: +msg.sender+, +tx.origin+, and +this+. Each time +calledFunction+ is called it may have a different execution context (different values for example in +msg.sender+), depending on whether it is called direclty or through +delegatecall+.

In +caller+, we first call the contract and library directly, by invoking the +calledFunction+ in each. Then, we explicitly use the low-level functions +call+ and +delegatecall+ to call the +calledContract.calledFunction+. This way we can see how the various calling mechanisms behave.

Let's run this in a truffle development environment and capture the events, to see how it looks:

----
truffle(develop)> migrate
Using network 'develop'.
[...]
Saving artifacts...
truffle(develop)> web3.eth.accounts[0]
'0x627306090abab3a6e1400e9345bc60c78a8bef57'
truffle(develop)> caller.address
'0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f'
truffle(develop)> calledContract.address
'0x345ca3e014aaf5dca488057592ee47305d9b3e10'
truffle(develop)> calledLibrary.address
'0xf25186b5081ff5ce73482ad761db0eb0d25abfbf'
truffle(develop)> caller.deployed().then( i => { callerDeployed = i })

truffle(develop)> callerDeployed.make_calls(calledContract.address).then(res => { res.logs.forEach( log => { console.log(log.args) })})
{ sender: '0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f',
  origin: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  from: '0x345ca3e014aaf5dca488057592ee47305d9b3e10' }
{ sender: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  origin: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  from: '0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f' }
{ sender: '0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f',
  origin: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  from: '0x345ca3e014aaf5dca488057592ee47305d9b3e10' }
{ sender: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  origin: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
  from: '0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f' }

----

Let's see what happened here. We called the +make_calls+ function and passed the address of +calledContract+, then caught the four events emitted by each of the different calls. Look at the +make_calls+ function and let's walk through each step.

The first call is:

----
_calledContract.calledFunction();
----

Here, we're calling the +calledContract.calledFunction+ directly, using the high-level ABI for +calledFunction+. The event emitted is:

----
sender: '0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f',
origin: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
from: '0x345ca3e014aaf5dca488057592ee47305d9b3e10'
----

As you can see, +msg.sender+ is the address of the +caller+ contract. The +tx.origin+ is the address of our wallet +web3.eth.accounts[0]+ that sent the transaction to +caller+. The event was emitted by +calledContract+, as we can see from the last argument in the event.

The next call in +make_calls+, is to the library:

----
calledLibrary.calledFunction();
----

It looks identical to how we called the contract, but behaves *very* differently. Let's look at the second event emitted:

----
sender: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
origin: '0x627306090abab3a6e1400e9345bc60c78a8bef57',
from: '0x8f0483125fcb9aaaefa9209d8e9d7b9c8b9fb90f'
----

This time, the +msg.sender+ is not the address of +caller+. Instead it is the address of our wallet, and is the same as the transaction origin. That's because when you call a library, the call is always +delegatecall+ and runs within the context of the caller. So, when +calledLibrary+ code was running, it inherited the execution context of +caller+, as if its code was running inside +caller+. The variable +this+ (shown as +from+ in the event emitted) is the address of +caller+, even though it is accessed from within +calledLibrary+.

The next two calls, using the low-level +call+ and +delegatecall+, verify our expectations, emitting events that mirror what we just saw above.

[[gas_sec]]
=== Gas considerations

Gas is described in more in detail in the <<gas>> section and is an incredibly important consideration in smart contract programming. Gas is a resource constraining the maximum amount of computation that Ethereum will allow a transaction to consume. If the gas limit is exceeded during computation, the following series of events occurs:

* An "out of gas" exception is thrown.
* The state of the contract prior to the function's execution is restored (reverted).
* The entire amount of the gas is given to the miner as a transaction fee, it is *not* refunded.

Because gas is paid by the user who creates that transaction, users are discouraged from calling functions that have a high gas cost. It is thus in the programmer's best interest to minimize the gas cost of a contract's functions. To this end, there are certain practices that are recommended when constructing smart contracts, so as to minimize the gas costs surrounding a function call.

==== Avoid dynamically-sized arrays

Any loop through a dynamically sized array wherein a function performs operations on each element or searches for a particular element introduces the risk of using too much gas. The contract may run out of gas before finding the desired result, or before acting on every element.

==== Avoid calls to other contracts

Calling other contracts, especially when the gas cost of their functions is not known, introduces the risk of running out of gas. Avoid using libraries that are not well tested and broadly used. The less scrutiny a library has received from other programmers, the greater the risk of using it.

==== Estimating gas cost

In case that you need to estimate the gas necessary to execute a certain method of a contract considering its call arguments, you could, for instance, use the following procedure;

[source, JavaScript]
var contract = web3.eth.contract(abi).at(address);
var gasEstimate = contract.myAweSomeMethod.estimateGas(arg1, arg2, {from: account});

*gasEstimate* will tell us the number of gas units needed for its execution.

To obtain the *gas price* from the network you can use;

[source, JavaScript]
var gasPrice = web3.eth.getGasPrice();

And from there, estimate the *gas cost*;

[source, JavaScript]
var gasCostInEther = web3.fromWei((gasEstimate * gasPrice), 'ether');

Let's apply our gas estimation functions to estimating the gas cost of our +Faucet+ example, using the code from the book's repository found here:

----
code/truffle/FaucetEvents
----

We start truffle in development mode, and execute a JavaScript file +gas_estimates.js+, which contains:

[source, JavaScript]
.gas_estimates.js: Using the estimateGas function
----
var FaucetContract = artifacts.require("./Faucet.sol");

FaucetContract.web3.eth.getGasPrice(function(error, result) {
    var gasPrice = Number(result);
    console.log("Gas Price is " + gasPrice + " wei"); // "10000000000000"

    // Get the contract instance
    FaucetContract.deployed().then(function(FaucetContractInstance) {

        // Use the keyword 'estimateGas' after the function name to get the gas estimation for this particular function (aprove)
		FaucetContractInstance.send(web3.toWei(1, "ether"));
        return FaucetContractInstance.withdraw.estimateGas(web3.toWei(0.1, "ether"));

    }).then(function(result) {
        var gas = Number(result);

        console.log("gas estimation = " + gas + " units");
        console.log("gas cost estimation = " + (gas * gasPrice) + " wei");
        console.log("gas cost estimation = " + FaucetContract.web3.fromWei((gas * gasPrice), 'ether') + " ether");
    });
});
----

Here's how that looks in the truffle development console:

----
$ truffle develop

truffle(develop)> exec gas_estimates.js
Using network 'develop'.

Gas Price is 20000000000 wei
gas estimation = 31397 units
gas cost estimation = 627940000000000 wei
gas cost estimation = 0.00062794 ether
----

It is recommended that you evaluate the gas cost of functions as part of your development workflow, to avoid any surprises when deploying contracts to the mainnet.


[[security_sec]]
=== Security considerations

Security is one of the most important considerations when writing smart contracts. As with other programs, a smart contract will execute exactly what is written, which is not always what the programmer intended. Furthermore, all smart contracts are public and any user can interact with them simply by creating a transaction. Any vulnerability can be exploited and losses are almost always impossible to recover.

In the field of smart-contract programming, mistakes are costly and easily exploited. It is therefore critical to follow best practices and use well tested design patterns.

_Defensive programming_ is a style of programming that is particularly well suited to programming smart contracts and has the following characteristics:

Minimalism/Simplicity:: Complexity is the enemy of security. The simpler the code, the less it does, the lower the chance of a bug or unforeseen effect. When first engaging in smart-contract programming, developers are tempted to try to write a lot of code. Instead, you should look through your smart-contract code and try to find ways to do less, with fewer lines of code, with less complexity and with fewer "features". If someone tells you that their project has produced "thousands of lines of code", you should question the security of that project. Simpler is more secure.

Code re-use:: As much as possible, try not to "reinvent the wheel". If a library or contract already exists that does most of what you need, re-use it. Within your own code, follow the DRY principle: Do not Repeat Yourself. If you see any snippet of code repeat more than once, ask yourself whether it could be written as a function or library and re-used. Code that has been extensively used and tested is likely more secure than any new code you write. Beware of Not-Invented-Here attitude, where you are tempted to "improve" a feature or component by building it from scratch. The security risk is often greater than the improvement value.

Code quality:: Smart-contract code is unforgiving. Every bug can lead to monetary loss. You should not treat smart-contract programming the same way as general purpose programming. Rather, you should apply rigorous engineering and software development methodologies, akin to aerospace engineering or a similarly unforgiving engineering discipline. Once you "launch" your code, there's little you can do to fix any problems.

Readability/Auditability:: Your code should be easy to comprehend and clear. The easier it is to read, the easier it is to audit. Smart contracts are public, as anyone can reverse engineer the bytecode. Therefore, you should develop your work in public, using collaborative and open source methodologies. You should write code that is well documented and easy to read, following the style conventions and naming conventions that are part of the Ethereum community.

Test coverage:: Test everything that you can test. Smart contracts run in a public execution environment, where anyone can execute them with whatever input they want. You should never assume that input, such as function arguments, is well formed, properly bounded and has a benign purpose. Test all arguments to make sure they are within expected ranges and properly formatted.

==== Common security risks

Smart contract programmers should be familiar with many of the most common security risks, so as to be able to detect and avoid the programming patterns that leave them exposed to these risks.

===== Re-entrancy

Re-entrancy is a phenomenon in programming in which a function or program is interrupted and then called again before its previous invocations have finished. In the context of smart contract programming, re-entrancy can occur when contract A calls a function in contract B, which in turn calls the same function in contract A, leading to a recursive execution. This can be particularly dangerous in a situation where the state of the contract is not updated until after the critical call is finished.

To understand this, imagine a withdrawal by a wallet contract calling a bank contract. Contract A calls the withdraw function in contract B, attempting to withdraw amount X. This scenario would involve the following actions:

1. Contract B checks whether A has the necessary balance to withdraw X
2. B transfers X to the address of A (runs A's payable fallback function)
3. B updates A's balance to reflect the withdrawal

Whenever a payment is sent to a contract, as in this example, the recipient contract (A) has the opportunity to execute a _payable_ function, such as the default fallback function. However, malicious attackers can take advantage of this execution. Imagine that in A's payable fallback function, contract A calls bank B's withdraw function _again_. The withdrawal function of B will now experience re-entrancy, as the same initial transaction is now causing a circular call.

"(1) A calls B which (2) calls A's payable function which (1) calls B again"

In the second iteration of B's withdrawal function, B will again check if A has the available balance. Since step 3 (which updates A's balance) has yet to be executed, it will appear to B that A still has the available funds to withdraw, no matter how many times this function is re-invoked. This cycle can be repeated as long as there is gas available to keep running. When A detects that gas is running low, it can stop calling B in the payable function. B will finally execute step 3, deducting X from A's balance. Yet, by this point, B may have executed hundreds of transfers and only deducted the amount once. A has effectively drained funds from B with this attack.

This exploit is particularly famous because of its relevance in the DAO attack. A user took advantage of the fact that the balance in a contract was changed after a call to transfer funds was made and withdrew millions of dollars worth of ether.

To guard against re-entrancy, the best practice is for a programmer to use the _Checks-Effects-Interactions_ pattern, wherein the effects of a function call (such as decreasing the balance) occur before making the call. In our example, this would mean switching steps 3 and 2: updating a user's balance before transfer.

In Ethereum, this is perfectly okay, because all effects of a transaction are atomic, meaning it is impossible for the balance to update without the user also being paid out. Either both occur, or an exception is thrown and neither occurs. This guards against re-entrancy attacks because all subsequent calls into the original withdrawal function will encounter the correct modified balance. By switching the two steps, A would be prevented from withdrawing more than their balance.

====== Delegatecall

The call method, as mentioned previously, "calls" into a function from within the calling contract's context.


////
TODO: expand Delegate Call section ^
////


====== Overflow and Underflow

////
TODO: expand
////



[[design_patterns_sec]]
==== Design Patterns

Software developers of any programming paradigm generally experience reoccurring design challenges centered around the topics of behavior, structure, interaction, and creation. Often these problems can be generalized and re-applied to future problems of a similar nature. When given a formal structure, these generalizations are called *Design Patterns*. Smart contracts have their own set of reoccurring design problems that can be solved using some of the patterns described below.

There is an endless number of design problems in the development of smart contracts, making it impossible to discuss all of them
here. For that reason, this section will focus on three of the most pervasive problem classifications in smart contract design: *access control*, *state flow*, and *fund disbursement*.

Throughout this section, we will be working on a contract that will ultimately incorporate all three of these design patterns. This contract will run a voting system that allows users to vote on "truth". The contract will suggest a claim such as "The Cubs won the World Series." or "It is raining in New York City" and then users will have
the opportunity to vote either true or false. The contract will consider the proposition as true if the majority of participants voted for true and likewise if the majority of participants voted for false. To incentivize truthfulness, every vote must send 100 ether to the contract and the funds contributed by the losing minority will be split up amongst the majority. Every participant in the majority will receive their portion of winnings from the minority as well as their initial investment.

This "truth voting" system is actually the foundation of Gnosis, a forecasting tool built on top of Ethereum. More information about Gnosis can be found here: https://gnosis.pm/

[[access_control_sec]]
===== Access control

Access control restricts which users may call contract functions. For the example, the owner of the truth voting contract may decide to limit those who can participate in the vote.
To accomplish this the contract must impose two access restrictions:

. Only an owner of the contract may add new users to the list of "allowed voters"
. Only allowed voters may cast a vote

Solidity function modifiers offer a concise way to implement these restrictions.

_Note: The following example uses an underscore semicolon within the modifier bodies. This is a Solidity feature used to tell the compiler when to run the modified function's body. A developer can act as if the modified function's body will be copied to the position of the underscore._
[source,solidity]
----
pragma solidity ^0.4.21;

contract TruthVote {

    address public owner = msg.sender;

    address[] true_votes;
    address[] false_votes;
    mapping (address => bool) voters;
    mapping (address => bool) hasVoted;

    uint VOTE_COST = 100;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier onlyVoter() {
        require(voters[msg.sender] != false);
        _;
    }

    modifier hasNotVoted() {
        require(hasVoted[msg.sender] == false);
        _;
    }

    function addVoter(address voter)
        public
        onlyOwner()
    {
        voters[voter] = true;
    }

    function vote(bool val)
        public
        payable
        onlyVoter()
        hasNotVoted()
    {
        if (msg.value >= VOTE_COST) {
            if (val) {
                true_votes.push(msg.sender);
            } else {
                false_votes.push(msg.sender);
            }
            hasVoted[msg.sender] = true;
        }
    }
}
----
*Description of Modifiers and Functions:*

- *onlyOwner*: this modifier can decorate a function such that the function will then only be callable by a sender with an address that matches that of *owner*.
- *onlyVoter*: this modifier can decorate a function such that the function will then only be callable by a registered voter.
- *addVoter(voter)*: this function is used to add a voter to the list of voters. This function uses the *onlyOwner* modifier so only the owner of this contract may call it.
- *vote(val)*: this function is used by a voter to vote either true or false to the presented proposition. It is decorated with the *onlyVoter* modifier so only registered voters may call it.

[[state_flow_sec]]
===== State flow

Many contracts will require some notion of operation state. The state of a contract will determine how the contract will behave and what operations it offers
at a given point in time. Let's return to our truth voting system for a more concrete example.

The operation of our voting system can be broken down into 3 distinct states.

. *Register*: The service has been created and the owner can now add voters.
. *Vote*:  All voters cast their votes.
. *Disperse*: Vote payments are divided and sent to the majority participants.

The following code continues to build on the access control code, but further restricts functionality to specific states.
In Solidity, it is commonplace to use enumerated values to represent states.

[source,solidity]
----
pragma solidity ^0.4.21;

contract TruthVote {
    enum States {
        REGISTER,
        VOTE,
        DISPERSE
    }

    address public owner = msg.sender;

    uint voteCost;

    address[] trueVotes;
    address[] falseVotes;


    mapping (address => bool) voters;
    mapping (address => bool) hasVoted;

    uint VOTE_COST = 100;

    States state;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier onlyVoter() {
        require(voters[msg.sender] != false);
        _;
    }

    modifier isCurrentState(States _stage) {
        require(state == _stage);
        _;
    }

    modifier hasNotVoted() {
        require(hasVoted[msg.sender] == false);
        _;
    }

    function startVote()
        public
        onlyOwner()
        isCurrentState(States.REGISTER)
    {
        goToNextState();
    }

    function goToNextState() internal {
        state = States(uint(state) + 1);
    }

    modifier pretransition() {
        goToNextState();
        _;
    }

    function addVoter(address voter)
        public
        onlyOwner()
        isCurrentState(States.REGISTER)
    {
        voters[voter] = true;
    }

    function vote(bool val)
        public
        payable
        isCurrentState(States.VOTE)
        onlyVoter()
        hasNotVoted()
    {
        if (msg.value >= VOTE_COST) {
            if (val) {
                trueVotes.push(msg.sender);
            } else {
                falseVotes.push(msg.sender);
            }
            hasVoted[msg.sender] = true;
        }
    }

    function disperse(bool val)
        public
        onlyOwner()
        isCurrentState(States.VOTE)
        pretransition()
    {
        address[] memory winningGroup;
        uint winningCompensation;
        if (trueVotes.length > falseVotes.length) {
            winningGroup = trueVotes;
            winningCompensation = VOTE_COST + (VOTE_COST*falseVotes.length) / trueVotes.length;
        } else if (trueVotes.length < falseVotes.length) {
            winningGroup = falseVotes;
            winningCompensation = VOTE_COST + (VOTE_COST*trueVotes.length) / falseVotes.length;
        } else {
            winningGroup = trueVotes;
            winningCompensation = VOTE_COST;
            for (uint i = 0; i < falseVotes.length; i++) {
                falseVotes[i].transfer(winningCompensation);
            }
        }

        for (uint j = 0; j < winningGroup.length; j++) {
            winningGroup[j].transfer(winningCompensation);
        }
    }
}
----

*Description of Modifiers and Functions:*

- *isCurrentState*: this modifier will require that the contract is in a specified state before continuing execution of the decorated function.
- *pretransition*: this modifier will transition to the next state before executing the rest of the decorated function
- *goToNextState*: function that transitions the contract to the next state
- *disperse*: function that calculates the majority and disperses winnings accordingly. Only the owner may call this function to officially close voting.
- *startVote*: function that the owner can use to start a vote.

It may be important to note that allowing the owner to close the voting process at will opens this contract up to abuse. In a more genuine implementation, the voting period should close after a publicly understood period of time. For the sake of this example, this is fine.

The additions made now ensure that voting is only allowed when the owner decides to start the voting period, users can only be registered by the owner before the vote happens, and funds are only dispersed after the vote closes.

[[withdraw_sec]]
===== Withdraw

Many contracts will offer some way for a user to retrieve money from it. In our working example, users of the majority are sent money directly when the contract
begins dispersing funds. Although this appears to work, it is an under-thought solution. The receiving address of the *addr.send()* call in *disperse* could be a contract that
has a fallback function which fails and consequently breaks *disperse*. This effectively stops all further majority participants from receiving their earning.
A better solution is to provide a withdraw function that a user can call to collect their earnings.

[source,solidity]
----
...

enum States {
    REGISTER,
    VOTE,
    DETERMINE,
    WITHDRAW
}

mapping (address => bool) votes;
uint trueCount;
uint falseCount;

bool winner;
uint winningCompensation;

modifier posttransition() {
    _;
    goToNextState();
}

function vote(bool val)
    public
    onlyVoter()
    isCurrentStage(State.VOTE)
{
    if (votes[msg.sender] == address(0) && msg.value >= VOTE_COST) {
        votes[msg.sender] = val;
        if (val) {
            trueCount++;
        } else {
            falseCount++;
        }
    }
}

function determine(bool val)
    public
    onlyOwner()
    isCurrentState(State.VOTE)
    pretransition()
    posttransition()
{
    if (trueCount > falseCount) {
        winner = true;
        winningCompensation = VOTE_COST + (VOTE_COST*false_votes.length) / true_votes.length;
    } else if (falseCount > trueCount) {
        winner = false;
        winningCompensation = VOTE_COST + (VOTE_COST*true_votes.length) / false_votes.length;
    } else {
        winningCompensation = VOTE_COST;
    }
}

function withdraw()
    public
    onlyVoter()
    isCurrentState(State.WITHDRAW)
{
    if (votes[msg.sender] != address(0)) {
        if (votes[msg.sender] == winner) {
            msg.sender.transfer(winningCompensation);
        }
    }
}

...
----

*Description of Modifiers and (Updated) Functions:*

- *posttransition*: transitions to the next state after the function call
- *determine*: this function is very similar to the previous *disperse* function except it now just calculates the winner and winning compensation and does not actually send any funds.
- *vote*: votes are now added to the votes mapping and true/false counters are incremented.
- *withdraw*: allows a voter to collect winnings (if any).



This way, if the send fails, it will only fail on the specific caller's case and not hinder all other user's ability to collect their winnings.

[[modularity_and_side_effects_sec]]
==== Modularity and side effects

////
TODO: add paragraph
////

[[contract_libraries_sec]]
==== Contract libraries

Github link: https://github.com/ethpm

Repository link: https://www.ethpm.com/registry

Website: https://www.ethpm.com/

Documentation: https://www.ethpm.com/docs/integration-guide

[[security_best_practices_sec]]
==== Security best practices

Github: https://github.com/ConsenSys/smart-contract-best-practices/

Docs: https://consensys.github.io/smart-contract-best-practices/

https://blog.zeppelin.solutions/onward-with-ethereum-smart-contract-security-97a827e47702

https://medium.com/zeppelin-blog/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05#.cox40d2ut

Perhaps the most fundamental software security principle is to maximize reuse of trusted code. In blockchain technologies, this is even condensed in an adage: "Do not roll your own crypto". In the case of smart contracts, this amounts to profiting as much as possible from freely available libraries that have been thoroughly vetted by the community.

In Ethereum, the most widely used solution is the https://openzeppelin.org/[OpenZeppelin] suite, an ample library of contracts ranging from implementations of `ERC20` and `ERC721` tokens, to many flavors of crowdsale models, to simple behaviors commonly found in contracts such as `Ownable`, `Pausable` or `LimitBalance`. The contracts in this repository have been extensively tested and in some cases even function as _de facto_ standard implementations. They are free to use, and are built and mantained by https://zeppelin.solutions[Zeppelin] together with an ever growing list of external contributors.

Also from Zeppelin is https://zeppelinos.org/[zeppelin_os], an open source platform of services and tools to develop and manage smart contract applications securely. zeppelin_os provides a layer on top of the EVM that makes it easy for developers to launch upgradeable DApps linked to an on-chain library of well tested contracts that are themselves upgradeable. Different versions of these libraries can coexist in the blockchain, and a vouching system allows users to propose or push improvements in different directions. A set of off-chain tools to debug, test, deploy and monitor decentralized applications is also provided by the platform.


[[further_reading_sec]]
==== Further reading

The Application Binary Interface (ABI) is strongly typed, known at compilation time and static. All contracts have the interface definitions of any contracts they intend to call available at compile-time.

A more rigorous and in-depth explanation of the Ethereum ABI can be found at
`https://solidity.readthedocs.io/en/develop/abi-spec.html`.
The link includes details about the formal specification of encoding and various helpful examples.

[[deploying_smart_contracts_sec]]
=== Deploying smart contracts

////
TODO: add paragraph
////

[[testing_frameworks]]
=== Testing smart contracts

////
TODO: add paragraph
////


[[testing_frameworks_sec]]
==== Testing Frameworks

There are several commonly-used test frameworks (in no particular order):

Truffle Test:: Part of the Truffle framework, Truffle allows for unit tests to be written in JavaScript (Mocha based) or Solidity. These tests are run against TestRPC/Ganache. More details on writing these tests are located at <<truffle>>

////
TODO: add anchor for <<truffle>>
////

Embark Framework Testing:: Embark integrates with Mocha to run unit tests written in JavaScript. The tests are in turn run against contracts deployed on TestRPC/Ganache. The Embark Framework automatically deploys smart contracts and will automatically redeploy the contracts when they are changed. It also keeps track of deployed contracts and deploys contracts when truly needed. Embark includes a testing library to rapidly run and test your contracts in an EVM, with functions like ```assert.equal()```. ```embark test``` will run any test files under directory test/.

DApp:: DApp uses native Solidity code (a library called ds-test) and a Parity built Rust library called Ethrun to execute Ethereum bytecode and then assert correctness. The ds-test library provides assertion functions for validating correctness and events for logging data in the console.

Assertions Functions includes
....
assert(bool condition)
assertEq(address a, address b)
assertEq(bytes32 a, bytes32 b)
assertEq(int a, int b)
assertEq(uint a, uint b)
assertEq0(bytes a, bytes b)
expectEventsExact(address target)
....

Logging Events will log information to the console, making them useful for debugging.
....
logs(bytes)
log_bytes32(bytes32)
log_named_bytes32(bytes32 key, bytes32 val)
log_named_address(bytes32 key, address val)
log_named_int(bytes32 key, int val)
log_named_uint(bytes32 key, uint val)
log_named_decimal_int(bytes32 key, int val, uint decimals)
log_named_decimal_uint(bytes32 key, uint val, uint decimals)
....

Populus:: Populus uses python and its own chain emulator to run contracts written in Solidity. Unit tests are written in Python with the pytest library. Populus supports writing contracts that are specifically for testing. These contract filenames should match the glob pattern ```Test*.sol``` and be located anywhere under the project tests directory ```./tests/```.

|=======
|Framework | Test Language(s)    | Testing Framework | Chain Emulator       | Website
|Truffle   | Javascript/Solidity | Mocha             | TestRPC/Ganache      | truffleframework.com
|Embark    | Javascript          | Mocha             | TestRPC/Ganache      | embark.readthedocs.io
|DApp      | Solidity            | ds-test (custom)  | Ethrun (Parity)      | dapp.readthedocs.io
|Populus   | Python              | Pytes             | Python chain emulator| populus.readthedocs.io
|=======

=======
If you this is your first time using geth, it might take a while to sync up to the network.
Then set up your variables with:
----
> var foo = eth(<CONTENTS_OF_ABI_FILE>)
> var byteCode = '0x<CONTENTS_OF_BIN_FILE>)
----
Fill in the parameters with the outputs from the more commands above.
Then finally deploy your contract with:
----
> var deploy = {from eth.coinbase, data:byteCode, gas:2000000}
> var fooInstance = foo(bar, baz)
----
=======

[[on_blockchain_testing_sec]]
==== On-Blockchain Testing

Although most testing shouldn't occur on deployed contracts, a contract's behavior can be checked via Ethereum clients.  The following commands can be used to assess a smart contract's state. These commands should be typed at the '+geth+' terminal, although any web3 calls will also support these commands.

....
eth.getTransactionReceipt(txhash);
....
Can be used to get the address of a contract at `+txhash+`.
....
eth.getCode(contractaddress)
....
Gets the code of a contract deployed at `+contractaddress+`. This can be used to verify proper deployment.
....
eth.getPastLogs(options)
....
Gets the full logs of the contract located at address, specified in options. This is helpful for viewing the history of a contract's calls.
....
eth.getStorageAt(address, position)
....
Gets the storage located at `+address+` with an offset of `+position+` shows the data stored in that contract.