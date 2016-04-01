*Are you looking for the Marbles app demo?  Thats not here, head to the [marbles example](https://github.com/IBM-Blockchain/marbles)* 

# ibm-blockchain-js
This is a Node.js library for easier interaction IBM Blockchain chaincode. 
All documentation is on this page.

Table Of Contents:

1. [IBC-js Function Documentation](#ibcjs)
1. [Chaincode Functions](#ccfunc)
1. [Object Formats](#formats)
1. [Chaincode Summary File](#ccsf)
1. [FAQ](#faq)

***

## Installation

```
npm install ibm-blockchain-js
```

***

## Usage Steps!
(example code also provided below)

1. Require this module
1. Pass network + chaincode parameters to ibc.load(options, my_cb):
1. Receive chaincode obj from callback to ibc.load(). ie: my_cb(e, chaincode)
1. You can now deploy your chaincode (if needed) with chaincode.deploy(func, args, null, cb)
1. Use dot notation on chaincode to call any of your chaincode functions ie:

```js
		// The functions below need to exist in your actual chaincode GoLang file(s) 
		chaincode.read('a', cb);			//will read variable "a" from current chaincode state
		chaincode.query(['test'], cb);		//will read the variable "test"
		chaincode.write('a', "test", cb)	//will write to vairable "a"
		chaincode.remove('a', cb)			//will delete variable "a"
		chaincode.init_marbles(ARGS, cb);	//calls my custom chaincode function init_marbles() and passes it ARGS
```

## Example

```js
	// Step 1 ==================================
	var Ibc1 = require('ibm-blockchain-js');
	var ibc = new Ibc1();
	var chaincode = {};

	// ==================================
	// configure ibc-js sdk
	// ==================================
	var options = 	{
		network:{
			peers:   [{
				"api_host": "xxx.xxx.xxx.xxx",
				"api_port": "xxxxx",
				"id": "xxxxxx-xxxx-xxx-xxx-xxxxxxxxxxxx_vpx",
				"api_url": "http://xxx.xxx.xxx.xxx:xxxxx"
			}],
			users:  [{
				"username": "user1",
				"secret": "xxxxxxxx"
			}],
			options: {							//this is optional
				quiet: true, 
				timeout: 60000
			}
		},
		chaincode:{
			zip_url: 'https://github.com/ibm-blockchain/marbles-chaincode/archive/master.zip',
			unzip_dir: 'marbles-chaincode-master/part2',
			git_url: 'https://github.com/ibm-blockchain/marbles-chaincode/part2'
		}
	};
	
	// Step 2 ==================================
	ibc.load(options, cb_ready);

	// Step 3 ==================================
	function cb_ready(err, cc){								//response has chaincode functions
		app1.setup(ibc, cc);
		app2.setup(ibc, cc);
	
	// Step 4 ==================================
		if(cc.details.deployed_name === ""){				//decide if i need to deploy or not
			cc.deploy('init', ['99'], './cc_summaries', cb_deployed);
		}
		else{
			console.log('chaincode summary file indicates chaincode has been previously deployed');
			cb_deployed();
		}
	}

	// Step 5 ==================================
	function cb_deployed(err){
		console.log('sdk has deployed code and waited');
		chaincode.read('a');
	}
```
	
***
***

## <a name="ibcjs"></a>IBM-Blockchain-JS Documentation
### ibc.load(options, [callback])
This is a function that wraps a typical startup using a standard Bluemix IBM Blockchain network. 
Take a look at how this function works, especially how it uses the register() function. 
If this is not applicable for your network (ie you have a custom IBM Blockchain network) you can easily create your own version of `ibc.load()` for your needs. 
It will run in order:

1. ibc.network() 
1. ibc.register() 
	- It will register the first peer with the first username, the 2nd peer against the 2nd username and so on.
	- This funciton only runs if valid users are found in options.network.users. A valid user is one that contains 'type_1'.
	- Any errors in register will stop execution and run callback(err).
1. ibc.load_chaincode() 
1. callback(err, cc) 

Options Parameter:

```js
	var options = 	{
		network:{
			peers:   [{
				"api_host": "xxx.xxx.xxx.xxx",
				"api_port": "xxxxx",
				"api_url": "http://xxx.xxx.xxx.xxx:xxxxx"
				"id": "xxxxxx-xxxx-xxx-xxx-xxxxxxxxxxxx_vpx",
			}],
			users:  [{
				"username": "user1",
				"secret": "xxxxxxxx"
			}],
			options: {						//this is optional
				quiet: true, 
				timeout: 60000
			}
		},
		chaincode:{
			zip_url: 'https://github.com/ibm-blockchain/marbles-chaincode/archive/master.zip', //http/https of a link to download zip
			unzip_dir: 'marbles-chaincode-master/part2',                                        //name/path to folder that contains the chaincode you want to deploy (path relative to unzipped root)
			git_url: 'https://github.com/ibm-blockchain/marbles-chaincode/part2',             //git https URL. should point to the desired chaincode repo AND directory
			
			deployed_name: null    //[optional] this is the hashed name of a deployed chaincode.  if you want to run with chaincode that is already deployed set it now, else it will be set when you deploy with the sdk
		}
	};
```

### ibc.load_chaincode(options, [callback])
Load the chaincode you want to use. 
It will be downloaded and parsed. 
The callback will receive (e, obj) where e is the error format and obj is the chaincode object.
"e" is null when there are no errors.
The chaincode object will have dot notation to the functions in the your chaincode. 

Example
	
```js
	var options = 	{
		zip_url: 'https://github.com/ibm-blockchain/marbles-chaincode/archive/master.zip', //http/https of a link to download zip
		unzip_dir: 'marbles-chaincode-master/part2',                                        //name/path to folder that contains the chaincode you want to deploy (path relative to unzipped root)
		git_url: 'https://github.com/ibm-blockchain/marbles-chaincode/part2',             //git https URL. should point to the desired chaincode repo AND directory
		
		deployed_name: null   //[optional] this is the hashed name of a deployed chaincode.  if you want to run with chaincode that is already deployed set it now, else it will be set when you deploy with the sdk
	};
	ibc.load_chaincode(options, cb_ready);
```

### ibc.network(arrayPeers, [options])
Set the information about the peers in the network.
This should be an array of peer objects. 
The optional options parameter should be an object with the field `quiet` and/or `timeout`.
- quiet = boolean - when true will print out only minimal HTTP debug information. Default `true`.
- timeout = integer - time in ms to wait for a http response. Default `60000`.

Ex:

```js
	var peers = [
		{
			"api_host": "xxx.xxx.xxx.xxx",
			"api_port": "xxxxx",
			"id": "xxxxxx-xxxx-xxx-xxx-xxxxxxxxxxxx_vpx",
			"api_url": "http://xxx.xxx.xxx.xxx:xxxxx"
		}
	]
	ibc.network(peers, {quiet: false, timeout: 120000});
```

### ibc.save(path [callback])
Save the [Chaincode Summary File](#ccsf) to a path.

Ex:

```js
	ibc.save('./');
```

### ibc.clear([callback])
Clear any loaded chaincode files including the downloaded chaincode repo, and [Chaincode Summary File](#ccsf).

Ex:

```js
	ibc.clear();
```

### ibc.chain_stats([callback])
Get statistics on the network's chain.  

Ex:

```js
	ibc.chain_stats(my_callback);
	function my_callback(e, stats){
		console.log('got some stats', stats);
	}
```

Example Chain Stats:

```js
	{
		"height": 10,
		"currentBlockHash": "n7uMlNMiOSUM8s02cslTRzZQQlVfm8wKT9FtL54o0ywy6BkvPMwSzN5R1tpquvqOwFFHyLSoW44n6rkFyvAsBw==",
		"previousBlockHash": "OESGPzacJO2Xc+5PB2zpmYVM8XlrwnEky0L2Ghok9oK1Lr/DWoxuBo2WwBca5zzJGq0fOeRQ7aOHgCjMupfL+Q=="
	}
```

### ibc.block_stats(id, [callback])
Get statistics on a particular block in the chain.  

Ex:

```js
	ibc.block_stats(my_callback);
	function my_callback(e, stats){
		console.log('got some stats', stats);
	}
```

Example Block Stats:

```js
	{
		"transactions": [
			{
				"type": 3,
				"chaincodeID": "EoABNWUzNGJmNWI1MWM1MWZiYzhlMWFmOThkYThhZDg0MGM2OWFjOWM5YTg4ODVlM2U0ZDBlNjNiM2I4MDc0ZWU2NjY2OWFjOTAzNTg4MzE1YTZjOGQ4ODY4M2Y1NjM0MThlMzMwNzQ3ZmVhZmU3ZWYyMGExY2Q1NGZmNzY4NWRhMTk=",
				"payload": "CrABCAESgwESgAE1ZTM0YmY1YjUxYzUxZmJjOGUxYWY5OGRhOGFkODQwYzY5YWM5YzlhODg4NWUzZTRkMGU2M2IzYjgwNzRlZTY2NjY5YWM5MDM1ODgzMTVhNmM4ZDg4NjgzZjU2MzQxOGUzMzA3NDdmZWFmZTdlZjIwYTFjZDU0ZmY3Njg1ZGExORomCgtpbml0X21hcmJsZRIHcng2YXRzcBIFZ3JlZW4SAjM1EgNCb2I=",
				"uuid": "b3da1d08-19b8-4d8c-a116-b46defb07a7c",
				"timestamp": {
					"seconds": 1453997627,
					"nanos": 856894462
				}
			}
		],
		"stateHash": "81ci8IAOeDh0ZwFM6hE/b3SfXt4tnZFemib7sI95cOsNcYMmtRxBWRBA7qnjPOCGU6snBRsFVnAliZXUigQ03w==",
		"previousBlockHash": "tpjUh4sgbaUQFO8wm8S8nrm7yCrBa4rphIiujfaYAlEVfzI8IZ0mjYMf+GiOZ6CZRNWPmf+5bekmGIfr0H6zdw==",
		"nonHashData": {
			"localLedgerCommitTimestamp": {
			"seconds": 1453997627,
			"nanos": 868868790
			}
		}
	}
```

### ibc.switchPeer(peerIndex)
The SDK will default to use peer[0].  This function will switch the default peer to another index.  

Ex:
	
```js
	ibc.switchPeer(2);
```
	
### ibc.register(peerIndex, enrollID, enrollsecret, [callback])
Only applicable oo a network with security enabled. 
register() will register against peer[peerIndex] with the provided credentials.
If successful the peer will now use this enrollID to perform any http requests.

Ex:
	
```js
	ibc.register(3, 'user1', 'xxxxxx', my_cb);
```

### ibc.monitor_blockheight(callback)
This will call your callback function whenever the block height has changed.
ie whenever a new block has been written to the chain.
It will also pass you the same response as in `chain_stats()`.

Ex:

```js
	ibc.monitor_blockheight(my_callback);
	function my_callback(e, chainstats){
		console.log('got a new block!', chainstats);
	}
```

***
***

##<a name="ccfunc"></a>Chaincode Functions
- Chaincode functions are dependent on actually be found inside your Go chaincode
- My advise is to build your chaincode off of the Marble Application one.  This way you get the basic CRUD functions below:

### chaincode.read(name, [username], [callback])
Read variable named name from chaincode state. 
This will call the `Query()` function in the Go chaincode, therefore the `Query()` function needs to exists in the cc. 
The variable name will be passed as `arg[0]` to `Query()`. 
The `username` parameter should be the desired secure context username that has already been registered against the selected peer. 
If left `null` the SDK will use a known username for the selected peer. (this is only relevant in a permissioned network)

### chaincode.query(args, [username], [callback])
This will call the query function with custom input arguments. 
Usually "args" is an array of strings. 
The `username` parameter should be the desired secure context username that has already been registered against the selected peer. 
If left `null` the SDK will use a known username for the selected peer. (this is only relevant in a permissioned network)

### chaincode.write(name, val, [username], [callback])
Write 'val' to variable named 'name'. This will call the `write()` function in the Go chaincode, therefore the `write()` function needs to exists in the cc.
The `username` parameter should be the desired secure context username that has already been registered against the selected peer. 
If left `null` the SDK will use a known username for the selected peer. (this is only relevant in a permissioned network)

### chaincode.remove(name, [username], [callback])
Delete variable named 'name'. This will call the `delete()` function in the Go chaincode, therefore the `delete()` function needs to exists in the cc.
The `username` parameter should be the desired secure context username that has already been registered against the selected peer. 
If left `null` the SDK will use a known username for the selected peer. (this is only relevant in a permissioned network)

### chaincode.deploy(func, args, [save_path], [username], [callback])
Deploy the chaincode. 
Call GoLang function named 'func' and feed it 'args'.
Optionally save [Chaincode Summary File](#ccsf) to 'save_path'. 
Usualy "args" is an array of strings.
The `username` parameter should be the desired secure context username that has already been registered against the selected peer. 
If left `null` the SDK will use a known username for the selected peer. (this is only relevant in a permissioned network)

### chaincode.CUSTOM_FUNCTION_NAME(args, [username], [callback])
Will invoke your Go function CUSTOM_FUNCTION_NAME and pass it 'arg'. 
Usualy "args" is an array of strings.
The `username` parameter should be the desired secure context username that has already been registered against the selected peer. 
If left `null` the SDK will use a known username for the selected peer. (this is only relevant in a permissioned network)

***
***

##<a name="formats"></a>Formats
### Chaincode Object
This is the main guy.
It is returned in the callback to load_chaincode() and contains all your cc functions + some of the setup/input data.

```js
	chaincode = 
		{
			CUSTOM_FUNCTION_NAME1: function(args, cb){etc...};	//call chaincode function and pass it args
			CUSTOM_FUNCTION_NAME2: function(args, cb){etc...};
			CUSTOM_FUNCTION_NAME3: function(args, cb){etc...};
			^^ etc...
			read: function(name, cb),                   //read variable
			query: function(args, cb),                  //^^
			write: function(name, value, cb),           //write/create variable
			remove: function(name, cb),	                //delete variable
			deploy: function(func, args, path, cb),     //deploy loaded chaincode
			details:{                                   //input options get stored here, sometimes handy
						deployed_name: '',              //hash of deployed chaincode
						func: [],                       //array of function names found
						git_url: '',
						peers: [],                      //peer list provided in network()
						timestamp: 0,                   //unix timestamp in ms of parsing
						users: [],                      //users provided in load()
						vars: [],                       //tbd
						unzip_dir: '',
						zip_url: '',
			}
		};
```

### Error Format

```js
	{
		name: "input error",
		code: 400,
		details: {msg: "did not provide git_url"}
	};
```
	
### <a name="ccsf"></a>Chaincode Summary File
This file is used internally. 
It is created in ibc.load_chaincode() and updated with chaincode.deploy(). 
A copy can be saved else where with ibc.save(path). 
I found it handy in niche cases, but it will probably be unhelpful to most developers. 

```js
	{
	"details": {
		"deployed_name": "f6c084c42b3bde90c03f214ac6e0426e3e594807901fb1464287f2c3a18ade717bc495298958287594f81bb0d0cfdd3b4346d438d3b587d4fc73cf78ae8f7dfe",
		"func": ["init", "Delete", "Write", "init_marble", "set_user", "open_trade", "perform_trade", "remember_me"],
		"unzip_dir": 'marbles-chaincode-master/part2',
		"git_url": 'https://github.com/ibm-blockchain/marbles-chaincode/part2'
		"peers": [{
			"name": "vp1-xxx.xxx.xxx.xxx",
			"api_host": "xxx.xxx.xxx.xxx",
			"api_port": "xxx",
			"id": "xxxxx_vp1",
			"ssl": false,
			"user": "user1"
		}],
		"vars": [],
		"zip_url": 'https://github.com/ibm-blockchain/marbles-chaincode/archive/master.zip',
		}
	}
```



#FAQ
- *ibc.load() appears to ignore all of my users for secure context. Then it complains it found no usernames and never registers with a Peer!*

Correct behavior of `ibc.load()` is to remove any usernames that do not contain 'type_1' in their name. 
This is to conform to the OBC Peer spec of what usernames a dev's app should use. 
If this is not applicable for your network (ie you have a custom IBM Blockchain network) you can easily create your own version of `ibc.load()` for your needs. 
I would copy the code found in `ibc.load()` then modifiy it to fit your own needs. 
Everything important that `ibc.load()` calls is exposed in this module. 

- *Do you have any examples that use this?*

Yes! Head over to the [Marbles Node.js Demo](https://github.com/IBM-Blockchain/marbles)