---
coding: utf-8

title: PayID Protocol
docname: draft-aanchal-payid-protocol-01
category: std

pi: [toc, sortrefs, symrefs, comments]
smart_quotes: off

area: security
author:

  -
    ins: A. Malhotra
    name: Aanchal Malhotra
    org: Ripple
    street: 315 Montgomery Street
    city: San Francisco
    region: CA
    code: 94104
    country: US
    phone: -----------------
    email: amalhotra@ripple.com
    uri: https://www.ripple.com

  -
    ins: D. Schwartz
    name: David Schwartz
    org: Ripple
    street: 315 Montgomery Street
    city: San Francisco
    region: CA
    code: 94104
    country: US
    phone: -----------------
    email: david@ripple.com
    uri: https://www.ripple.com

normative:
    RFC2119:
    RFC2818:
    RFC8446:
    RFC7258:
    RFC7231:
    RFC7413:
    RFC6265:
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
    RFC4732:

--- note_Feedback

This specification is a draft proposal, and is part of the [PayID Protocol
](https://payid.org/) work. Feedback related to this specification should
 be sent to <rfcs@payid.org>.

--- abstract
This specification defines the PayID protocol - an application-layer protocol, which can be used to interact with a PayID-enabled service provider. The primary use case is to discover payment account information along with optional metadata identified by a PayID [PayID-URI][]. The protocol is based on HTTP transfer of PayID protocol messages over a secure transport.

--- middle

# Terminology
  This protocol can be referred to as the `Basic PayID Protocol` or `PayID Protocol`. The following terminology is used by this specification.

   * Endpoint: either the client or the server of the connection.
     * Sending Endpoint: sending side of the transaction (wallet or exchange).
     * Receiving Endpoint: receiving side of the transaction (wallet or exchange).

   * PayID client: The endpoint that initiates the PayID protocol.

   * PayID owner: The owner of the PayID URI as described in [PayID-URI][].

   * PayID server: The endpoint that returns payment account information.

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119][].

# Introduction
   [PAYID-URI][] describes a URI scheme to identify payment account(s) at a service provider. [PAYID-DISCOVERY][], on the other hand, defines how to transform a PayID URI into a PayID URL that can be used by other protocols to interact with a PayID-enabled service provider but does not define the protocol(s) to do so.

   This document specifies the PayID protocol - an application-layer protocol which can be used to interact with a PayID-enabled service provider identified by a PayID URL using standard HTTP methods over a secure transport. In its most basic mode, a PayID protocol resource returns a JavaScript Object Notation (JSON) object representing the payment account(s) information along with optional metadata corresponding to the queried PayID URI [PayID-URI][]. The protocol defines new media formatting types for requests and responses, but uses normal HTTP content negotiation mechanisms for selecting alternatives that the PayID client and server may prefer in anticipation of serving different use cases.

## Design Goals

   * Extensibility

   Although the primary use case for the payment account(s) information resource returned via the Basic PayID protocol is assumed to be for making payments, the PayID protocol is designed to be easily extensible to facilitate creation and retrieval of other resources about the PayID owner, PayID client and/or PayID server that might be required for making payments.

   * Neutrality: Currency and Network Agnostic

   The PayID protocol is designed to be a fundamentally neutral protocol. The PayID protocol is capable of returning a PayID owner's payment account(s) information for any network that they (or their service) can support. This makes the PayID protocol a network and currency agnostic protocol, capable of enabling payments in BTC, XRP, ERC-20 tokens, Lightning, ILP, or even fiat networks like ACH.

   * Decentralized & Peer-to-Peer

   Just like email servers, anyone can run their own PayID server or use third-party hosted services. If self-hosted, the PayID protocol introduces no new counterparty risk or changes to a service’s security or privacy model. PayID protocol doesn’t require new, complex, and potentially unreliable peer discovery protocols, instead establishing direct peer-to-peer connections between communicating parties from the start.
   PayID is built on the most successful decentralized network: the web. There is no designated centralized authority, or a risk of a patchwork of different standards in different jurisdictions that make a global solution impossibly complex.

   * Service Sovereignty

   Each service provider that uses PayID for their users maintains full control of its PayID URL space and PayID service, and has the ability to incorporate any policies they choose, including privacy, authentication, and security.  They also have full sovereignty over users on their domain, just like with email. PayID is highly generalized and does not prescribe any particular solution outside of the standardized communication, which makes it compatible with existing compliance and user management tools and philosophies.

# PayID Server Discovery
   To support PayID protocol, the PayID client needs to discover the PayID URL corresponding to the PayID URI [PAYID-URI][]. This can be obtained either by using mechanisms described in [PAYID-DISCOVERY][] or by manually entering the PayID URL.

# JSON Format Design
  JSON, as described in [RFC8259][], defines a test format for serializing structured data. Objects are serialized as an unordered collection of name/value pairs. JSON does not define any semantics around the name/value pairs that make up an object. PayID protocol's JSON format defines name/value pairs that annotate a JSON object, property or array for PayID protocol resources.

  The PayID client MAY request a PayID response in JSON format through the `Accept` header with the media type as defined below, optionally followed by format parameters. One of the optional parameters is the case-insensitive `q` value as described in Section 5.3.1 of [RFC7231][] to indicate relative preference.

  Each message body is represented as a single JSON object. This object contains a name/value pair whose value is the correct representation for a primitive value as described in [RFC8259][], or an array or object as described in the section below.

  If the PayID server does not support JSON format, it SHOULD reply with an appropriate error response.

## HTTP Method
  The PayID protocol payment account(s) information resource is requested using the HTTP GET method.

  Following are the media types to request the payment account(s) information resource on different payment-networks and environments.

## Media Type of the Payment Account(s) Information Resource
  The media type for the payment account information resource is `application/* + json`.

## Response for application/* + json
  The response body for `application/* + json` is a JSON object with the following name/value pairs.

      {
       optional string payId,
       required string version,
       required Address[] addresses,
       optional string memo,
      }

### payId
   The value of `payId` field is the PayID URI in the client request that identifies the payment account information that the JSON object describes.

   The `payId` field is an optional field in the response.

### version
  The value of the `version` field is the PayID Protocol version that this response payload adheres to.

  The `version` field is a required field in the response.

### addresses
   The value of `addresses` field is a JSON array of type `Address` of one or more JSON objects with the following name/value pairs.

      {
       required string paymentNetwork,
       optional string environment,
       required string addressDetailsType,
       required addressDetailsType addressDetails
      }

   * paymentNetwork: The payment-network of the payment address (for example: BTC, XRPL, or ACH)
   * environment: The environment of the payment-network of the payment address (for example: MAINNET or TESTNET)
   * addressDetailsType: The value of `addressDetailsType` is one of the following strings, which indicates the object shape of `addressDetails`.
       * CryptoAddressDetails
       * FiatAddressDetails
   * addressDetails: The value of `addressDetails` is the address information necessary to send payment on a specific paymentNetwork and environment.

  The `addresses` field MUST be present in the response.

#### addressDetails
   We define the following two types of payment address types.

  * CryptoAddressDetails: This is a JSON object with the following name/value pairs.

    {
      required string address,
      optional string tag
    }

      * address: The value of `address` field contains the on-ledger address corresponding to this owner.
      * tag: The value of `tag` field is the tag value used by some cryptocurrencies to distinguish accounts contained within a singular address. E.g XRP Ledger's destination tag.

  * FiatAddressDetails: This is a JSON object with the following name/value pairs.

    {
      required string accountNumber,
      optional string routingNumber
    }

      * accountNumber: The value of `accountNumber` contains the fiat bank account number.
      * routingNumber: The value of `routingNumber` is the routing number used by some fiat payment networks.

### memo
   The `memo` string may specify additional metadata corresponding to a payment.

   The `memo` string is an OPTIONAL field in the response.

## Meaning of Media Type application/* + json
  `*` may represent different payment-networks and environments. In this document, we propose standards with the media types specific to XRP, ILP, and ACH payment networks. We also propose media types that return all addresses across all payment networks. Other payment networks MUST establish standard media types for their networks at IANA.

  * Accept: application/json

  Returns all payment account(s) information corresponding to the requested PayID URI

  * Accept: application/payid+json

  Returns all payment account(s) information corresponding to the requested PayID URI

  * Accept: application/xrpl-mainnet+json

  Returns XRPL mainnet classic addresses (and tags)

  * Accept: application/xrpl-testnet+json

  Returns XRPL testnet classic addresses (and tags)

  * Accept: application/xrpl-devnet+json

  Returns XRPL devnet classic addresses (and tags)

  * Accept: application/interledger-testnet+json

  Returns mainnet payment pointer to initiate SPSP request

  * Accept: application/interledger-devnet+json

  Returns testnet payment pointer to initiate SPSP request

  * Accept: application/ach+json

  Returns account and routing number

  The PayID client MAY specify more than one media type along with the preference parameter. The server MUST respond as described in the Content Negotiation section below.

# Header Fields
  PayID protocol defines semantics around the following request and response headers. Additional headers MAY be defined, but have no unique semantics defined in the PayID protocol.

## Common Headers {#common-headers}
  The following headers are common between PayID requests and responses.

### Header Content-Type
  PayID requests and responses with a JSON message body MUST have a `Content-Type` header value of `application-json`.

### Header Content-Length
  As defined in [RFC7230][], a request or response SHOULD include a `Content-Length` header when the message's length can be determined prior to being transferred. PayID protocol does not add any additional requirements over HTTP for writing Content-Length.

### Header PayID-Version
  The PayID client MAY include the PayID version request header field to specify the version of the PayID protocol used to generate the request.

  If present on a request, the PayID server SHOULD interpret the request according to the rules defined in the specified version of the PayID protocol or fail the request with an appropriate error response code.

  If not specified in a request, the PayID server MAY fail the request with an appropriate error code, or may return a PayID response object which includes the `version` key to indicate which version of the PayID Protocol the response payload is following.

## Request Headers
  In addition to common Headers, the PayID client MAY specify the following request header.

### Header Accept
  The PayID client MAY specify the `Accept` request header field with at least one of the registered media types (Section X). The purpose of this header is to indicate what type of content can be understood in the response. It specifies the `paymentNetwork` and `environment` of the payment account and its representation format for which the PayID client wants to receive information. The representation format is always JSON.

  The PayID server MAY reject formats that specify unknown or unsupported format parameters.

## Response Headers
  In addition to the Common Headers, the PayID server SHOULD specify the following response header.

### Header Cache-Control
  The PayID server SHOULD include the `Cache-Control` header, to indicate how the PayID server operator wants PayID responses to be cached.

###  Header Access-Control-Allow-Origin
  The PayID server MUST include a `Access-Control-Allow-Origin: *` header, allowing all origins to make cross-origin requests to resolve PayIDs.

# Extensibility

## Payload Extensibility
  PayID protocol supports extensibility in the payload, according to the specific format.

  Regardless of the format, additional content MUST NOT be present if it needs to be understood by the receiver in order to correctly interpret the payload according to the specified PayID Version. Thus, clients MUST be prepared to handle or safely ignore any content not specifically defined in the version of the payload specified by the PayID Version.

## Header Field Extensibility
  The PayID protocol defines semantics around certain HTTP request and response headers. Services that support a version of the PayID protocol conform to the processing requirements for the headers defined by this specification for that version.

  Individual services MUST NOT define custom headers, as custom headers complicate Cross-Origin Resource Sharing (CORS) requests.

## Format Extensibility
  A PayID service MUST support JSON format as described above and MAY support additional formats response bodies.

# Basic PayID Protocol
  The Basic PayID protocol is used to request a payment account(s) information resource identified by a PayID URI from a PayID-enabled service provider identified by a PayID URL using HTTP over secure transport. When successful, the PayID protocol always returns the JSON representation of a payment account(s) information resource along with optional metadata. This information can be used for any purposes outside the scope of this document, though it is expected the most common application would be making payment.

  The Basic PayID protocol comprises request and response messages, each of which is defined in more detail below. The following is a visual representation of the basic protocol flow:


    PayID client                                         PayID server
       |                                                          |
       |              1.) GET request to PayID URL                |
       |--------------------------------------------------------->|
       |                                                          | |                                                          |
       |              2.) 200 OK                                  |
       |                  Payment account information response    |
       |<---------------------------------------------------------|
       |                                                          |
       |                                                          |


## Step 1: HTTP Request to PayID URL using HTTP GET Method
  A basic PayID client issues a query using the HTTP GET method to the PayID URL without any query parameters and body.

  The PayID client MUST query the PayID server using HTTPS only. [RFC2818][] defines how HTTPS verifies the PayID server's identity. If the HTTPS connection cannot be established for any reason, then the PayID client MUST accept that the PayID request has failed and MUST NOT attempt to reissue the PayID request using HTTP over a non-secure connection.

## Step 2: Payment Account Information Response
  In response, the PayID server returns a JSON object representation of a payment account(s) information resource for the payment-network and environment requested by the PayID client in the request `Accept` header field, along with other required and/or optional metadata.

  A PayID server MUST be able to process the `application/json` and `application/payid+json` header type.

  If the PayID server does not contain the payment account information corresponding to the request, the PayID server SHOULD respond with an appropriate error message.

  A PayID server MAY redirect the PayID client; if it does, the redirection MUST only be to an `https` URI and the PayID client MUST perform certificate validation again when redirected.

## Step 3: Parse Payment Account Information Response
  If the PayID server returns a valid response, the response will contain one or more of the fields defined above.

# Example Use of Basic PayID Protocol
   This section shows sample use of Basic PayID protocol in several hypothetical scenarios.

## Basic PayID Protocol by a Wallet
   Suppose Alice wishes to send a friend some XRP from a web-based wallet provider that Alice has an account on. Alice would log-in to the wallet provider and enter Bob's PayID (say, `bob$receiver.example.com`) into the wallet UI to start the payment.
   The Wallet application would first discover the PayID URL for the PayID service-provider using one of the mechanisms described in PayID discovery [PAYID-DISCOVERY][] protocol.

   The Wallet application would then issue an HTTPS GET request:

     GET https://receiver.example.com/bob HTTP/1.1
     Accept: application/payid+json
     PayID-Version: 1.0

   The PayID server might respond like this:

     HTTP/1.1 200 OK
     Content-Type: application/payid+json
     Content-Length: 403
     PayID-Version: 1.0
     Cache-Control: no-store
     Server: Apache/1.3.11
     {
       "payId" : "bob$receiver.example.com",
       "version": "1.0",
       "addresses": [
        {
          "paymentNetwork" : "XRPL",
          "environment" : "TESTNET",
          "addressDetailsType" : "CryptoAddressDetails",
          "addressDetails" : {
            "address" : "rDk7FQvkQxQQNGTtfM2Fr66s7Nm3k87vdS"
          }
        }, {
          "paymentNetwork" : "ACH",
          "environment" : "MAINNET",
          "addressDetailsType" : "FiatAddressDetails",
          "addressDetails" : {
            "accountNumber": "000123456789",
            "routingNumber": "123456789"
          }
        }
      ],
      "memo" : "Additional optional information"
     }

  In the above example we see that the PayID server returned a list of payment accounts identified by PayID `bob$receiver.example.com`. This is because Alice's Wallet asked for all the payment accounts corresponding to the PayID in the `Accept` header.
  Alice's Wallet MAY then use the payment account information to make payments.

  Another example:

     GET https://receiver.example.com/bob HTTP/1.1
     Accept: application/xrpl-testnet+json; q=0.4,
             application/ach+json; q=0.1
     PayID-Version= 1.0

   The PayID server might respond like this:

     HTTP/1.1 200 OK
     Content-Type: application/xrpl-testnet+json
     Content-Length: 403
     PayID-version: 1.0
     Cache-Control: max-age=0
     Server: Apache/1.3.11
     {
       "payId" : "bob$receiver.example.com",
       "version": "1.0",
       "addresses" : [
         {
           "paymentNetwork" : "XRPL",
           "environment" : "TESTNET",
           "addressDetailsType" : "CryptoAddressDetails",
           "addressDetails" : {
              "address" : "rDk7FQvkQxQQNGTtfM2Fr66s7Nm3k87vdS"
            }
          }
        ]
     }


# Common Response Status Codes (TODO)
  A PayID server MAY respond to a request using any valid HTTP response code appropriate for the request. The PayID server SHOULD be as specific as possible in its choice of an HTTP specific status code.

## Success Responses
  The following response codes represent successful requests.

### Response Code 200 OK
 A request that does not create a resource returns 200 OK if it is completed successfully and the value of the resource is not null. null. In this case, the response body MUST contain the value of the resource specified in the request URL.

### Response Code 3xx Redirection
  As per [RFC7231][], a 3xx Redirection indicates that further action needs to be taken by the client in order to fulfill the request. In this case, the response SHOULD include a Location header, as appropriate, with the URL from which the result can be obtained; it MAY include a Retry-After header.

## Client Error Responses
  Error codes in the 4xx range indicate a client error, such as a malformed request.
  In the case that a response body is defined for the error code, the body of the error is as defined for the appropriate format.

# Content Negotiation

  The PayID client MAY choose to query for all possible payment addresses corresponding to a PayID URI

     GET https://receiver.example.com/bob HTTP/1.1
     Accept: application/payid+json

  In this case, the PayID server MAY respond with all payment account(s) information associated with the queried PayID.

  Alternatively, the PayID client MAY choose to query for a subset payment account(s) information in the order of preference.

     GET https://receiver.example.com/bob HTTP/1.1
     Accept: application/xrpl-testnet+json; q=0.4,
             application/xrpl-mainnet+json; q= 0.1

  In this case, the PayID server MUST respond with the payment account(s) information corresponding to at least one of the payment-networks and environments mentioned in the `Accept` header in the order of client request preference. If none of those exist, the PayID server SHOULD send an appropriate error response.

  Alternatively, the PayID client MAY combine the above two approaches.

     GET https://receiver.example.com/bob HTTP/1.1
     Accept: application/xrpl-testnet+json; q=0.4,
             application/xrpl-mainnet+json; q= 0.1,
             application/payid+json

  In this case, the PayID server MUST respond with the payment account information corresponding to at least one of the `[paymenNnetwork, environment]` tuples mentioned in the `Accept` header in the order of PayID client's preference. If none of those exist, then the PayID server MUST respond with payment account(s) information corresponding to all payment accounts associated with the queried PayID URI.

# Versioning
 Versioning enables clients and servers to evolve independently. PayID protocol defines semantics for protocol versioning.

 PayID requests and responses are versioned according to the PayID-Version header, as well as the `version` key in the PayID response.

 PayID clients include the PayID-Version header in order to specify the maximum acceptable response version. PayID servers SHOULD respond with the maximum supported version that is less than or equal to the requested `major`.

# Security Considerations
  This security considerations section only considers PayID clients and servers bound to implementations as defined in this document. Such implementations have the following characteristics:

  * PayID URIs are static and well-known to the PayID client; PayID server URLs can be static or discovered.

  The following are considered out-of-scope:

  * Communication between the PayID owner and the wallet or exchange (which acts as PayID server) for PayID URI registration, etc.
  * Communication between the sender of the transaction and the PayID client to transfer information such as PayID URI and other transaction details, etc.
  * PayID server URL discovery by the PayID client. Implementations using PayID-Discovery [PAYID-DISCOVERY][] MUST consider the security considerations in the corresponding document.
  * PayID server URL resolution by the PayID client. Implementations using DNS, DNSSEC, DoH, DoT, etc. MUST consider the security considerations of the corresponding documents.

## Network Attacks
  Basic PayID protocol's security model assumes the following network attackers:

  * Off-path attacker: An off-path attacker can be anywhere on the network. She can inject and spoof packets but can not observe, or tamper with the legitimate traffic between the PayID client and the server.

  * On-path attacker: An on-path attacker can eavesdrop, inject, spoof and replay packets, but can not drop, delay or tamper with the legitimate traffic.

  * In-path or Man-in-the-middle (MiTM) attacker: An MiTM is the most powerful network attacker. An MiTM has full access to the communication path between the PayID client and the server. She can observe, modify, delay and drop network packets.

  Additionally we assume that the attacker has enough resources to mount an attack but can not break the security guarantees provided by the cryptographic primitives of the underlying secure transport.

  The basic PayID protocol runs over HTTPS and thus relies on the security of the underlying transport. Implementations utilizing TLS 1.3 benefit from the TLS security profile defined in [RFC8446][] against all the above network attackers.

### Denial-of-Service (DoS) attacks
  As such, cryptography can not defend against DoS attacks because any attacker can stop/interrupt the PayID protocol by:
  * Dropping network packets,
  * Exhaustion of resources either at the network level or at PayID client and/or server.

  The PayID servers are recommended to follow general best network configuration practices to defend against such attacks [RFC4732][].

## Information Integrity
  The HTTPS connection provides transport security for the interaction between PayID client and server but does not provide the response integrity of the data provided by PayID server. A PayID client has no way of knowing if data provided in the payment account information resource has been manipulated at the PayID server, either due to malicious behavior on the part of the PayID server administrator or as a result of being compromised by an attacker. As with any information service available on the Internet, PayID clients should be wary of the information received from untrusted sources.

# Privacy Considerations (Best Practices)
  The PayID client and server should be aware that placing information on the Internet means that any one can access that information. While PayID protocol is an extremely useful tool to discovering payment account(s) information corresponding to a human-rememberable PayID URI, PayID owners should also understand the associated privacy risks. The easy access to payment account information via PayID protocol was a design goal of the protocol, not a limitation.

## Access Control
  PayID protocol SHOULD NOT be used to provide payment account(s) information corresponding to a PayID URI unless providing that data via PayID protocol by the relevant PayID server was explicitly authorized by the PayID owner. If a PayID owner wishes to limit access to information, PayID servers MAY provide an interface by which PayID owners can select which information is exposed through the PayID server interface. For example, PayID servers MAY allow PayID owners to mark certain data as `public` and then utilize that marking as a means of determining what information to expose via PayID protocol. The PayID servers MAY also allow PayID owners to provide a whitelist of users who are authorized to access the specific information. In such a case, the PayID server MUST authenticate the PayID client.

## Payment Address Rotation
  The power of PayID protocol comes from providing a single place where others can find payment account(s) information corresponding to a PayID URI, but PayID owners should be aware of how easily payment account information that one might publish can be used in unintended ways. As one example, one might query a PayID server only to see if a given PayID URI is valid and if so, get the list of associated payment account information. If the PayID server uses the same payment address each time, it becomes easy for a third-party to track one's entire payment history. The PayID server thus MAY follow the best practice of payment address rotation on every query for cryptocurrencies addresses to mitigate this privacy concern.

## On the Wire
  PayID protocol over HTTPS encrypts the traffic and requires mutual authentication of the PayID client and the PayID server. This mitigates both passive surveillance [RFC7258][] and the active attacks that attempt to divert PayID protocol queries to rogue servers.

  Additionally, the use of the HTTPS default port 443 and the ability
  to mix PayID protocol traffic with other HTTPS traffic on the same connection can deter unprivileged on-path devices from interfering with PayID operations and make PayID traffic analysis more difficult.

## In the PayID Server
  The Basic PayID protocol data contains no information about the PayID client; however, various transports of PayID queries and responses do provide data that can be used to correlate requests. A Basic PayID protocol implementation is built on IP, TCP, TLS and HTTP. Each layer contains one or more common features that can be used to correlate queries to the same identity.

  At the IP level, the PayID client address provides obvious correlation information. This can be mitigated by use of NAT, proxy, VPN, or simple address rotation over time. It may be aggravated  by use of a PayID server that can correlate real-time addressing information with other identifiers, such as when PayID server and other services are operated by the same entity.

  PayID client implementations that use one TCP connection for multiple PayID requests directly group those requests. Long-lived connections have better performance behaviours than short-lived connections; however they group more requests, which can expose more information to correlation and consolidation. TCP-based solutions may also seek performance through the use of TCP Fast Open [RFC7413][]. The cookies used in TCP Fast Open may allow PayID servers to correlate TLS connections together.

  TCP-based implementations often achieve better handshake performance through the use of some form of session resumption mechanism, such as Section 2.2 of [RFC8446][]. Session resumption creates a trivial mechanism for a server to correlate TLS connections together.

  HTTP's feature set can also be used for identification and tracking in a number of ways. For example, Authentication request header fields explicitly identify profiles in use and HTTP cookies are designed as an explicit state-tracking mechanism and are often used as an authentication mechanism.

  Additionally, the `User-Agent` and `Accept-Language` request header fields often convey specific information about the PayID client version or locale. This allows for content-negotiation and operational work-arounds for implementation bugs. Request header fields that control caching can expose state information about a subset of the client's history. Mixing PayID queries with other HTTP requests on the same connection also provides an opportunity for richer data correlation.

  The PayID protocol design allows implementations to fully leverage the HTTP ecosystem, including features that are not enumerated in this document. Utilizing the full set of HTTP features enables PayID to be more than HTTP tunnel, but it is at the cost of opening up implementations to the full set of privacy considerations of HTTP.

  Implementations of PayID clients and servers need to consider the benefits and privacy impacts of these features, and their deployment context, when deciding whether or not to enable them. Implementations are advised to expose the minimal set of data needed to achieve the desired feature set.

  Determining whether or not PayID client implementation requires HTTP cookie [RFC6265][] support is particularly important because HTTP cookies are the primary state tracking mechanism in HTTP, HTTP cookies SHOULD NOT be accepted by PayID clients unless they are explicitly required by a use case.

  Overall, the PayID protocol does not introduce significant additional privacy concerns beyond those associated with using the underlying IP, TCP, TLS and HTTP layers.

# IANA Considerations
  This document defines registries for PayID protocol version and application/* +json media types.

## Header Field Registration
  Header field name: PayID-Version: major.minor

  Applicable Protocol: "PayID protocol"

  Status: `standard`

  Author/Change controller: Refer to the contact details of the authors in this document.

  Specification Document: `PayID protocol`

## Media Type Registration
  This document registers multiple media types, listed in Table 1.

| Type        | Subtype                  |Specification |
|-------------|--------------------------|---------------
| application | xrpl-mainnet+json        |
| application | xrpl-testnet+json        |
| application | xrpl-devnet+json         |
| application | ach+json                 |
| application | interledger-mainnet+json |
| application | interledger-testnet+json |
| application | payid+json               |

  Type Name: application

  Subtype name: This document registers multiple subtypes as listed in table 1

  Required parameters: n/a

  Optional parameters: n/a

  Encoding considerations:  Encoding considerations are identical to those specified for the "application/json" media type. See[RFC7159][].

  Security considerations: Security considerations relating to the generation and consumption of PayID protocol messages are discussed in Section X.

  Interoperability considerations:  This document specifies the format of conforming messages and the interpretation thereof.

  Published specification: This document is the specification for these media types; see Table 1 for the section documenting each media type.

  Applications that use this media type:  PayID servers and PayID clients.

  Additional information:

   Magic number(s):  n/a

   File extension(s):  This document uses the mime type to refer to protocol messages and thus does not require a file extension.

   Macintosh file type code(s):  n/a

  Person & email address to contact for further information:  See Authors' Addresses section.

  Intended usage:  COMMON

  Restrictions on usage:  n/a

  Author:  See Authors' Addresses section.

  Change controller:  Internet Engineering Task Force (mailto:iesg@ietf.org).

# Acknowledgments
