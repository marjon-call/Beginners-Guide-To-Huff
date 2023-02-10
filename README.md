# Beginners-Guide-To-Huff

Huff is a powerful smart contract programming language that gives developers the most control of how their programs are written. If you are familiar with Solidity, you understand the high level of how to write a smart contract. Then, as you progress as a developer, you may start to learn Yul. Yul allows you to write in pseudo assembly language. This gives you more control over how you write your smart contracts, by allowing you to specify certain EVM opcodes. Huff, on the other hand, solely focuses on letting users write their smart contracts in opcodes. The EVM is a stack machine, and Huff allows you to write smart contracts as they look on the stack. This is an advanced article, and I recommend you have a good understanding of Solidity before learning Huff. Additionally, it may help if you have a basic understanding of how Yul works, but it is not necessary to understand Huff.

If you are unfamiliar with the EVM, the EVM stands for Ethereum Virtual Machine. The EVM is responsible for interpreting your code. It provides a standardized way of computing bytecode. If you look on Etherscan, you will see all calldata and code is compiled into bytecode (hexadecimal). This is why you can write a smart contract in various programming languages, such as Solidity, Vyper, and Huff. By the end of this article, you will have a better understanding of how that bytecode is generated. 


Before we get started, here is how you can set up your project to follow along: <br>
- To install Huff, check out their website's installation process: https://docs.huff.sh/get-started/installing/
- I will be using foundry to write and test my contracts throughout this article. To use foundry check out this link: https://docs.huff.sh/get-started/project-quickstart/#using-the-template


### How The Stack Works
Let’s start by getting a better understanding of how a stack machine works. I’m sure somewhere along your programming journey you’ve heard the metaphor that a stack machine is like a stack of plates. You can keep appending plates to the top, and can only remove plates from the top of your stack. That’s exactly how a stack machine works, but replace plates with instructructions (or opcodes as they are referred to when referencing the EVM). One of the most common opcodes is ```PUSH```. ```PUSH``` is typically suffixed with an unsigned integer between one and thirty-two, referencing the amount of bytes you are going to push. For example, ```PUSH1``` is telling the EVM to push one byte onto the stack. Let’s look at a simple example of how this works.
```
PUSH1 0x01 //  Pushes 1 byte to the stack               | STACK: [0x01]
PUSH2 0x1000 // Pushes 2 bytes to the stack             | STACK: [0x1000, 0x01]
ADD // Performs addition with the two preceding bytes   | STACK: [0x1001]
```
As you can see we are starting from the top of the stack, and we work our way down. Opcodes like ```ADD```, ```SUB```, ```MUL```, and ```DIV``` take the two preceding stack variables and perform an operation on them. 

### How Storage Works
Storage is how we save state variables in the blockchain. Both stack (sometimes referred to as local) variables and memory variables are not persistent. After the transaction finishes executing the variables disappear. Storage, on the other hand, updates the state of the blockchain. For example, in a staking contract you will need to store how many tokens the staker receives on the blockchain, but you do not need to store the variables used to calculate how many tokens the staker receives. 

Storage is organized into a series of slots. There are 2^256 slots in total, and each slot can store 256 bits (32 bytes) of data. That's where ```uint256``` and ```bytes32``` get their names from in Solidity. When working with storage we start with slot 0 and increment from there. If you have a variable that does not take up an entire storage slot, such as a ```uint128```, you can pack that storage slot with another variable as long as it can fit in the remaining 128 bits. This is gas efficient for your smart contract. Storage takes up space on the blockchain, which makes it more expensive to use. Accessing a storage slot for the first time in your smart contract is referred to as touching cold storage. This will cost you 2,100 gas. After you access that storage slot it is then referred to as warm storage, which only costs 100 gas to touch. 100 gas is still pretty expensive so it is best practice to store your storage variables as stack variables or memory variables if you plan on using them frequently in a function.

It is important to note that if your variable (or variables) do not take up the entirety of a storage slot, they will be padded on the left with 0’s. Unless, you are working with a string, which is padded on the right side. If you store a ```bytes1``` variable equal to ```0x01``` to slot 0, storage will look like this: <br>
Slot 0: ```0x0000000000000000000000000000000000000000000000000000000000000001``` <br>
Now if you pack another ```bytes1``` variable equal to ```0x02``` it will look like this: <br>
Slot 0: ```0x0000000000000000000000000000000000000000000000000000000000000201``` <br>
Take note that we are reading and writing from right to left.

Now that you have a basic understanding of how storage works, let’s look at memory!

### How Memory Works
Memory, as previously mentioned, is not persistent. This makes memory a lot cheaper to work with, but still more expensive than the stack. Memory works in a series of 32 bytes. They are sometimes referred to as words, but I will mostly refer to them as memory locations. The first 22 words you access cost a linear amount of gas to use. After that, using memory increases quadratically per word. 

In storage we refer to slot 0, slot 1…, but in memory we use hexadecimal to refer to memory locations. Slot 0 would be memory location ```0x00``` - ```0x20```. ```0x20``` is hexadecimal for 32, which should make sense because we are working in serieses of 32 bytes. Unlike storage, memory will not pack your variables. With Huff, we do not need to worry about the ```scratch space``` or ```free memory pointer```, because, as I mentioned, we are in complete control of allocating these memory locations.

Here are some important operations that require us to use memory:
Return values for external calls
Set function values for external calls
Get values from external calls
Revert with an error string
Log messages
Hash with keccak256()
Create other smart contracts
Now that we understand the basics of memory, let’s look at calldata!

### How Calldata Works
Calldata is the hexadecimal that is generated when calling a function in a smart contract. The first 4 bytes of that calldata is what is referred to as a function selector. The Function selector is generated by taking the ```keccak256()``` of the function’s string (i.e. ```addTwo(uint256,uint256)```) and taking the first 4 bytes. This is how the EVM knows what function you are calling. From there, the rest of the calldata is the parameters for the function call. Calldata costs sixty-eight gas for every nonzero byte and four gas for every empty byte.

Technically, with languages like Huff and Yul, you can format your calldata however you prefer, but it is still best practice to follow the above format. 

Now that we know the basics of calldata, let’s look at how variables are stored!
