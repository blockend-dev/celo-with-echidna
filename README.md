# Smart Contract Security Testing on Celo Blockchain with Echidna: A Step-by-Step Guide

## Table of Contents
- [Smart Contract Security Testing on Celo Blockchain with Echidna: A Step-by-Step Guide](#smart-contract-security-testing-on-celo-blockchain-with-echidna-a-step-by-step-guide)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Setting up the Development Environment](#setting-up-the-development-environment)
  - [Create an ERC-20 Token Contract](#create-an-erc-20-token-contract)
  - [Testing the Contract using Echidna](#testing-the-contract-using-echidna)
  - [Deploy the Contract](#deploy-the-contract)
  - [Conclusion](#conclusion)

## Introduction

Smart contracts are self-executing programs that run on blockchain platforms. They can be used to automate various processes, including financial transactions, voting systems, and supply chain management. Smart contracts are immutable and tamper-proof, making them a secure way to automate business processes. However, despite their security features, smart contracts are still vulnerable to bugs and vulnerabilities that can be exploited by attackers.

To ensure the security of smart contracts, developers need to test them thoroughly. Testing can help identify vulnerabilities and bugs that could lead to a loss of funds or other damages. And this is where Echidna comes in!

**[Echidna](https://github.com/crytic/echidna)** is a powerful tool for testing smart contracts, and it can be used to identify potential security vulnerabilities in a smart contract.

This tutorial will walk you through the process of using Echidna to test the security of a smart contract deployed on the Celo network. We will start by setting up the development environment and creating an ERC-20 token contract. We will then use Echidna to generate and run test cases on the contract to identify any potential security vulnerabilities.

## Prerequisites
Before we get started, there are a few prerequisites that you need to have in place:
- Basic understanding of JavaScript programming language.
- Basic understanding of Solidity programming language.
- Basic understanding of how the Celo network and blockchain works.
- Node.js and npm installed on your machine.

## Setting up the Development Environment
To get started with developing smart contracts on Celo, you need to set up your development environment. 

Install Node.js and npm: Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. npm is a package manager for Node.js. You can download Node.js and npm from the official website: https://nodejs.org/en/download/.


## Create an ERC-20 Token Contract

Now that you have set up your development environment, you can create an ERC-20 token contract. ERC-20 is a standard interface for tokens on the Ethereum network, and it is also supported on Celo.

For the sake of this tutorial, here is the code for an ERC-20 token contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.8;

contract OxToken {
    string public name = "Ox Token";
    string public symbol = "OXT";
    uint256 public totalSupply;

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    constructor(uint256 _totalSupply) {
        balanceOf[msg.sender] = _totalSupply;
        totalSupply = _totalSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value, "Insufficient balance");
        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value, "Insufficient balance");
        require(allowance[_from][msg.sender] >= _value, "Insufficient allowance");
        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;
        emit Transfer(_from, _to, _value);
        return true;
    }
}
```

The smart contract above implements the ERC-20 token standard, which is a popular standard used for creating fungible tokens on the Ethereum blockchain. The ERC-20 standard defines a set of functions and events that a smart contract must implement in order to be considered an ERC-20 compliant token.

The contract defines a public string variable `name` and `symbol` that represents the name and symbol of the token respectively. It also defines a public uint256 variable `totalSupply` that represents the total supply of the token.

Also, there are two mapping data structures: `balanceOf` and `allowance`. The `balanceOf` mapping maps an address to a uint256 value that represents the balance of that address. The `allowance` mapping maps an address to another mapping, which maps an address to a uint256 value that represents the amount of tokens that the owner of the first address has allowed the second address to spend.

Again, there are two events: `Transfer` and `Approval`. The `Transfer` event is emitted whenever tokens are transferred from one address to another. The `Approval` event is emitted whenever an address approves another address to spend a certain amount of tokens.

The constructor takes a uint256 argument `_totalSupply` and sets the balance of the contract creator to `_totalSupply` and the totalSupply to `_totalSupply`.

The are three functions: `transfer`, `approve`, and `transferFrom.` The `transfer` function takes two arguments: an address `_to` and a uint256 `_value`. It transfers `_value` number of tokens from the sender's address to `_to`. The function checks if the sender has sufficient balance before executing the transfer and emits the `Transfer` event.

The `approve` function takes two arguments: an address `_spender` and a uint256 `_value`. It approves `_spender` to spend `_value ` which is the number of tokens on behalf of the sender. The function emits the `Approval` event.

The `transferFrom` function takes three arguments: an address `_from`, an address `_to`, and a uint256 `_value`. It transfers `_value`  - number of tokens from `_from `to `_to` on behalf of the sender. The function checks if `_from` has sufficient balance and if the sender has sufficient allowance before executing the transfer and emits the `Transfer` event.

In simple terms, this contract implements the ERC-20 interface and allows users to transfer tokens, approve other addresses to spend tokens, and transfer tokens from one address to another.

Next, we will write a test script to test the contract using Echidna.

## Testing the Contract using Echidna

Now that we have created our ERC-20 token contract, we can test it using Echidna. Echidna is a property-based testing tool for Ethereum smart contracts that allows you to generate and run test cases automatically.

Below are the steps to test the ERC-20 token contract using Echidna:

1. Install Echidna: You can install Echidna using npm with the following command:

```bash
npm install -g echidna-test-runner
```

2. Create a new file named test.js in the root directory of your project where you will write the code for testing the smart contract.

```javascript
const { assert } = require('chai');
const ethers = require('ethers');
const { echidna } = require('@trailofbits/echidna');

describe("OxToken Test Suit" () => {
  let OxToken, oxToken;

  beforeEach(async () => {
    OxToken = await ethers.getContractFactory('OxToken');
    oxToken = await OxToken.deploy('Ox Token', 'OTK', 10000);
    await oxToken.deployed();
  });

  it("should set the correct name, symbol and total supply", async () => {
    const name = await oxToken.name();
    const symbol = await oxToken.symbol();
    const totalSupply = await oxToken.totalSupply();

    assert.equal(name, 'Ox Token', 'Incorrect name');
    assert.equal(symbol, 'OTK', 'Incorrect symbol');
    assert.equal(totalSupply, 10000, 'Incorrect total supply');
  });

  it("should transfer tokens between accounts", async () => {
    const [owner, recipient] = await ethers.getSigners();

    await oxToken.transfer(recipient.address, 100);
    const balanceRecipient = await oxToken.balanceOf(recipient.address);

    assert.equal(balanceRecipient, 100, 'Incorrect balance for recipient');
    
    const balanceOwner = await oxToken.balanceOf(owner.address);

    assert.equal(balanceOwner, 9900, 'Incorrect balance for owner');
  });

  it("should fail if trying to transfer more tokens than the balance", async () => {
    const [owner, recipient] = await ethers.getSigners();

    await assert.revert(oxToken.transfer(recipient.address, 101), 'Transfer amount exceeds balance');
  });

  it("should approve and transfer tokens from an approved spender", async () => {
    const owner = await oxToken.owner();
    const spender = ethers.Wallet.createRandom().address;
    const recipient = ethers.Wallet.createRandom().address;

    await oxToken.approve(spender, 100);
    await oxToken.connect(ethers.provider.getSigner(spender)).transferFrom(owner, recipient, 100);
    const balance = await oxToken.balanceOf(recipient);

    assert.equal(balance, 100, 'Incorrect balance');
  });
});

describe("OxToken - Echidna", function () {
  let contract;

  beforeEach(async () => {
    const OxToken = await ethers.getContractFactory('OxToken');
    contract = await echidna.init(OxToken, 10000);
  });

  it("should maintain the total supply", async () => {
    await contract.methods.transfer(ethers.constants.AddressZero, 1);
    const totalSupply = await contract.methods.totalSupply().call();

    assert.equal(totalSupply, 10000, 'Incorrect total supply');
  });
});
```

Let's go through the code together.
We defined a describe block for testing the `OxToken `contract and created a `beforeEach` hook to deploy a new instance of the contract before each test.

In the first test, you will test the constructor by verifying that the name, symbol and total supply are set correctly.

In the second test, you will test the transfer function by transferring tokens from the owner account to a new recipient account and verifying that the balance of the recipient account is updated correctly.

In the third test, you are testing that the `transfer` function should fail if trying to transfer more tokens than the balance.

In the fourth test, test the `approve` and `transferFrom` functions by approving a new spender account to transfer tokens and verifying that the balance of the recipient account is updated correctly.

In the second describe block, we use Echidna to test the `Oxoken` contract by ensuring that the total supply of tokens remains constant after a transfer to the null address (0x0) is attempted. This test is to ensure that the total supply cannot be increased or decreased by an invalid transfer operation.

Next, use the `echidna.init()` function to initialize Echidna with the `OxToken` contract and set the maximum supply of tokens to 10,000. Then define a test that transfers 1 token to the null address and verifies that the total supply remains at 10,000.

These tests cover the basic functionality of the ERC-20 token and ensure that the contract is working correctly and securely.

## Deploy the Contract

To deploy the OxToken contract on the Celo network, you can use the following steps:

1. Install the Celo CLI by following the instructions in the official documentation: https://docs.celo.org/getting-started/basics/install-the-celo-sdk.

2. Connect to a Celo network. You can either connect to the Alfajores testnet or the Celo mainnet. For example, to connect to the Alfajores testnet, you can run the following command:

  ```bash
    celocli network:switch --network alfajores
  ```

3. Create a new account and fund it with some testnet or mainnet CELO using the Celo CLI. For example, to create a new account and fund it with 1 CELO on the Alfajores testnet, you can run the following commands:

```bash
celocli account:create
celocli transfer:celo --from <YOUR_ACCOUNT_ADDRESS> --to <NEW_ACCOUNT_ADDRESS> --value 1
```

Replace <YOUR_ACCOUNT_ADDRESS> with your own account address and <NEW_ACCOUNT_ADDRESS> with the address of the new account that you just created.

4. Compile the `OxToken` contract using the Solidity compiler. You can use the `solc` command-line tool or any other Solidity compiler that you prefer. For example, to compile the contract using `solc`, you can run the following command:

```bash
solc OxToken.sol --bin --abi --optimize -o build/
```
5. Deploy the OxToken contract using the Celo CLI. You can use the `contract:deploy` command to deploy the contract. For example, to deploy the contract on the Alfajores testnet, you can run the following command:

```bash
celocli contract:deploy --from <YOUR_ACCOUNT_ADDRESS> --path build/OxToken.bin --constructor-args 1000000
```

Replace <YOUR_ACCOUNT_ADDRESS> with your own account address.

This will deploy the contract with an initial supply of 1,000,000 tokens.

6. Verify the contract source code on the Celo blockchain explorer. To do this, you can use the `contract:verify` command of the Celo CLI. For example, to verify the contract on the Alfajores testnet, you can run the following command:

```bash
celocli contract:verify --address <CONTRACT_ADDRESS> --path OxToken.sol
```
Replace <CONTRACT_ADDRESS> with the address of the deployed contract.

Congratulations, you have now deployed and verified the OxToken contract on the Celo network


Finally, let's cover some important security considerations for writing smart contracts on the Celo blockchain.

It's crucial to take security seriously while creating smart contracts on the Celo blockchain or any other blockchain. Smart contracts are executed on the blockchain and can have serious consequences if they contain bugs or vulnerabilities

So, here are some important security considerations to keep in mind when writing smart contracts on the Celo blockchain.

1. Avoid using the `transfer` function
The `transfer` function is commonly used to transfer tokens from one account to another in an ERC-20 contract. However, it has some security issues that can lead to the loss of funds.

The `transfer` function is susceptible to the reentrancy attack. This attack occurs when a malicious contract calls the `transfer` function of another contract before the first call has finished executing. This can result in the attacker being able to withdraw funds repeatedly until the contract's balance is depleted.

To avoid this issue, it is recommended to use the `transferFrom` function instead of the `transfer` function. The `transferFrom` function requires approval from the token owner before the transfer can take place, which prevents the reentrancy attack.

2. Use safe math libraries
Arithmetic operations in Solidity can cause integer overflow or underflow, which can result in unexpected behavior or loss of funds. To avoid these issues, it is recommended to use safe math libraries, such as OpenZeppelin's SafeMath, when performing arithmetic operations.

SafeMath provides functions for safe addition, subtraction, multiplication, and division, which ensure that the result of the operation is within the expected range and prevents integer overflow and underflow.

3. Keep the contract simple
Complex contracts are more difficult to understand and can have more vulnerabilities. It is recommended to keep the contract simple and modular, with each function performing a specific task.

4. Avoid using external calls
External calls can be used to interact with other contracts on the blockchain. However, external calls can also introduce security issues, such as reentrancy attacks or unexpected behavior.

It is recommended to avoid using external calls whenever possible. If an external call is necessary, it is important to verify the integrity of the external contract and ensure that it cannot be manipulated to exploit vulnerabilities in the calling contract.

5. Test the contract thoroughly
Thorough testing is essential for ensuring the security and correctness of a smart contract. Automated testing tools, such as Echidna, can help identify vulnerabilities and ensure that the contract functions as expected.

In addition to automated testing, manual testing and code reviews by multiple developers can help identify issues and ensure that the contract is secure and correct.

And that is it! Congratulations on coming this far, and learning one of the important concepts in blockchain-smart contract security.

## Conclusion

Writing secure and robust smart contracts is essential for building decentralized applications on the Celo blockchain. In this tutorial, we have covered the basic functionality of the ERC-20 token and demonstrated how to use Echidna to test a smart contract on the Celo blockchain.

By following these best practices and taking security seriously, you can build secure and reliable smart contracts on the Celo blockchain and contribute to the growth of the decentralized finance ecosystem.

