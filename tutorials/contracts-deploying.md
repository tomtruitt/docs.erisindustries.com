---

layout: docs
title: "Tutorials | Deploying your Smart Contracts to a Chain - Eris v0.11"

---

# Introduction

For this tutorial, we are going to use a very simple `get` and `set` contract. All this contract does is get and sets a value.

**Note** -- This tutorial is built for Eris versions `>= 0.11.1`. For other versions of this tutorial please see below:

* [v0.11.0](/tutorials/deprecated/contractsdeploying-v0.11.0/)

## Dependencies

Welcome! This tutorial assumes that you have a chain ready to deploy smart contracts to. Should you not have any idea what smart contracts are, please see our [explainer here](/explainers/smart_contracts). If you do not have a chain ready to work on, please see our [chain making tutorial](/tutorials/chain-making).

## Contracts Strategy

We are going to use a very simple `get` / `set` contract which sets a variable and gets that same variable. It is about the easiest interactive contract one can imagine and as such we will use that for showing how to work with the eris platform.

## Overview of Tutorial

In general we are going to take two steps in order to get our contracts deployed to the blockchain:

1. Make sure your application package has the proper information
2. Deploy the contracts

# Make A Contract for Idi

The first thing we're going to do is to add a very simple contract.

```bash
cd ~/.eris/apps
mkdir idi
cd idi
```

Now you'll make a file in this directory. Let's assume that is called idi.sol and has the following contents

```javascript
contract IdisContractsFTW {
  uint storedData;

  function set(uint x) {
    storedData = x;
  }

  function get() constant returns (uint retVal) {
    return storedData;
  }
}
```

What does this contract do? Well, it isn't very interesting we know. It merely `gets` and `sets` a value which is an unsigned integer type.

# Fixup your epm.yaml

Next we need to make an epm.yaml and make it look something like this:

```yaml
jobs:

- name: setStorageBase
  job:
    set:
      val: 5

- name: deployStorageK
  job:
    deploy:
      contract: idi.sol
      wait: true

- name: setStorage
  job:
    call:
      destination: $deployStorageK
      data: set $setStorageBase
      wait: true

- name: queryStorage
  job:
    query-contract:
      destination: $deployStorageK
      data: get

- name: assertStorage
  job:
    assert:
      key: $queryStorage
      relation: eq
      val: $setStorageBase
```

Now. What does this file mean? Well, this file is the manager file for how to deploy and test your smart contracts. eris:package_manager will read this file and perform a sequence of `jobs` with the various parameters supplied for the job type. It will perform these in the order they are built into the yaml file. So let's go through them one by one and explain what each of these jobs are doing. For more on using various jobs [please see the jobs specification](/documentation/eris-pm/latest/jobs_specification/).

### Job 1: Set Job

The set job simply sets a variable. eris:package_manager includes a very naive key value store which can be used for anything.

### Job 2: Deploy Job

This job will compile and deploy the `idi.sol` contract using Eris' compiler service (or run your own locally). The job will wait for the deploy transaction to confirm before it will procede.

### Job 3: Call Job

This job will send a call to the contract. eris:package_manager will automagically utilize the abi's produced during the compilation process and allow users to formulate contracts calls using the very simple notation of `functionName` `params`. eris:package_manager also allows for variable expansion.

So what this job is doing is this. The job is pulling the value of the `$setStorage` job (eris:package_manager knows this because it resolved `$` + `jobName` to the result of the `setStorage` job) and replacing that with the value, which is `5`. Then it will send that `5` value to the `set` function of the contract which is at the `destination` that is the result of the `deployStorageK` job; in other words the result of Job 3. For more on variables in eris:package_manager, please see the [variables specification](/documentation/eris-pm/latest/variable_specification/)

Finally, it is waiting on the call to be sunk into a block before it will proceed.

### Job 4: Query Contract Job

This job is going to send what are alternatively called `simulated calls` or just `queries` to an accessor function of a contract. In other words, these are `read` transactions. Generally the `query-contract` is married to an accessor function (such as `get` in the `idi.sol` contract). Usually accessor, or read only functions, in a solidity contracts are denoted as a `constant` function which means that any call sent to the contract will not update the state of the contract.

The value returned from a query-contract job then is usually paired with an assert.

### Job 5: Assert Job

In order to know that things have deployed or gone through correctly, you need to be able to assert relations. eris:package_manager provides you with:

* equality
* non-equality
* greater than or equals (for integers & unsigned integers values only)
* greater than (for integers & unsigned integers values only)
* less than or equals (for integers & unsigned integers values only)
* less than (for integers & unsigned integers values only)

Relations can use either `eq` `ne` `ge` `gt` `le` `lt` syntax, or, in the alternative they can use `==` `!=` `>=` `>` `<=` `<` syntax in the relation field. This is similar to bash. To make this more explicit we have chosen in the above epm.yaml to use the `eq` syntax, but feel free to replace with `==` syntax if you want.

Both the `key` and the `val` (which in other testing frameworks are the `given` and `expect`ed in an assert function) use variable expansion to compare the result of what was supposed to be sent to the `setStorageBase` job (which should have been sent to and stored in the contracts' storage) with what was received from the `queryStorage` job (which in turn called the `get` function of the contract).

# Deploy (and Test) The Contract

First, let's get our chain turned back on.

```bash
eris chains ls
```

If simplechain is not running, then turn it on with:

```bash
eris chains start simplechain
```

If, when you ran `chains ls`, simplechain is not among the Active Containers then restart it with:

```bash
eris chains new simplechain --dir $chain_dir_this
```

Now, we are ready to deploy this world changing contract. Make sure you are in the `~/.eris/apps/idi` folder, or wherever you saved your epm.yaml. Note that this is a very common pattern in simple contract testing and development; namely to (1) deploy a contract; (2) send it some transactions (or `call`s); (3) query some results from the contract (or `query-contract`s); and (4) assert a result. As you get moving with contract development you will likely find yourself doing this a lot.

```bash
addr=$(cat $chain_dir/addresses.csv | grep simplechain_full_000 | cut -d ',' -f 1)
echo $addr
```

That will make sure we have available the address we would like to use to deploy the contracts. Now we're ready. If the above does not output an address then check your $chain_dir variable and also check that the `simplechain_full_000` variable exists in the addresses.csv.

```bash
eris pkgs do --chain simplechain --address $addr
```

You *should* be able to use any of the addresses you generated during the chainmaking tutorial since they all have the same permission levels on the chain (which, if you followed the simple tutorial are basically all public). If you are using this tutorial outside of the tutorial sequence then you can just give it the address that you'd like to use to sign transactions instead of the `grep simplechain_full_000` bash expansion.

(For those that do not know what is happening in that bash line: `cat` is used to "print the file" and "pipe" it into the second command; `grep` is a finder tool which will find the line which has the right name we want to use; the `cut` says split the line at the `,` and give me the first field).

Note that eris:package_manager can override the account which is used in any single job and/or, eris:package_manager can set a default `account` job which will establish a default account within the yaml. We find setting the default account within the yaml to usually be counter-productive because others will not be able to easily use your yaml unless they have the same keys in their `eris-keys` (which we **never** recommend). For more on using accounts [please see the jobs specification](/documentation/eris-pm/latest/jobs_specification/).

That's it! Your contract is all ready to go. You should see the output in `epm.json` which will have the transaction hash of the transactions as well as the address if the deployed `idi.sol` contract.

**Troubleshooting**

If you get an error which looks something like this:

```irc
Performing action. This can sometimes take a wee while
Post http://chain:46657/status: dial tcp 172.17.0.3:46657: getsockopt: connection refused
Container eris_interactive_eris_service_idi_tmp_deploy_1 exited with status 1
```

That means that your chain is not started. Please start the chain and give the chain a second to reboot before rerunning the deploy command again.

<hr />

If you get an error which looks something like this:

```irc
open /home/eris/.eris/keys/data/1040E6521541DAB4E7EE57F21226DD17CE9F0FB7/1040E6521541DAB4E7EE57F21226DD17CE9F0FB7: no such file or directory
Container 74a9dbf3d72a2f67e2280bc792e30c7b37fa57e3d04aeb348222f72448bdc84a exited with status 1
```

What is this telling us? Well, it is telling us that it doesn't have the key in the `keys` container. So what you'll want to do is to update with one of the keys you have generated during the prior tutorials.

To see what keys are currently on your key signing daemon do this:

```
eris keys ls
```

If you do not have any keys then please take the time to [make some keys](/tutorials/tool-specific/keyexporting). After you find a key which you currently have, then add that as the `address` flag to the `eris pkgs do` command.

<hr />

If you choose the wrong key then you'll get an error which will probably look something like this:

```irc
Error deploying contract idi.sol: unknown account 03E3FAC131CC111D78B569CEC45FA42CE5DA8AD8
Container edbae127e1a31f1f85fbe14359362f7943028e57dc5eec4d91a71df706f5240f exited with status 1
```

This means that the account `03E3FAC131CC111D78B569CEC45FA42CE5DA8AD8` has not been registered in the genesis.json. The account which is not registered will be the same account you told epm to use via the signing server (`eris-keys`).

To "see" your genesis.json then do this:

```
eris chains cat simplechain genesis
```

<hr />

If the account you are trying to use has not been registered in the genesis.json (or, latterly, has not been given the appropriate [permissions](/documentation/eris-db-permissions/) via permission transactions) and been given the appropriate permissions, then it will not be able to perform the actions it needs to in order to deploy and test the contract. The easiest thing to do at this point is to [update your genesis.json](/tutorials/tool-specific/genesisupdating/).

Once you have the following sorted:

1. The account flag used matches a key which is known to the signing daemon; and
2. The account flag used matches an account in the genesis.json of a chain;

Then you'll be ready to:

```bash
eris pkgs do --chain simplechain --address ADDR
```

Where `ADDR` in the above command is the address you want to use.

<hr />

If you get an error which looks like this:

```irc
ERROR =>
Saving ABI                                    /home/eris/.eris/apps/idi/abi
open /home/eris/.eris/apps/idi/abi: is a directory
Container eris_interactive_eris_service_idi_tmp_deploy_1 exited with status 1
```

That is not a very helpful error, we realize. Usually, but not always means that there is a problem with your chain. To debug this follow this sequence:

* Check that the account you are using to deply the contracts is in the genesis.json on your host (`cat ~/.eris/chains/simplechain/genesis.json`).
* Check that the genesis.json in the chain's data container matches the one in your `chain_dir` (`eris chains cat simplechain genesis`).

If these are not the same, then you will need to reset your chain:

```bash
eris chains stop -rxf simplechain
eris chains new simplechain --dir $chain_dir_this
```

If those are the same, the next thing to check is that the priv_validators are fine.

* Check the priv_validator.json (which is the key that the eris chain uses) on your host (`cat ~/.eris/chains/simplechain/priv_validator.json`).
* Check the priv_validator.json in the chain's data container matches the one in your `chain_dir` (`eris data exec simplechain "cat /home/eris/.eris/chains/simplechain/priv_validator.json`).

If they're different reset the chain (see above for instructions).

* Finally, check that the key in the priv_validator.json is registered in the genesis.json (the same address should be the single validator and also the key in the priv_validator.json). If the priv_validator.json is not in the genesis.json then make sure it is added to the accounts and the validators sections. See the [chain making](/tutorials/chain-making/) tutorial for instructions.

When that is done, reset the chain and you should be good to go.

<hr />

If you have an error which complains about how a container cannot be removed which looks something like this:

```irc
Error removing intermediate container 1f3a1d541241:
Driver btrfs failed to remove root filesystem
1f3a1d541241e757d48f34329508253e9ee139380b7b914a3b1104677eb0e8ee:
Failed to destroy btrfs snapshot: operation not permitted
```

Then rerun the `pkgs do` command with the `--rm` flag at the end which will stop the containers from trying to be removed as part of the tear down sequence.

**End Troubleshooting**

Since we have a deployed contract on a running chain, please do take a look at the available options for eris contracts with:

```bash
eris pkgs do -h
```

That's it! :-)

# Where to next?

**Next you'll want to interact with your contracts, so please see our [contract interaction tutorial](/tutorials/contracts-interacting/).**

If you would like to learn more about how to program smart contracts, please see [our smart contract tutorial series](/tutorials/solidity/).
