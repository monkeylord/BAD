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

 1. *Action Prefix* : What action the data is for. Typically it is first bytes of Hash(ApplicationName||DestinationPublicKey||SourcePublicKey||ActionName||OtherSharedSecret). However, any prefix is OK as long as application can map actions to it.
 
 2. *Application Data* : Raw application data which will be explained by application. Usually encrypted, but plaintext is OK.
 
 3. *Data Source Signature* : Signature of `Action Prefix||Application Data` signed by Data Source Public Key. This prevent untrusted data.

## Note

The Data Source Public Key play as a shared secret in application which is not public, and need not be public.

There is no intention for a unrelated third party to track/verify/analyse application data.

All of ApplicationName, DestinationPublicKey, SourcePublicKey, ActionName are shared secret. When kept secret, only users known the secret can locate the prefix. Combined with encryption on Application Data, a third party can only sniff trafics but not understand what data it is.

## Usage

### 1. Upload Application Data

Here's a example of building upload OP_RETURN in BSV.

~~~


~~~

### 2. Load Application Data

Here's a example of loading Application Data from BitDB.

~~~
~~~

### 3. Listener to Action

Here's a example of Listening Action from BitSocket.

~~~
~~~
