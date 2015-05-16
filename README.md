## stratum-php
[![Build Status](https://travis-ci.org/Bit-Wasp/stratum-php.svg?branch=master)](http://travis-ci.org/Bit-Wasp/stratum-php)
[![Code Coverage](https://scrutinizer-ci.com/g/bit-wasp/stratum-php/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/bit-wasp/stratum-php/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/Bit-Wasp/stratum-php/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/Bit-Wasp/stratum-php/?branch=master)
 
Implementation of the Stratum protocol (for electrum and mining) using ReactPHP

Currently this library supports a TCP transport to stratum servers. 
Examples of these and other servers can be found on the Electrum server list.

Stratum enables rather stateless wallets to be built with minimal effort, and depending on the use-case could replace running a full node.

https://docs.google.com/document/d/17zHy1SUlhgtCMbypO8cHgpWH73V5iUQKk_0rWvMqSNs/edit?hl=en_US
https://electrum.orain.org/wiki/Stratum_protocol_specification

### Electrum commands
  - server.banner
  - server.version [$clientVersion, $protocolVersion] -  Client sends its own version and version of the protocol it supports. Server responds with its supported version of the protocol 
  - server.donation_address - Return a servers donation address; can be empty. 
  - server.peers.subscribe - (not a subscription) Returns a list of stratum servers connected on IRC
  - blockchain.numblocks.subscribe - Return the current height of the chain
  - blockchain.transaction.broadcast [$transactionHex] - broadcast the transaction, return the transaction id
  - blockchain.transaction.get_merkle [$txid, $txHeight]
  - blockchain.transaction.get [$txid]
  - blockchain.address.subscribe [$address]
  - blockchain.address.get_history [$address]
  - blockchain.address.get_balance [$address] - return an array of confirmed 
  - blockchain.address.get_proof [$address]
  - blockchain.address.listunspent [$address] - return an array of [{'txhash':..,'tx_pos':..,'value':..,'height':...},...]
  - blockchain.utxo.get_address [$txid, $nOutput] - return the address for this transaction output
  - blockchain.block.get_header [$blockHeight] - return the header for the given block height
  - blockchain.block.get_chunk [$blockHeight] - return the block hex for the given block
  - mining.subscribe - client subscribes for work
  - mining.authorize [$username, $password] - authorize a worker
  - mining.submit [$username, $jobId, $extraNonce2, $nTime, $nonce]

### Example
```php

use \BitWasp\Bitcoin\Stratum\StratumClient;
use \BitWasp\Bitcoin\Stratum\Request\RequestFactory;

$requestFactory = new RequestFactory;

$host = 'electrum3.hachre.de';
$port = 50001;

// Initialize react event loop, resolver, and connector
$loop = React\EventLoop\Factory::create();
$dnsResolverFactory = new React\Dns\Resolver\Factory();
$dns = $dnsResolverFactory->create('8.8.8.8', $loop);
$connector = new \React\SocketClient\Connector($loop, $dns);

// Initialize TCP transport for stratum client
$tcp = new \BitWasp\Bitcoin\Stratum\Connector\Tcp($connector, $host, $port);
$stratum = new StratumClient($tcp, $requestFactory);

// Make the query, receive a Promise
$t = $stratum->request('blockchain.transaction.get', ['2439243c47803a613728beab5ccfd7a426c9bfdd069d463b28f6f49915801988']);
$t->then(function ($response) {
    echo $response;
});

$loop->run();

```
