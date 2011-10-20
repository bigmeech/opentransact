# OpenTransact Core 1.0
##Abstract
This document specifies the OpenTransact standard for requesting and performing transfers of assets over the http protocol.
##Status of Document
This document is in draft and reflects the current designs from the OpenTransact mailing list.
##1. Introduction
The goal is to develop an extremly simple low level standard for transfering an amount of an asset from one account to another.

Most payment systems and existing standards use the messaging paradigm for historical reasons. OpenTransact specifically rejects the message paradigm and prefers the restful resource approach as used on the web with URL's and HTTP at it's core.

We aim to create a new standard from scratch ignoring all legacy systems, while leaving it flexible enough to allow applications built on top of it to deal with legacy systems.

The standard is designed to follow standard RESTful practices and be concise and human readable.

###1.1 Assets
Within OpenTransact we use the accounting definition of the word "Asset" as anything of value. Eg. 
- money
- shares
- bonds
- mobile phone minutes
- hours of work
- property
- domain names
- physical products etc.

####1.1.1 Asset Service
An Asset Service is a service maintained by an organization to manage accounts of one assets. For money and other financial assets the Asset Service would normally be run by a Financial Service Provider of some type. However there are many types of assets that could be offered by non financial services.

####1.1.2 Transaction URL
Each Asset Service has a unique transaction URL. This allows us a  don't need to get into details in our standard about application specific details like currency, card type, size, color etc.

This transaction URL would follow basic REST practices.

- A transaction URL should be a simple clean URL like http://eepay.com/transactions.
- A POST to the URL is used for creating a transaction transfering value.
- A GET to the URL from a normal html web browser should present a human readable description of the asset.
- A GET to the URL from a normal web browser with specific parameters becomes a payment request that the user can authorize.
- A GET to the URL in a machine readable form such as json returns meta data about the asset an optionally a list of transactions that the current user is allowed to see.
- Each transaction has a unique URL eg. http://epay.com/transactions/aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d.

#### 1.1.3 Example of Asset Services

Lets say it's an imaginary electronic currency eepay.com they only offer one asset type, their currency. So they would only have one transaction url:

* http://eepay.com/transactions

If it offered multiple currencies it should have a url for each currency as they are really separate asset types:

* http://eepay.com/transactions/usd
* http://eepay.com/transactions/euro

If a larger existing bank offered various kinds of services each one would have it's own URL:

* http://megabank.com/current
* http://megabank.com/savings
* http://megabank.com/bonds/mortgage

A mutual fund company would have a url for each of their funds.

* http://fidelify.com/funds/sp500
* http://fidelify.com/funds/emergingmarkets

A broker could actually also implement an OpenTransact API and have a different URL for each symbol eg:

* http://feetrade.com/trade/AAPL
* http://feetrade.com/trade/EBAY

This would allow them to create their own internal market.

You could even imagine with GoDaddy's new patent for offering shares in domain names they could do something like this for each of their domains offered as shares:

* http://godaddyexchange.com/domains/stakeventures.com/shares

If we let the URL do the describing, there are tons of different possibilities. We can keep the core API very simple, while allowing pretty open support for all manners of asset types.

All the above examples are fungible assets. In general it is best practice that one item of value for one asset is fungible for one another.

For unique items such as domain names, property titles and diamonds that are unique and infungible, we can still create asset urls for each item, but only accept transfer amounts of 1.

###1.2 Roles
OpenTransact defines 4 roles:
- Asset provider
	- The entity providing or issuing the asset
- Transferer
	- The entity transfering an amount of the asset
- Transferee
	- The entity receiving an amount of the asset
- 3rd party application
	- An application performing a transfer on behalf of the Transferer

###1.3 Transfer 
We will use the term "Transfer" as it is more widely applicable than "Payment".

A Transfer is legally a transfer in owner ship of some amount of the Asset from the Transferer to the Transferee.

Eg. A Payment of $12 from Bob to Alice is a Transfer of 12$ with Bob being the Transferer and Alice the Transferee.

###1.4 Transfer Request
A Transfer request is when the Transferee requests a transfer of a given amount of an asset from the Transferer.

Eg. Alice sends an invoice to Bob for $12. This is a Transfer Request from Alice to Bob where Alice is the Transferee and Bob the Transferer.

A Transfer Request is simply a specially formatted payment link that takes Bob to Asset Service if followed. The Asset Service shall present the Transfer Request to Bob who can authorize or decline it.

###1.5 Transfer Authorization Request
A Transfer Authorization Request is when  the Transferee or a third party application requests an authorization to transfer of a given amount of an asset from the Transferer.

Eg. Bob wants to hire someone on an online job board to edit a document for $33. The Job Board creates a Transfer Authorization Request link. Bob follows this link to the Asset Service and authorizes the Job Board to transfer $33 from his account to some one else within a time period.

A Transfer Request is simply a specially formatted payment link that takes Bob to Asset Service if followed. The Asset Service shall present the Transfer Request to Bob who can authorize or decline it. If Bob authorizes it the Asset Service issues an authorization code to the Job Board that they can use to exchange for an OAuth token.

###1.6 Exchange Transaction
A Transfer is often but not always part of an exchange of value between 2 types of assets. Eg.:
- 10 consulting hours exchanged for 1100EUR
- 10 USD exchanged for 8 EURO
- 0.99 USD exchanged for an mp3 song

There are as many different exchange mechanisms for create exchanges as there are exchange types. eg.
- Invoicing system
- App Store
- Web shop
- Auction site
- Stock Exchange

It is outside the scope for OpenTransact to define every single type of exchange that is possible. However OpenTransact should be able to be a fundamental building block in building such systems. It should also integrate well with exchange systems that don't yet understand OpenTransact.

####1.6.1 Exchanged item URI
In an Exchange Transaction we can include a url to the exchanged item. This URI could either be a link to the exchanged item itself or the unique URI identifying the transaction itself.

####1.6.2 Authorization
A useful building block for creating Exchange services is the Transfer Authorization Request [1.5].

### 1.7 Vocabulary
As part of OpenTransact we are creating a simple vocabulary of parameter names that can be used in various parts of the standard.

- *asset* - Transaction URL
- *from* - Transferer account identifier
- *to* - Transferee account identifier
- *amount* - Amount of asset
- *note* - Textual description of transfer
- *for* - URI of exchanged item
- *redirect_uri* - URI to redirect User Agent to
- *callback_uri* - URI for performing callback after request
- *client_id* - Identifier of 3rd party application

They are designed to be small and semantically correct. eg

> A transfer of 10  USD  from Bob to Alice for consulting.

Would become the following:
- amount=10
- asset=http://pay.me/usd
- from=bob@test.com
- to=alice@test.com
- note=Consulting on XYZ project
- for=http://myinvoice.test/invoices/123123

### 1.8 Authentication
OpenTransact does not define any new authentication mechanisms, but relies on the Asset Provider's existing mechanisms for authenticating the Transferer and  OAuth 2.0 [I-D.ietf-oauth-v2] for authenticating 3rd party applications on behalf of the Transferer.

### 1.9 Parameter encoding
Since OpenTransact is designed to be simple to implement, the basic parameter encoding is URL form encoding.

Any data responses should be in JSON format.

### 1.10 Extensions
OpenTransact is designed to be extensible, either through proprietary extensions, conventions or futher standards.

For example it may be useful to follow the lat/lon convention allowing geotagging of data.

You could also add digital signatures ontop of the standard creating a triple entry book entry system[IANG]

###1.11 Notational Conventions
The ke	y words 'MUST', 'MUST NOT', 'REQUIRED', 'SHALL', 'SHALL NOT', 'SHOULD', 'SHOULD NOT', 'RECOMMENDED', 'MAY', and 'OPTIONAL' in this specification are to be interpreted as described in [RFC2119].

This specification uses the Augmented Backus-Naur Form (ABNF) notation of [RFC5234].

Certain security-related terms are to be understood in the sense defined in [RFC4949]. These terms include, but are not limited to, 'attack', 'authentication', 'authorization', 'certificate', 'confidentiality', 'credential', 'encryption', 'identity', 'sign', 'signature', 'trust', 'validate', and 'verify'.

Unless otherwise noted, all the protocol parameter names and values are case sensitive.

##2. Transfer Request

A Transfer Request consists of a GET to the Asset URL.

    GET /usd?to=bill@example.com&amount=100.00&note=Milk&redirect_uri=http://site.com/callback HTTP/1.1
    Host: pay.me

We use the following parameters from our common vocabulary in [1.6]. All fields are optional:

-*to* Account identifier of Transferee. If left out it defaults to the 3rd party applications own account on Asset Service or a predefined account as specified when authorizing the access token.
-*amount* Amount as a number with decimal points. Symbols are allowed but SHOULD be ignored. If left out it defaults to the Asset's minimum transfer, 1 or an amount predefined when authorizing the access token.
-*note* Textual description, which can include hash tags. Asset Service may truncate this. No default.
-*from* Account identifier of Transferer. This should normally be left out as it is implied by the authorizer of the Access Token. The Asset Service MUST verify that the Access Token is authorized to transfer from this account. This could be useful for Asset providers charging their customers accounts.
-*for* URI identifying the exchanged item. 
-*redirect_uri* URI for redirecting client to afterwards
-*callback_uri* URI for performing a web callback

When a user follows this link, the Asset Service should present the user with a form authorizing the payment.

###2.1 Response


##3. Transfer

A transfer consists of a HTTP POST to the asset url by a 3rd party application on behalf of the Transferer.

The Application should have an OAuth 2.0 access token issued as defined in the OAuth 2.0 draft section 5.

    POST /usd HTTP/1.1
    Host: pay.me
    Authorization: Bearer vF9dft4qmT
   
    to=bill@example.com&amount=100.00&note=Milk

We use the following parameters from our common vocabulary in [1.6]. All fields are optional:

-*to* Account identifier of Transferee. If left out it defaults to the 3rd party applications own account on Asset Service or a predefined account as specified when authorizing the access token.
-*amount* Amount as a number with decimal points. Symbols are allowed but SHOULD be ignored. If left out it defaults to the Asset's minimum transfer, 1 or an amount predefined when authorizing the access token.
-*note* Textual description, which can include hash tags. Asset Service may truncate this. No default.
-*from* Account identifier of Transferer. This should normally be left out as it is implied by the authorizer of the Access Token. The Asset Service MUST verify that the Access Token is authorized to transfer from this account. This could be useful for Asset providers charging their customers accounts.
-*for* URI identifying the exchanged item. 
-*callback_uri* URI for performing a web callback