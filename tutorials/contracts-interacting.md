---

layout: docs
title: "Tutorials | Interacting with Contracts Using eris-contracts"

---

# Dependencies

This tutorial is a continuation of our [contracts deploying tutorial](/tutorials/contracts-deploying). If you have not completed that tutorial, please do so before working with this tutorial. Once we have some contracts deployed to our chain, we will want to interact with them. Indeed, that is the entire point.

So let us expand the very simple idiscontract out into a very simple smart contract backed appliction. To do this, we will be using node.js. To use this tutorial you will need a relatively recent version of [node](https://nodejs.org/en/download/package-manager/) installed on your box.

# Introduction

What we are going to make is a very simple application which tells the user what the value held in idi's contract currently is, then it will ask the user what the value should be changed to, once the user enters the new value then the application will change the value in idi's contract and display the new value for the user. It couldn't get any more simple.

# Set up the Application

As with all node.js applications, we will start by making a package.json. This should be made in the same folder as your `epm.yaml`. We will keep the `package.json` very simple.

```json
{
  "name": "idis_app",
  "version": "0.0.1",
  "dependencies": {
    "eris-contracts": "^0.13.1",
    "prompt": "*"
  }
}
```

Once you have saved your package.json then you will run (from the same directory) this command:

```bash
npm install
```

That will install eris-db.js and eris-contracts.js and their dependencies (plus another simple node.js package we'll use in the application).

**Troubleshooting**

If you are on Ubuntu 14.04 LTS, the version of NPM which will likely be installed will create an error when installing eris-contracts. Please see the [fix here](https://github.com/eris-ltd/eris-contracts.js#installation).

<hr />

If you are behind a firewall then you may need to let npm know about your proxy. To do that add a line to your ~/.npmrc:

```
proxy=http://myproxy.com
```

<hr />

If you get an error which looks like this:

```irc
> bufferutil@1.2.1 install /root/.eris/apps/idi/node_modules/bufferutil
> node-gyp rebuild

gyp ERR! build error
gyp ERR! stack Error: not found: make
gyp ERR! stack     at F (/usr/lib/node_modules/npm/node_modules/which/which.js:78:19)
```

Then you will need to install `make` on your platform with `apt-get install make` (or the equivalent package manager for your operating system).

**End Troubleshooting**

# Make the Main Application Script

Once we have that set up, then we'll make an `app.js` file and we'll add the following contents into it:

```javascript
// requires
var fs = require ('fs');
var prompt = require('prompt');
var erisC = require('eris-contracts');

// NOTE. On Windows/OSX do not use localhost. find the
// url of your chain with:
// docker-machine ls
// and find the docker machine name you are using (usually default or eris).
// for example, if the URL returned by docker-machine is tcp://192.168.99.100:2376
// then your erisdbURL should be http://192.168.99.100:1337/rpc
var erisdbURL = "http://localhost:1337/rpc";

// get the abi and deployed data squared away
var contractData = require('./epm.json');
var idisContractAddress = contractData["deployStorageK"];
var idisAbi = JSON.parse(fs.readFileSync("./abi/" + idisContractAddress));

// properly instantiate the contract objects manager using the erisdb URL
// and the account data (which is a temporary hack)
var accountData = require('./accounts.json');
var contractsManager = erisC.newContractManagerDev(erisdbURL, accountData.simplechain_full_000);

// properly instantiate the contract objects using the abi and address
var idisContract = contractsManager.newContractFactory(idisAbi).at(idisContractAddress);

// display the current value of idi's contract by calling
// the `get` function of idi's contract
function getValue(callback) {
  idisContract.get(function(error, result){
    if (error) { throw error }
    console.log("Idi's number is:\t\t\t" + result.toNumber());
    callback();
  });
}

// prompt the user to change the value of idi's contract
function changeValue() {
  prompt.message = "What number should Idi make it?";
  prompt.delimiter = "\t";
  prompt.start();
  prompt.get(['value'], function (error, result) {
    if (error) { throw error }
    setValue(result.value)
  });
}

// using eris-contracts call the `set` function of idi's
// contract using the value which was recieved from the
// changeValue prompt
function setValue(value) {
  idisContract.set(value, function(error, result){
    if (error) { throw error }
    getValue(function(){});
  });
}

// run
getValue(changeValue);
```

**N.B.** -- for *not Linux users*, please see the comments on lines 6-9 about the `var erisdbURL = "http://localhost:1337/rpc";` line of the script (spoiler alert, only do that on Linux).

The code should be self explanatory if you understand even a little bit of javascript. Once we properly instantiate all of the objects then there are three functions.

The first function, the `getValue` function will call the `get` function of `idi.sol` (see the previous tutorial for the code of that contract) and then it will display the result of that to the command line. This function takes a callback which fires after the result of the call to idi.sol's get function (which simply returns the `storedData`).

The second function is a simple function which will prompt the user to change the value (there is no validation here to make sure it is a number, so when playing with this just sure make it is a number). Once the user has entered what the value should be then the `setValue` function will be called.

The final function, the `setValue` function will call the `set` function of `idi.sol`. It will sign the transaction using the account data populated in the account.json file and then send it to the chain. It will block until the transaciton has been added to a block after which the callback will fire. The callback here is very simple in that it calls the `getValue` function to display what the result is after the `setValue` transaction has happened.

The beginning of the script, which gets everything sets up includes this line:

```javascript
var contractData = require('./epm.json');
```

But in the [previous tutorial](/tutorials/contracts-deploying/) we only worked with an `epm.yaml`, not an `epm.json`. So what is the `epm.json`? That file is an artifact of the `eris pkgs do` process. If you look at the `epm.json` file it should look something like this:

```javascript
{
  "account": "1FDD813D68F73BBABFEA6EF6FB83118441CFC347",
  "assertStorage": "passed",
  "deployStorageK": "41672393A960D3A706C8345B5149961F8A755BBC",
  "queryStorage": "5",
  "setStorage": "67394093FD93BB4C8BEE2C1CB38BC454AAA56E86",
  "setStorageBase": "5"
}
```

The json file is the result of each of the jobs. What we really need from this file is the contracts address that was deployed (the key to the `deployStorageK` field) so that the app.js script knows what contract on the chain it should be "talking" to.

We need to do one final thing before we finish this section. We need to copy over the accounts.json which was an artifact of the chain making process and is included in our chains directory into this directory so that it can be consumed by the eris-contracts.js library.

```bash
cp $chain_dir/accounts.json .
```

**Troubleshooting**

If you do not have an `epm.json` file that means there was a problem with the contracts deploy. Please resolve that problem by carefully following the [previous tutorial](/tutorials/contracts-deploying/) before continuing with this tutorial.

**End Troubleshooting**

For more about eris-contracts please see [the eris-contracts documentation](/documentation/eris-contracts-js/).

# Run The Application

Now we are ready to go:

```bash
node app.js
```

**Troubleshooting**

If you just started your chain you may not have any contracts on it. To solve this, run:

```
eris pkgs do --chain $chainname --address $addr
```

Where `$chainname` is the name of the chain you want to use and `$addr` is the address of the account you would like to use. [See the contracts deploying tutorial for more information](/tutorials/contracts-deploying/).

When you do the deploy command you may get an error which looks like this:

```irc
Error creating data container API error (500): Invalid container name (eris_data_idis app_tmp_deploy_1), only [a-zA-Z0-9][a-zA-Z0-9_.-] are allowed.
```

This is an "error" from Docker. What it means is that you have a space in the `"name"` field of your package.json which is used by eris to setup the container. We will control for this error in the future, but for now just replace the space in the name field with an underscore `_`.

When you run the app.js if you get an error which looks like this:

```irc
Error callback from sendTransaction
/home/ubuntu/.eris/apps/idi/app.js:29
    if (error) { throw error }
                       ^
Error: Error: socket hang up
    at createHangUpError (http.js:1472:15)
    at Socket.socketOnEnd [as onend] (http.js:1568:23)
    at Socket.g (events.js:180:16)
    at Socket.EventEmitter.emit (events.js:117:20)
    at _stream_readable.js:920:16
    at process._tickCallback (node.js:415:13)
    at httpRequest.onreadystatechange (/home/ubuntu/.eris/apps/idi/node_modules/eris-db/lib/rpc/http.js:101:26)
    at dispatchEvent (/home/ubuntu/.eris/apps/idi/node_modules/xmlhttprequest/lib/XMLHttpRequest.js:572:25)
    at setState (/home/ubuntu/.eris/apps/idi/node_modules/xmlhttprequest/lib/XMLHttpRequest.js:591:14)
    at handleError (/home/ubuntu/.eris/apps/idi/node_modules/xmlhttprequest/lib/XMLHttpRequest.js:516:5)
    at ClientRequest.errorHandler (/home/ubuntu/.eris/apps/idi/node_modules/xmlhttprequest/lib/XMLHttpRequest.js:443:14)
    at ClientRequest.EventEmitter.emit (events.js:95:17)
    at Socket.socketOnEnd [as onend] (http.js:1568:9)
    at Socket.g (events.js:180:16)
    at Socket.EventEmitter.emit (events.js:117:20)
    at _stream_readable.js:920:16
```

What that means is that the api port for your chain is not running. There is a very easy fix for this:

```bash
eris chains stop -rf $chainname
eris chains start $chainname
```

Where `$chainname` in the above sequence is the name of the chain you are using. What those commands will do is to first stop and remove the service container for the chain (this will leave its data container) and then it will restart the chain's service container but when it does so, eris will make sure the api port for your chain is running.

**N.B.** In eris 0.11 the official release we automatically turn on the API of an eris:db style chain. If you are using one of the release candiates (0.11.0-rc1 or 0.11.0-rc2) for 0.11 you will need to change the above start command to `eris chains start -a $chainname` rather than simply `eris chains start $chainname`.

**End Troubleshooting**

The first time you run the script it should tell you that the value is `5` or whatever value you entered into the `setStorageBase` job of the epm.yaml from the [previous tutorial](/tutorials/contracts-deploying). Then it will prompt you to change the value. The second time you run the script it should tell you that the value is whatever you entered the first time and so on.

Congratulations, you've just made your very own smart contract backed application on a permissioned blockchain!

# Where to next?

**Next you'll want to see how to a make a [longer running application](/tutorials/services-making/).**
