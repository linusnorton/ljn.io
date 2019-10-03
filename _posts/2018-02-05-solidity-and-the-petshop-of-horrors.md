---
title: Solidity and the Pet Shop of Horrors
date: 2018-02-05 07:00
lastmod: 2018-02-05 07:00
description: Solidity is a language designed for programming on the Ethereum blockchain. Unfortunately it contains a number of rough edges, let me introduce you to the pet shop of horrors...
layout: post
---

Starting out with a new programming language is always a huge undertaking. I've often thought that it's a valuable learning experience for both the the person learning and those maintaining the language as it's easy to forget what misconceptions a beginner might have. I'm going to go over some of the surprises I found whilst learning Solidity. 

## Solidity

Solidity is a language designed specifically for the Ethereum Virtual Machine. It's relatively new (2014/2015) and described as similar to JavaScript but with static types. I spend most of my time writing TypeScript so the syntax is fairly familiar but appearances can be deceiving, under the hood the language is a very different beast. 

## The Pet Shop

<div class="pull-right" markdown="1" style="width: 400px; margin: 10px 15px 0 30px">

![puppies](/asset/img/solidity/puppies-puppies.jpg)

</div>

My preferred method of learning is called "converging" (see [Kolb's learning styles](https://www.businessballs.com/self-awareness/kolbs-learning-styles-64/)), which essentially means I like to learn by doing. Luckily there are some really good resources geared towards people like myself. 

I found [Truffle's pet shop tutorial](http://truffleframework.com/tutorials/pet-shop) particularly useful. In the tutorial you create an example contract that allows people to adopt pets from a pet store. You also set up a minimal website to execute the contract using web3.

I ran through the tutorial and managed to get everything working without any major hiccups. Feeling happy that I had a project with a test harness to play with I began to experiment, and that's when I ran into the Pet Shop of Horrors. 


## The Horrors

The pet shop tutorial sets up a registry of pets that can be adopted. Initially Pets are just represented by IDs (unsigned integers) so my first modification was to set them up as struct:

```
struct Pet {
  string name;
  uint8 age;
}
```

Now instead of adopting an ID I can adopt a Pet. Wrong! 

**Passing structs as arguments is allowed, but only if everything in the struct is fixed length, and is not a mapping** [[1](https://ethereum.stackexchange.com/questions/3681/how-to-interact-with-the-contract-with-more-complex-argumentssort-of-a-string-o)][[2](https://forum.ethereum.org/discussion/2417/structs-as-function-parameters)].

Our Pet struct has a name of string, which is a variable length array of byte32 so no can do.

No problem, we'll set up a registry of pets that are added to the contract. Then we can just pull back the pet by it's ID. Wrong again! 

**Public methods cannot return structs (but private methods can)** [[3](https://ethereum.stackexchange.com/questions/7317/how-can-i-return-struct-when-function-is-called)].

Okay, I will just add a mapping (HashMap) of pet name to ID so I can look up each pets ID. Nope!

**Mappings can only have fixed length keys, strings are variable length** [[4](https://ethereum.stackexchange.com/questions/15828/using-strings-as-indices-in-solidity)][[5](https://github.com/ethereum/solidity/issues/1550)].

Okay, let's reverse that. I'll keep a mapping of ID to pet name and add a method to look up the pet name... great!

**You CAN return a string from a method**.

Now all I need is to be able to return a list of an owners pets. Every time a pet gets adopted I'll maintain an index that tracks each owners pets, then I'll add a method to return the array.

Compiler says no:

```
UnimplementedFeatureError: Nested dynamic arrays not implemented here.
Compilation failed. See above.
```

**Methods cannot return variable length arrays** [[6](https://ethereum.stackexchange.com/questions/17312/solidity-can-you-return-dynamic-arrays-in-a-function)].

Okay, Solidity really doesn't want me to store the details of the pets on-chain (in the contract) so I won't. 

I'll just store an array of strings that represent pets up for adoption. Now my UI needs to display that list. Oh wait.

**You cannot read an entire array from the contract** [[7](https://ethereum.stackexchange.com/questions/3114/calling-public-array-of-structs-using-web3)].

Instead you have to iterate through each item:

```
const count = await contract.getNumberOfPets.call();
const adoptions = [];
    
for (let i = 0; i < count; i++) {
  const petName = await contract.pets.call(i);
  adoptions.push(petName);
}
```

Jeez, all those asynchronous calls are going to scale well. Nevermind. 

Now let's add a method to put a pet up for adoption. Fairly simple:

```
function addPet(string memory petName) public returns (uint) {
  pets.push(petName);
 
  return pets.length - 1;
}
```

Even though the method returns the new length, I need to call the `getNumberOfPets` method again because:

**Functions that mutate the contract cannot return values** [[8](https://ethereum.stackexchange.com/questions/3285/how-to-get-return-values-when-function-with-argument-is-called)].

On top of these issues I also stumbled across this one:

**Modifiers cannot exist in a library**. [[9](https://github.com/ethereum/solidity/issues/2467)].

## Enough

As you can see, I hit my fair share of issues with something as trivial as setting up an example pet shop. 

There are a surprising number of constraints in Solidity, but don't take this as criticism . Instead accept these "horrors" for what they are - a beginners notes on using a new language. Some of them are just inherent restrictions of building something on a blockchain. To be fair some of them are non-obvious choices made by the language designers. 

I'd also note that my style of learning ("just do it!") exacerbates these issues. Some of these issues are documented, unfortunately many are not. 

After grappling with these limitations I did finally reach a sort of zen, or at least state of acceptance with the language. I think most of the initial shock comes from the perception that Solidity is similar to JavaScript. When I re-calibrated my expectations to be closer to C (which I have not used for a long, long time) everything became a bit clearer. 

You have to think very hard about storage and memory constraints. You also have to build a lot of the scaffolding yourself, but hey, C was fun wasn't it?

![flying-pig](/asset/img/solidity/flying-pig.jpg){: .img-responsive }
