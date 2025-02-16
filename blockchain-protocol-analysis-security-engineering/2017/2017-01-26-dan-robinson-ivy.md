---
title: Dan Robinson - Ivy (2017-01-26)
transcript_by: Michael Folkson
categories: ['conference']
tags: ['smart contracts']
---

Name: Dan Robinson

Topic: Ivy: A Declarative Predicate Language for Smart Contracts

Location: BPASE 2017, Stanford University

Date: January 26th 2017

Video: https://www.youtube.com/watch?feature=youtu.be&v=xdxs7VJTqns

# Intro

Hi I’m Dan Robinson, I’m one of the Product Architects at Chain which is an enterprise blockchain infrastructure company for financial institutions. We have a full stack blockchain protocol and what I am going to talk about is part of that that also can be used for other applications including as I am going to demo here, make it easier to write scripts for Bitcoin Script.

# Introduction: Two Blockchain Models

To start out I want to talk about two blockchain models. This isn’t the purpose of why I am here, we are not going to get into the religious debate about which of these is a better fit for certain blockchain applications. Broadly speaking you can divide blockchains into those whose smart contract model is based on mutable state, which is accounts and contracts, this is the model that Ethereum uses. And ones that are based on immutable state, the UTXO model which is the one that Bitcoin uses. Chain’s blockchain is based on this immutable state as is Bitcoin’s and Corda’s. I didn’t realize this before today but Tezos probably fits somewhere in between these and doesn’t fit into this model. Generally you could say it has a UTXO model. That is something interesting that I am definitely going to look into.

# Smart contract development in Ethereum

A lot of people here I’m sure are familiar with how smart contract development in Ethereum works. There is a low level language, EVM Assembly which you would never want to write yourself. Then there is a higher level language which is Solidity. These are the same contracts. This happens to be the withdrawal contract for the DAO that was hard forked. As we are well aware writing code in Solidity does not make it impossible to have catastrophic errors but it would be completely impossible to write anything of any kind of complexity bug free if you were writing it in Assembly so very few people do this. It is part of the luxury of being able to write your programs in Solidity.

# Smart contract development in Bitcoin

Bitcoin has a low level scripting language as well. Bitcoin’s programs, I will talk about how they are different from Ethereum’s, when people write Bitcoin script and write papers demoing how these applications will work. Andrew Miller’s talk earlier about trying to do a lottery in Bitcoin, they generally use this low level stack based language, these opcodes, or they write it in pseudo code. There is no higher level language that I am aware of that people use to write smart contracts in Bitcoin or any other of these UXTO based languages. That is the role that Ivy can play. This is a version of that script that is written in Ivy and specifically a dialect of Ivy that is meant for Bitcoin.

# Ivy

What is Ivy? Ivy is a predicate language (delta_0). Fortunately Russell (O’Connor) was on earlier to explain exactly what that means. I was able to add the point that Ivy is a delta_0 language. Programs can only succeed or fail. You don’t have any loss of generality or power here at least in the version of Ivy that runs on Chain’s blockchain. Programs can only either succeed or fail. What a particular program does is it locks up value in a UTXO on a blockchain. You can consider it like a safe that has to be opened to obtain the value that is stored within. Ivy was designed to write programs for the Chain virtual machine. A little about Chain, it is a private blockchain platform and it is a multi asset blockchain. Its virtual machine is more powerful than Bitcoin’s and it is Turing complete but it is still based on this UTXO model. Ivy was designed to make it easier to write programs for this model but a limited dialect of it. The one I am going to present today compiles to Bitcoin Script. What I am going to talk about is how this makes a little easier to write, read and teach Bitcoin scripting.

# Ivy - Benefits

Some of the benefits of Ivy syntactically. It abstracts away from stack manipulation opcodes so you don’t have any of these OP_DUP, OP_SWAP. Instead it allows you to define named variables. Instead of having this unfamiliar postfix notation that is part of having a low level stack based language this is the code to verify that 1+1=2. It puts it in a more familiar syntax. It is not an especially high level language. It should be fairly obvious once we have got the hang of it to see how any Ivy program will compile to Bitcoin Script. But it allows you to think at a higher level abstraction and it makes it more readable. That is part of the benefit. I have also found that it is a very useful way to teach Bitcoin Script and what Bitcoin Script does, how a single Bitcoin transaction works which is something that a lot of people, even people in the Bitcoin space, don’t really get into, know much about or think about. 

# Ivy Examples

We are going to go through a few examples of Ivy. I am also going to try some live coding. I have never before shown this to anybody outside of the company. This is a prototype compiler from Ivy. This version is from Ivy to Bitcoin Script. We are going to do some live coding and see exactly how Ivy compiles to Bitcoin Script and how it is executed.

# PayToPubKey

This is what people should be familiar with as a pay-to-pub-key transaction. This is the original simplest design for a Bitcoin transaction. It is essentially obsolete because it has been replaced by pay-to-pub-key-hash and pay-to-script-hash and other more sophisticated and secure ways of doing scripting. This would be the very simplest way to have a control program on Bitcoin that checks for a signature. We will go through how that low level Bitcoin Script program works and we will also show how it maps to our Ivy program. In Ivy we define a program and that goes into what in the Bitcoin transaction is called a scriptPubKey. That is part of the output that stores a particular amount of Bitcoin on the blockchain. You could have 5 Bitcoin locked up by one of these programs. At the time you lock that up, at the time you create this output you specify the program argument that you see here. That is the public key that you want to be able to spend that amount of Bitcoin. When you pay into a particular address that address is incorporated in this public key. Down there the scriptPubKey program (<pubkey> OP_CHECKSIG) that’s the address that you send the money to. That is provided at the time that program is created. 

Every program in Ivy has one or more paths. This one only has one path. We will see one later that has multiple. This one takes a single argument (scriptSig). This argument is provided at the time the UTXO is spent. This is a signature on the transaction. You see here that is one of the arguments and that goes in the scriptSig field in a Bitcoin transaction. We are not going to fixate on the stack language here but what the low level language down there does is when you spend a input in a Bitcoin transaction you first put the arguments that are provided from the witness of the transaction, you put those onto the stack first and then you execute the program that was specified in the output. That program, it is run on stack based machine. The first thing you do is push this signature. As I show here the first thing that the program will do is push the pubkey onto the stack. Then it executes your verification logic here. All this does is run a single CHECKSIG instruction which checks the public key against the signature and makes sure it validates. This program returns TRUE if that signature matches. This leaves TRUE on the stack and Bitcoin requires you to leave an extra value on the stack when it is done. If you have provided a valid signature for this transaction matching that pubkey you can spend this UTXO. This is really the “Hello world” program in Ivy. Let’s get slightly more complex.

# PayToPubKeyHash

Let’s add a single additional condition which is the hash. Instead of specifying the public key in your control program that lets anyone see what your public key is ahead of time, it is a little less secure in case there is ever a compromise of the elliptic curve cryptography that these signatures are based on. Instead you provide a hash of the public key in your output. The lower stack program down there, that is the code that is part of the output. You hardcode this value. 

`OP_DUP OP_HASH160 <pkHash> OP_EQUALVERIFY OP_CHECKSIG`

On the previous one you hardcode the pubkey value into the program, now you are hardcoding the hash into the output program. This program, when you generate it, when you instantiate it with this particular program level argument it creates that program down at the bottom and it fills in that value for the public key hash. At the time you spend it you have to provide the public key as well as the signature. The first thing that the program does is it makes sure that the public key that you provided matches the hash. The hash of the public key is what was previously committed to. Then it does the same thing, it checks the signature. Let’s take a look at our compiler.

This is our Ivy [compiler](https://ivylang.org/bitcoin/create). This is your average web based interactive compiler. This here is the same program, the first program that we were looking at. You will see it compiles here. This compiler doesn’t actually optimize it as well as potentially it could. There is a little additional stack manipulation overhead that goes on there. This shows the stack trace of what happened. The first thing is your program specifies this argument, the pubkey. This is part of the initial program. When it is executed you provide the path arguments, that is the lowest thing on the stack. This is what the stack looks like. It puts the signature here, then it puts the signature with the pubkey on top. This is redundant, it could be optimized out. It runs CHECKSIG on the public key and the signature. The virtual machine verifies the top value was TRUE, in other words that the public key matched the signature.

So let’s turn this into our pubkey hash. Instead of taking a public key it is going to take the hash of that public key. Instead of taking one argument, the signature, at the time that it is spent, it is going to take two arguments: the public key and the signature. We are going to verify that the hash of the public key is equal to the pubKeyHash. This is the compiled program.

`<pubKeyHash> OP_OVER OP_HASH160 OP_OVER OP_EQUAL OP_VERIFY 2 OP_PICK 2 OP_PICK OP_CHECKSIG`

This puts the public key hash onto the stack first. That is what is hardcoded into the program. This is what the program actually looks like in the output. You will see here that this first checks the quality of the pubKeyHash and only then does it do the signature checking. Like I said it is a slightly less optimized compiler than it potentially could be but it gets the idea across of the logic that is actually being done.

# Escrow with timeout

Let’s look at a slightly more complex transaction, still on Bitcoin Script. This is an escrow transaction. The way this one works is suppose Alice wants to send money to Bob but wants it to be escrowed for a limited amount of time and can only be approved, the escrow agent will be allowed to cancel that transaction within that amount of time. So possibly she is sending this money pending delivery of some goods and the escrow agent would be able to confirm that those goods were delivered. The logic behind this isn’t particularly important but there is a lot of variance on these kinds of programs that when chained together can give you a lot more complex logic. This simple example will show how it can be expressed with multiple paths. You see here this program unlike the other one has multiple paths. There are two things that can happen. Either the transaction could be approved or the transaction could be cancelled. That requires first that the timeout has happened. This would happen if neither the agent nor the sender is willing to sign the transaction. The agent stays available and the sender is bitter. After a particular timeout the other recipient is able to obtain the money that was meant to be sent. You could imagine a different business logic here where the sender would get the money back. The happy path is approval. This is a 2-of-3 multisig between the sender, the recipient and the agent. If the agent approves the transaction or if the agent wants to cancel the transaction the agent can sign with the recipient or the sender. If we look at how this will look in Ivy and on the stack. 

`<sender> <recipient> <agent> <timeout> 0 5 OP_PICK OP_EQUAL OP_IF  6 OP_PICK 6 OP_PICK 2 5 OP_PICK 5 OP_PICK 5 OP_PICK 3 OP_CHECKMULTISIG OP_VERIFY OP_ELSE 1 5 OP_PICK OP_EQUAL OP_IF OP_PICK OP_CHECKLOCKTIMEVERIFY OP_DROP OP_VERIFY 5 OP_PICK 2 OP_PICK OP_CHECKSIG OP_VERIFY OP_ELSE OP_ENDIF 1`

What this does is if everyone is familiar with how this logic in Bitcoin Script works, is you’d have an IF statement, a conditional that checks the value of some path selector here. This will either be a 0 or a 1, meaning that you should either approve or cancel the transaction. The first thing this does is check whether that path selector is equal to 0. If so it takes this path that expects 3 items on the stack. Otherwise it checks if the path selector is 1 and if so it does the other logic with the other items expected on the stack. This can get you a lot of flexibility and really simplifies all that random bookkeeping conditional logic, the idea of having multiple paths that can be followed.

# Covenants

Now let’s get a little ridiculous. I was very pleased to go [after](https://diyhpl.us/wiki/transcripts/blockchain-protocol-analysis-security-engineering/2017/2017-01-26-russell-oconnor-posts-theorem/) Russell (O’Connor) for a couple of reasons. One is that he explained what a predicate language was but another one is that I use one of the examples from his [blog post](https://blockstream.com/2016/11/02/en-covenants-in-elements-alpha/) in this presentation. If everyone is familiar with covenants, covenants would be a feature that could be added to Bitcoin and is available on other blockchain platforms including Chain’s. This is the ability for an input to be conditional on the presence of a particular output in that transaction. This can be used in Bitcoin to implement a lot of interesting ideas. Andrew mentioned that it saves a lot of trouble and can be used to make a more efficient lottery in Bitcoin and it can be used for some security things like vaults. On multi asset blockchains like Chain’s it allows you to get really interesting in the kind of smart contract applications that you can do. Russell had a post where he realized that the two additional signatures in Blockstream’s Elements Alpha sidechain and those script extensions allow you to implement covenants in a very roundabout way. This is a very simple covenant and this is one that requires that the input can be spent but only to an identical output, an output whose script is the same as the input script. He calls it a recursive covenant. You could also a consider it a quine. It has to get a little clever in order to use these opcodes that don’t give you direct access to the output to be able to construct that transaction. I am not going into the logic of that but it is a really fun blog post if you want to read it. I found this.

```
	1.	<0x0100000001>
	2.	OP_SWAP OP_SIZE 36 OP_NUMEQUALVERIFY OP_CAT
	3.	<0x00> OP_CAT
	4.	OP_SWAP OP_SIZE 32 OP_NUMEQUALVERIFY OP_CAT
	5.	<0x00005f> OP_CAT
	6.	2 OP_PICK OP_SIZE 95 OP_NUMEQUALVERIFY OP_CAT
	7.	<0xffffffff0100> OP_CAT
	8.	OP_SWAP OP_SIZE 32 OP_NUMEQUALVERIFY OP_CAT
	9.	<0x0000> OP_HASH256 OP_CAT
	10.	<0x17a914> OP_CAT
	11.	OP_SWAP OP_HASH160 OP_CAT
	12.	<0x870000000001000000> OP_CAT
	13.	OP_SHA256
	14.	1 OP_DUP OP_CAT OP_DUP OP_CAT OP_DUP OP_CAT OP_DUP OP_CAT OP_DUP OP_CAT OP_DUP OP_CAT
	15.	OP_DUP
	16.	2 OP_ROLL 3 OP_PICK
	17.	OP_CHECKSIGFROMSTACKVERIFY
	18.	1 OP_CAT OP_SWAP
	19.	OP_CHECKSIG
```
It is impossible for a human to evaluate this in their head or understand it. Let’s see if we can do a little better. This is the covenants code in Ivy, that roundabout way to get covenants from CHECKSIGFROMSTACK and CAT. What this essentially does is it builds a new transaction that is confirmed that it has only output and that is an output with the same script as this current one. It makes sure by checking first a signature on the transaction and then checking a signature with the same public key on a hash using the CHECKSIGFROMSTACK opcode, it checks the identity of the transaction. It is a roundabout way of doing it. It is slightly clearer in this but we generally think why not just add covenants as a first class feature. The point I am trying to make here is that Bitcoin script can be a lot more flexible if we added some opcodes. But often it is hard to explain how. It is hard to do these examples where you show these particular applications because you are always writing the examples in low level script or in pseudo code. So having something like this to be able to say “Here is how we would do this if we added this particular opcode” is a way to make the case for a new opcode. I was amazed by the creativity in this post but one nice thing about working at a private blockchain company instead of working on Bitcoin opcodes, we can just add an opcode by fiat with a pull request. There is no political fighting over it at all. 

# Offer

We just did. This is an example. I am not going to show the compiled version of this. This compiles to the Chain virtual machine. This takes advantage of both that opcode that I told you about. You’ll see this uses a `hasOutput` function that we provide that essentially checks for the presence of a particular output in a transaction. But it also uses the fact that Chain is a multi asset blockchain. Multi asset blockchains really allow you to take full advantage of covenants and these other features because it allows you to implement a smart contract where payment of one asset is conditional on receiving another asset. That is what this is, this offer. This program is used to secure a particular amount of value, let’s say 1 share of Microsoft. You are offering a particular price and in some other asset. You could be demanding 50 US dollars for this and you put this on the blockchain. You want them to send it to your public key, you want them to send it to you as a seller. You provide that as part of the program. There are two paths to this offer. One is to lift the offer which in the finance world it means to accept an offer. That allows the person who is lifting the offer to unlock that asset and get the share of Microsoft as long as the same transaction also pays the specified amount to the seller’s account. You’ll see here that account thing, it instantiates a new Account program, a pay-to-pub-key program or pay-to-pub-key-hash program based on the seller’s key that is specified there. This is another feature that we have that allows us to take advantage of these features. The other path, if you have put this up and no one is biting on your offer, the seller can sign the transaction and withdraw their asset from there. This is just one example. If you want more examples of how Ivy can be used on the Chain blockchain or how it can be used generally on a multi asset blockchain you can check out our website. 

# Potential applications and compilation targets

Why did we do all this? The primary reason is to compile to the Chain virtual machine, for applications for multi asset blockchains. But I think these ideas are very applicable to writing programs for Bitcoin script and specifically more complex applications or extensions to Bitcoin script. We went over some very simple toy applications. But when you get into something like lotteries or the Lightning Network if you have to do all your thinking at the low level opcode level it can be very hard to get a grip on the full thing that you are doing. This allows you to work at a higher level of abstraction. A lot of people think that the reason people aren’t writing smart contract applications in Bitcoin is because the language is less powerful but as some people have mentioned you can do a lot with Bitcoin script and particularly with chains of transactions. I think we are only starting to scratch the surface of in applications like the Lightning Network. The question is why haven’t people started to take advantage of the power that we have? I think the answer is a lack of ability to write these programs in a way that is intuitive to an ordinary developer. The other effect is the lack of power of Bitcoin script. There are some opcodes as I mentioned that would make it a lot easier to write these applications. The lack of power is partly due to it being hard to explain what these additional opcodes would do. And nobody could really do much with them even if we had them because it is hard to write these programs. I think this difficulty might be endogenous to this problem and so having an easier higher level language to make that case for additional opcodes could be a good way to get there. Other possible uses of Ivy, this is not just uses for Ivy but also ideas that Ivy could learn from and take some ideas from, would be compiling to Crypto-Conditions. Crypto-Conditions is used in the Interledger protocol. It is a predicate language. Programs can only return TRUE or FALSE given certain inputs. Any of these delta_0 languages, Ivy could potentially compile to or compile from. The other application that we think about is making it easier to write zk-SNARKS. Right now it is very hard for an ordinary programmer to write a given zk-SNARK. A zk-SNARK allows you to do a confidential proof that you have an input such that a given program returns TRUE. It is a delta_0 language. Right now it is hard for me to write that kind of program. Prove that you have this number that when multiplied by another number will reach some other number. Or prove that you have a preimage to this hash in zero knowledge. It is hard to write those. If we had a simple scripting language for writing these predicate languages it could be useful. If you want to find out more about Ivy:

https://github.com/ivy-lang/ivy-bitcoin

https://docs.ivylang.org/bitcoin/

https://ivylang.org/bitcoin

