---
coding: utf-8

title: Draft 1 - PayID Easy Checkout Protocol
docname: payid-easy-checkout-protocol
category: std

pi: [toc, sortrefs, symrefs, comments]
smart_quotes: off

area: security
author:
  -
    ins: N. Kramer
    name: Noah Kramer
    org: Ripple
    street: 315 Montgomery Street
    city: San Francisco
    region: CA
    code: 94104
    country: US
    phone: -----------------
    email: nkramer@ripple.com
    uri: https://www.ripple.com
  -
    ins: D. Fuelling
    name: David Fuelling
    org: Ripple
    street: 315 Montgomery Street
    city: San Francisco
    region: CA
    code: 94104
    country: US
    phone: -----------------
    email: fuelling@ripple.com
    uri: https://www.ripple.com
  -
    ins: I. Simpson
    name: Ian Simpson
    org: Ripple
    street: 315 Montgomery Street
    city: San Francisco
    region: CA
    code: 94104
    country: US
    phone: -----------------
    email: isimpson@ripple.com
    uri: https://www.ripple.com

normative:
    RFC2119:
    RFC2818:
    RFC3986:
    RFC6265:
    RFC7231:
    RFC7413:
    RFC6570:
    RFC8446:
    PAYID-PROTOCOL:
      title: "PayID Protocol"
      author:
         ins: A. Malhotra
         fullname: Aanchal Malhotra
         ins: D. Schwartz
         fullname: David Schwartz
    PAYID-URI:
      title: "The 'payid' URI Scheme"
      target: https://tbd.example.com/
      author:
        ins: D. Fuelling
        fullname: David Fuelling
    PAYID-DISCOVERY:
      title: "PayID Discovery"
      author:
        ins: D. Fuelling
        fullname: David Fuelling

informative:
    RFC5988:

--- note_Feedback

This specification is a draft proposal, and is part of the [PayID Protocol](https://payid.org/) initiative. Feedback related to this document should be sent in the form of a Github issue at: https://github.com/payid-org/rfcs/issues.
 
--- abstract
This specification formalizes how a payment recipient, such as a merchant or a non-profit, can automatically 
initiate a payment from a payer using only the payer's PayID. 

--- middle

# Terminology

This protocol can be referred to as the `PayID Easy Checkout Protocol`. It uses the following terminology:
   
* PayID Easy Checkout Client: A client that queries a PayID Discovery Server using the PayID Discovery Protocol as defined in [PAYID-DISCOVERY][] and that assembles a PayID Easy Checkout URL.
* PayID Discovery Server: the endpoint that returns a PayID Discovery JRD conforming to the PayID Discovery Protocol as defined in [PAYID-DISCOVERY][].
* Recipient: Individual or entity receiving a payment (e.g., e-commerce merchant, charity).
* Payer: Individual or entity originating a payment to a `recipient`.
* Wallet: A device or application that holds funds (may be a non-custodial wallet).
* PayID Easy Checkout URL: The URL that is the result of the PayID Easy Checkout protocol; can be used to redirect a client to a wallet corresponding to a particular PayID as defined in [PAYID-URI][].

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119][] and [RFC9174][].

# Introduction

The PayID Easy Checkout Protocol is a minimal protocol that allows a recipient (e.g., an online merchant or a charity) to
request a payment from a payer using only the payer's PayID as defined in [PAYID-URI][]. Implementations
of the protocol require little to no server-side engineering efforts, while creating a seamless and uniform
user experience for payers.

The main focus of the Protocol is on PayID Easy Checkout Discovery, which defines how a PayID Easy Checkout Client can 
use a PayID to retrieve a PayID Easy Checkout URL which represents a resource that the payer's digital wallet can use to initiate 
a payment to the merchant. 

Though the [appendix](#appendix) of this specification provides an example usage of a 
PayID Easy Checkout URL using Web Redirects, supplemental RFCs are needed to define the different ways in which a PayID
client can utilize a PayID Easy Checkout URL.

# PayID Easy Checkout Protocol
The PayID Easy Checkout Protocol can be used to facilitate an end-to-end checkout flow between a payment recipient, such
as an online merchant, and a payer via a sending client, such as a wallet.

The protocol is comprised of two parts:

1. PayID Easy Checkout Discovery
2. PayID Easy Checkout URL Assembly

## PayID Easy Checkout Discovery
PayID Easy Checkout Discovery extends [PAYID-DISCOVERY][] by defining a new link in the PayID metadata JRD
returned by a PayID Discovery query. This link, defined in the [JRD section](#payid-easy-checkout-jrds) 
of this specification, includes the PayID Easy Checkout URL representing a resource on the wallet which can
be used to complete a payment.

Recipients who wish to initiate an Easy Checkout flow MUST query the sender's PayID Discovery Server to 
obtain a PayID Easy Checkout URL. PayID Discovery Servers that wish to enable PayID Easy
Checkout MUST include a JRD Link conforming to the definition in the [JRD section](#payid-easy-checkout-jrds) of this paper 
in all PayID Easy Checkout Discovery responses.

Recipients SHOULD implement fallback measures to complete a checkout flow if a payer's wallet does not support PayID Easy Checkout.

The following steps describe how a PayID Easy Checkout Client can query a PayID Discovery Server to obtain a PayID Easy Checkout URL.

### Step 1: Assemble PayID Easy Checkout Discovery URL
The process of assembling a PayID Discovery URL is defined in section 4.1.1 of [PAYID-DISCOVERY][], and is
the same as for this protocol.

### Step 2: Query PayID Easy Checkout Discovery URL
Querying the PayID Discovery URL is defined in section 4.1.2 of [PAYID-DISCOVERY][], and is performed
in the same way as this protocol.

Clients SHOULD implement fallback measures to complete checkout if the PayID Easy Checkout Discovery query fails.

### Step 3: Parse PayID Easy Checkout Metadata
If PayID Easy Checkout is supported, a PayID Discovery server MUST respond with an HTTP status code `200` and a JSON payload
containing a JSON Resource Descriptor (JRD) that contains a link conforming to the [JRD section](#payid-easy-checkout-jrds) of this document.

For example, a PayID Discovery Server might respond to a PayID Discovery query with the following payload:

     {
        "subject": "payid:alice$wallet.com",
        "links": [
            {
                "rel" : "https://payid.org/ns/payid-easy-checkout-uri/1.0",
                "href": "https://wallet.com/checkout"
            }
        ]
     }
     
A PayID Easy Checkout client MUST parse this response to find the PayID Easy Checkout Link. 
If the JRD returned from the PayID Discovery query does not contain a 
PayID Easy Checkout Link in its 'links' collection, PayID Easy Checkout is considered to have failed.
Once a PayID Easy Checkout URL has been obtained from the PayID Easy Checkout Link, 
PayID Easy Checkout Discovery is considered to be complete. 

## PayID Easy Checkout URL Assembly
A PayID Easy Checkout URL represents the resource on a wallet that can
be used by a payer to complete a payment. However, before directing a payer to their wallet, the recipient
MUST append all of the query parameters defined in the [following section](#payid-easy-checkout-url-query-parameters).

Once a PayID Easy Checkout URL is assembled, PayID Easy Checkout is considered to be complete.

### PayID Easy Checkout URL Query Parameters
This specification defines several query parameter names and corresponding datatypes which MUST be added to the
PayID Easy Checkout URL before redirecting a payer to their wallet. The PayID Easy Checkout URL SHOULD be parsed 
by the wallet in order to retrieve any values set by the recipient. It is RECOMMENDED that wallets use these 
values to pre-populate a payment transaction.
    
| Name           | Type             | Description                                                          |
|----------------|------------------|----------------------------------------------------------------------|
| amount         | integer          | The amount that should be sent by the payer to the recipient         |
| receiverPayId  | string           | The [PAYID-URI][] of the receiver                           |
| assetCode      | string           | The currency code that denominates the amount as defined in [PAYID-PROTOCOL][] |
| assetScale     | short            | Defines how many units make up one regular unit of the assetCode     |
| paymentNetwork | string           | The payment network, as defined in [PAYID-PROTOCOL][], that the sender should use to send a payment. |
| nextUrl        | HTTP Url string  | A URL that the sender's wallet can navigate a sender to after the sender completes a payment  |
|----------------|------------------|----------------------------------------------------------------------|
    
When adding values into a URI 'query' part as defined by
[RFC3986][], values with characters outside the character set allowed by query parameters in [RFC3986][]
MUST be percent or otherwise encoded.

Protocols MAY define additional query parameter names and syntax rules, but MUST NOT
change the meaning of the variables specified in this document.

For example:

    Input:    alice$wallet.com
              amount=10
              receiverPayId=pay$merchant.com
              assetCode=XRP
              assetScale=6
              network=XRPL
              nextUrl=https://merchant.com/thankyou
    PayID Easy Checkout URL: https://wallet.com/checkout
    Output:   https://wallet.com/checkout?amount=100000&receiverPayId=payid%2Apay%24merchant.com&assetCode=XRP&assetScale=6&paymentNetwork=XRPL&nextUrl=https://merchant.com/thankyou

# PayID Easy Checkout JRDs
This section defines the PayID Easy Checkout Link, which conforms to section 4.4 of the
Webfinger RFC.  In order for a PayID Discovery Server to enable PayID Easy Checkout, a PayID Discovery query to the server
MUST return a JRD containing a PayID Easy Checkout Link.

The Link MUST include the Link Relation Type defined in [PayID Easy Checkout URL](#iana-considerations) in the object's 'rel' field.
The Link MUST also include a PayID Easy Checkout URL in the 'href' field of the link.

    * 'rel': `https://payid.org/ns/payid-easy-checkout-uri/1.0`
    * 'href': A PayID Easy Checkout URL

The following is an example of a PayID Easy Checkout Link:

    {
        "rel": "https://payid.org/ns/payid-easy-checkout-uri/1.0",
        "href": "https://wallet.com/checkout"
    }

# Security Considerations
Various security considerations should be taken into account for PayID
Easy Checkout.

The security considerations for PayID Easy Checkout Discovery are discussed in 
section 6 of [PAYID-DISCOVERY][].

## PayID Easy Checkout Redirection URI Manipulation
When a payer uses the resource located at the PayID Easy Checkout URL, a hijacker could manipulate
the data encoded in the URL to trick the sender into sending a payment to a different PayID than was originally
requested, or manipulate other points of PayID Easy Checkout data to trick the sender. 

Additionally, if a hijacker gained access to the merchant client, they could replace the PayID Easy Checkout URL
for the purposes of a phishing attack.

Current work on the PayID Protocol and its extensions may prove useful in mitigating these risks. 

## Access Control
As with all web resources, access to the PayID Discovery resource could
require authentication. See section 6 of [RFC7033][] for Access Control
considerations.

Furthermore, it is RECOMMENDED that PayID Discovery Servers only expose PayID Easy Checkout URLs
which resolve to a protected resource.  

# IANA Considerations
  ## New Link Relation Types
  This document defines the following Link relation type per [RFC7033][].
  See section 3 for examples of each type of Link.
  
  ### PayID Easy Checkout URL
  
    * Relation Type ('rel'): `https://payid.org/ns/payid-easy-checkout-uri/1.0`
    * Media Type: `application/jrd+json`
    * Description: PayID Easy Checkout URL, version 1.0

# Acknowledgments

# Appendix

## Motivation
The PayID Easy Checkout Protocol aims to enable a consistent user experience for payers paying for goods
or services by standardizing the interaction between merchants/non-profits and customer/donor wallets.
Given the ability to assign arbitrary metadata to a PayID as defined in [PayID-Discovery][], there is an opportunity
to standardize the set of interactions between merchant and payer, specifically the process by which a merchant
directs a payer to their digital wallet to complete a payment.
We believe this protocol will enable an improved paying experience by reducing the number
of steps a payer must take to complete a transaction.

PayID Easy Checkout also limits the engineering effort needed to implement the protocol. 
Clients wishing to adopt this pattern should only need to implement UI-level changes in order to make the flow function 
as intended, which may aid in expanding overall adoption, further enhancing the protocol's user experience benefits. 

### Design Goals

#### Minimal effort for the Payer

In order for a payer to checkout using the PayID Easy Checkout protocol, the payer only needs to provide a merchant
with their PayID Easy Checkout enabled PayID.

#### No New Server-Side Software

Apart from a PayID Discovery compliant PayID Discovery Server, The PayID Easy Checkout Protocol does not require server-side 
software to be run by either the payer or merchant for a payment. The PayID Discovery Server is capable of providing details 
of where to send the payer via the PayID Discovery Protocol. Assuming the wallet used by the payer has implemented 
support in their UI for the PayID Easy Checkout Protocol, the payer can be redirected to their wallet 
to complete their transaction.

## Example Usage
This section shows a non-normative example of PayID Easy Checkout between a hypothetical merchant (recipient) and payer. The merchant
accepts payments using the PayID pay$merchant.com, and the payer controls the PayID alice$wallet.com.

### PayID Easy Checkout Initiation
In this example, the payer might place some items in an online shopping cart on the merchant's web-site, then choose
to checkout.  The merchant would then render a form asking for the payer's PayID, as well as a "Checkout with PayID"
button.  Once the payer inputs their PayID `alice$wallet.com` and clicks the "Checkout with PayID" button, the merchant
begins the PayID Easy Checkout flow.

### PayID Easy Checkout Wallet Discovery
The merchant UI would first assemble the PayID Easy Checkout URL as defined in [PayID Easy Checkout Discovery](#payid-easy-checkout-discovery),
yielding the URL `https://wallet.com/.well-known/webfinger?resource=payid%3Aalice%24wallet.com`. 
The merchant UI would then [query the assembled URL](#step-2-query-payid-easy-checkout-discovery-url).

The HTTP request in this example would look like this:
    
    GET /.well-known/webfinger?resource=payid%3Aalice%24wallet.com
    Host: wallet.com
    
If the payer's PayID Discovery Server has enabled PayID Easy Checkout in their wallet, the server would respond with something like this:
     
     HTTP/1.1 200 OK
     Access-Control-Allow-Origin: *
     Content-Type: application/jrd+json

     {
       "subject" : "payid:alice$wallet.com",
       "links" :
       [
         {  
           "rel": "https://payid.org/ns/payid-easy-checkout-uri/1.0",
           "template": "https://wallet.com/checkout"
         }
       ]
     }

### Assemble PayID Easy Checkout URL with Query Parameters
The merchant UI would parse the PayID Discovery response and iterate over the 'links' collection to find the link with 
the Relation Type of "https://payid.org/ns/payid-easy-checkout-uri/1.0". The merchant UI would then add all of the query
parameters defined in [PayID Easy Checkout URL Query Parameters](#payid-easy-checkout-url-query-parameters) to the URL included in the JRD Link. 
One query parameter of note is the "nextUrl" parameter, which allows the merchant to supply a redirect or callback URL 
for the sender's wallet to call once the payer has confirmed the payment. In this example, the merchant would like 
to display a "Thank You" page, and replaces `{nextUrl}` with `https://merchant.com/thankyou`.

#### Correlating a Payment to an Invoice
Merchants and non-profits will often times need to correlate discrete layer 1 payments to invoice or transaction entities
in the merchants' native systems. The merchant in this example may have an invoice tracking system, on which an invoice
gets created for the goods that the payer is buying with a unique identifier of `1045464`. A common practice for correlating
layer 1 payments to a specific transaction or invoice is to accept payments on a different layer 1 address for each invoice
so that the merchant can listen for payments into that address and tie the payment to the invoice.  However, because
the PayID Easy Checkout URL only provides the receiver's PayID, there is currently no way to associate the address that
is given to the payer to the invoice.

In order to accomplish this, a merchant could provide a unique PayID containing the invoice identifier 
for each PayID Easy Checkout transaction. In this example, the merchant would first associate a payment address with the
invoice ID, and would then redirect the payer to their wallet with the `receiverPayId` query parameter set to `pay-1045464$merchant.com`.
When the merchant PayID Discovery Server receives a query for the address associated with that PayID, they could return the previously
stored payment address. When the merchant receives a payment to that address, they can then associate the layer 1 payment
with the invoice.

### Redirect Payer to Their Wallet
Once the merchant UI populates the required query parameters in the URL template, the merchant UI redirects the payer to 
the Redirect URL so that the payer can confirm the payment.

### Payer Confirms Payment
After the payer clicks the "Pay with PayID" button the merchant's UI, and the merchant performs the previous steps,
the payer will be redirected to the Redirect URL, which is a front end resource of the wallet. The wallet UI can
read the query parameters from the Redirect URL and render a confirmation page or modal with all of the required fields
pre-populated.

Once the payer confirms the payment, the wallet would perform a PayID address lookup on the "receiverPayId" query
parameter to get the payment address of the merchant and submit a transaction to the underlying ledger or payment system.
The merchant can then redirect the user back to the URL specified in the "nextUrl" query parameter, which will display
the "Thank You" page of the merchant.