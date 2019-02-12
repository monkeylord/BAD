# BAD

> Bitcoin Application Data Protocol

Store and load application data on the Bitcoin SV blockchain

## Intro

BAD is an OP_RETURN protocol to store and load arbitrary application data on Bitcoin.

The design goal:

* The low-layer protocol to store application data to the blockchain securely.

* A protocol to load application data from blockchain securely.

* Data producer and data consumer are saperated.

* Basic authenication

## Protocol

* unlike B protocol, BAD protocol use dynamic Action Prefix instead of fixed prefix.

Here is a example of what BAD protocol looks like:

~~~

OP_RETURN
  [Action Prefix]
  [Application Data]
  [Data Source Signature]

~~~

The order is `Action Prefix`, `Application Data` , `Data Source Signature`, and as you can see, `Data Source Public Key` is not included deliberately.

 1. *Action Prefix* : What action the data is for. Typically it is first bytes of Hash(ApplicationName||Destination||Source||ActionName||OtherSharedSecret). However, any prefix is OK as long as application can map actions to it.

 2. *Application Data* : Raw application data which will be explained by application. Usually encrypted, but plaintext is OK.

 3. *Data Source Signature* : Signature of `Action Prefix||Application Data` signed by Data Source Public Key. This prevent untrusted data.

## Note

The Data Source Public Key play as a shared secret in application which is not public, and need not be public.

There is no intention for a unrelated third party to track/verify/analyze application data.

All of ApplicationName, Destination, Source, ActionName are shared secret. When kept secret, only users known the secret can locate the prefix. Combined with encryption on Application Data, a third party can only sniff traffics but not understand what data it is.

BAD protocol only provide basic reliability. Application should parse application data by itself.

## Usage

> Note: It's not necessary to use ECDSA in BAD protocol, as ECDSA reveal signer's public key.
>
> Any Signing algorithm is OK in OP_RETURN as long as application support it.

### 1. Upload Application Data

Here's a example of building upload OP_RETURN in BSV.

~~~javascript
var bsv = require('bsv')
require('bsv/message')

var sitePublicKey = '02039de46159d2f3bc4273010c87ff9f6cff7e1aac2581f5e70bf043d987d8b612'
var AdminKey = '9f4fa66a17d8aecf1e755088fd7d7dfdc26ca740a6d12a314bce453046f5d7f5'

var sharedSecret = {
    application : 'App1',
    destination : sitePublicKey,
    source : 'admin',
    action : 'post'
}

var actionPrefix = bsv.crypto.Hash.sha256sha256(Buffer.from(JSON.stringify(sharedSecret)))
var appData = 'Data posted by admin'
var signature = bsv.Message(appData).sign(bsv.PrivateKey('9f4fa66a17d8aecf1e755088fd7d7dfdc26ca740a6d12a314bce453046f5d7f5'))

//Build Output
var script = new bsv.Script.buildDataOut(actionPrefix).add(Buffer.from(appData)).add(Buffer.from(signature))
var output = new bsv.Transaction.Output({
    satoshis:0,
    script:script
  })

~~~

### 2. Load Application Data

Here's a example of loading Application Data from BitDB.

~~~javascript
var bsv = require('bsv')
require('bsv/message')

var sitePublicKey = '02039de46159d2f3bc4273010c87ff9f6cff7e1aac2581f5e70bf043d987d8b612'
var AdminPubKey = '03ab22d5c039a797c473857f6d94afd9d036d062c4e22a8d1d3a7727ce1f97a9c0'

var sharedSecret = {
    application : 'App1',
    destination : sitePublicKey,
    source : 'admin',
    action : 'post'
}

var actionPrefix = bsv.crypto.Hash.sha256sha256(Buffer.from(JSON.stringify(sharedSecret)))

var query = {
    "v": 3,
    "q": {
        "find": { "out.h1": actionPrefix.toString('hex')},
    },
    "r": {
        "f": "[ .[] | {appData: .out[0].s2, signature: .out[0].s3, txid: .tx.h, time: .blk.i} ]"
    }
}

/*
Fetch Data from BitDB
*/
AppDataResults = fetchResults.c
    .sort((a,b)=>a.time-b.time)
    .filter((entry)=>bsv.Message(entry.appData)
        .verify(bsv.PublicKey(AdminPubKey).toAddress(),entry.signature)
	)
    .map((entry)=>entry.appData)
~~~

### 3. Listen to Actions

Same to `Load Application Data`