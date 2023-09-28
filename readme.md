

# The Simons Gift Card Gateway API Developer hub
Welcome to the gift card gateway API developer hub from **La Maison Simons**.
You'll find comprehensive guides and documentation to help you get started with our transactional API as quickly as possible, as well as offer you support encounter any issues. Let's jump right in!

# Definitions
- **Partner**: Entity that sells Simons Gift Cards through their own channels
- **VGC**: Virtual Gift Card. Can be redeemed online or in-store but does not have a physical counterpart
- **PGC**: Physical Gift Card. Can be redeemed online or in-store
- **PAN** The gift card number
- **PIN** The gift card secret (required for online purchases only)

# HOW-TO API GUIDES 

APIs are the building blocks that allow interoperability for major business platforms on the web.  The *Gift Card API* from **La Maison Simons** allow you to make gift card transactions direclty from your platform.

[How-to authenticate with mutual TLS (mTLS)](#how-to-authenticate-with-mutual-tls-mtls)
[How-to activate virtual gift card(s)](#how-to-activate-virtual-gift-cards)
[How-to activate new physical gift card](#how-to-activate-new-physical-gift-card)
[How-to reload a virtual gift card](#how-to-reload-a-virtual-gift-card)
[How-to commit a transaction](#how-to-commit-a-transaction)
[How-to reverse a transaction](#how-to-reverse-a-transaction)
[How-to get a card balance](#how-to-get-a-card-balance)
[How-to disable a card](#how-to-disable-a-card)
[Wiki](#wiki)
## How-to authenticate using mutual TLS (mTLS)

The *Gift Card API* uses mutual TLS to allow the authentication of clients and servers.

-   The authentication is the process of verifying who you are.

In order to authenticate, all you have to do is [request a certificate](#How-to-authenticate-?) and configure your HTTP client to use mTLS with the obtained certificate previously. There is no need to use custom HTTP headers or access tokens. All calls to the *Gift Card API*  must use mTLS.

As a partner, the gift card API by **La Maison Simons** lets you :

 - Activate new virtual gift card(s);
 - Activate a new physical gift card;
 - Reload a virtual gift card;
 - Rollback a transaction;
 - Retrieve a card balance;
 - Disable a card.

### What is a mutual TLS (mTLS)?
[Mutual TLS](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/) is a method for mutual authentification. It ensures that both parties at each end of network connection are who they claim to by verifying that they both have the correct private key.
### How to authenticate ?
#### 1. Generate a key pair and a CSR file
The partner (that's you !) must generate a public and private key pair and a certificate signing request (CSR). You can use any tool you want, such as [OpenSSL](https://github.com/openssl/openssl). Here is how you can generate a key pair and a CSR using OpenSSL:

    openssl req -newkey rsa:2048 -nodes -keyout privatekey.key -out request.csr

OpenSSL will ask you a few questions. Please make sure to input in the correct information based on the name and the location of your company.

:warning: We recommend that you use a 2048-bit RSA key
:warning: You may choose to protect your private key using a passphrase by remoding the -nodes argument. However, please keep in mind that this may not be supported by your HTTP client and that this is entirely optional
:heavy_exclamation_mark: **Do not share your private key** You must never share your private key with anyone, including Simons. Make sure to protect this file appropriately. Anyone with access to your private key can make transactions on your behalf.


#### 2. Send us your certificate signing request (CSR)

 - Send the CSR file (*Not the private key!*) over email to cardgateway_support@simons.ca
 - Simons will return a signed certificate. This certificate is valid for the duration of the contract only. Make sure to [renew your certificate before it expires](#Renew-your-certificate-before-it-expires).
 - A certificate is valid for a single environnement only. This means that a certificate that was signed for the UAT environnement will not be accepted by the production environnement. You may request a certificate for each environnement, but they need to be base on different private keys.

#### 3. Configure your client's web to use mTLS
Configure your web client to authenticate with mutual TLS (mTLS)  by using the certificate file you received from Simons at the previous step. Instructions are specific to your development language and framework. Refer to documentation of your HTTP client for more information.

#### 4. Renew your certificate before it expires

ToDo. This section is is construction.

## How to connect to the API

### Service URLs

|  Environnement | URL |
|--|--|
| Production | https://cardgateway.simons.ca/api/v1 |
| User Acceptance Testing |  https://cardgateway.uatsimons.ca/api/v1|

### OpenAPI spec file
ToDo

### Postman collection file

ToDo

## How-to activate virtual gift card(s)
The activate virtual gift card(s) request can be used to create new Simons virtual gift card(s) (activateVGC) based on the quantity, amount and currency of your choice.

In order to create a VGC activation transaction, you should use this URI:

    /api/v1/transaction/vgc

The HTTP verb is **POST**
The HTTP header must include:

    Accept: application/json 
    Content-Type: application/json

To form the body of the request, you will need a list of object containing the quantity of card(s), the amount per card and the currency.

Here is an example of request to activate new virtual gift cards:

    {
    "quantity": 10,
    "amount": 20,
    "currency": "CAD",
    "autoCommit": false
    }

:warning: Please note that we only support the CAD currency at the moment.

You are strongly encouraged to add additionnal information in your request. Here is how you can do it:

    {
      "quantity": 0,
      "amount": 0,
      "currency": "CAD",
      "autoCommit": false,
      "additionalInfo": {
        "merchName": "string",
        "partnerRefNumber": "string",
        "merchRefNumber": "string",
        "storeInfo": {
          "storeId": "string",
          "termId": "string",
          "storeLocation": {
            "address1": "string",
            "address2": "string",
            "city": "string",
            "state": "string",
            "zipPostalCode": "string"
          }
        }
      }
    }

ToDo: Add description of each field

### Response
If the request is successful (HTTP 200 OK), you will receive a transaction number and other informations such as:
 - The total cost
 - The quantity of cards to be activated
 - The currency
 - The transaction hash

Here is an example of  response you will get from the VGC activation call:

    {
    "noTransaction": "72cca3cf-2d97-49b2-9ad1-ef82923ccf3c",
    "createdDate": "2022-10-17T15:18:58.382+0000",
    "transactionInfo": {
    "totalCost": 200.00,
    "quantity": 10,
    "currency": "CAD",
    "txnHash": "72cca3cf-2d97-49b2-9ad1-ef82923ccf3c-200.0",
    "cards": []
    }
    }

At this point, your transaction is valid, but not yet committed. Committing is the action of making a transaction official. The next section shows how to commit a transaction.

If your response returns a bad request code (400), please refer to [this section](#wiki) to get further information on the cause of the error.



### Commit your transaction 
#### Commit with a signing key
Once you received the request response, you will need to commit your transaction. 

Most partners are required to digitally sign their transaction when committing. This requirement is in place to guarantee [non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation).
Follow the [How-to sign and commit a transaction](#how-to-commit-a-transaction-with-a-signing-key) guidelines to implement the *commit* request.
#### Auto-commit
If it's agreed with *La Maison Simons* that you can commit transaction without signing them, you can use the auto-commit feature and receive the generated card(s) information in a single call. Follow [How-to auto-commit a transaction](#how-to-auto-commit-a-transaction) guidelines to implement this functionality.

## How-to activate new physical gift card
The *activate a physical gift card (activatePGC)* request is used to activate a Simons physical gift card with a user specified amount. Denominations need to be agreed upon with Simons ahead of time.

Physical Gift Cards are pre-printed with a gift card number, but do not contain any value until they are activated.

In order to invoke the *activatePGC* feature you should use this URI:

    /api/v1/transaction/pgc/{pan}

Make sure to replace **{pan}** with the gift card number that you are trying to activate.
    
The https action must be a **POST**
The https header must include:

    Accept: application/json 
    Content-Type: application/json


The body of the request, you will need to specify the amount and the currency.

Here is an example of request to activate a new physical gift card:

    {
    "amount": 20,
    "currency": "CAD",
    "autoCommit": false
    }
You are strongly encouraged to add additionnal information in your request. Here is how you can do it:

    {
      "amount": 0,
      "currency": "CAD",
      "autoCommit": false,
      "additionalInfo": {
        "merchName": "string",
        "partnerRefNumber": "string",
        "merchRefNumber": "string",
        "storeInfo": {
          "storeId": "string",
          "termId": "string",
          "storeLocation": {
            "address1": "string",
            "address2": "string",
            "city": "string",
            "state": "string",
            "zipPostalCode": "string"
          }
        }
      }
    }


### Response 
If the  response is successful (HTTP 200 OK) you will receive the the transaction number, the date of creation and transaction informations such as:
 - The currency
 - The transaction hash
 - 
Here is an example of request response you will get from the *activateVGC* call:

       {
         "noTransaction": "4fb91445-0b34-44bd-9d7b-b2cc3b0bd238",
         "createdDate": "2022-10-17T14:51:07.623826",
         "transactionInfo": {
         "txnHash": "4fb91445-0b34-44bd-9d7b-b2cc3b0bd238-20.0",
         "currency": "CAD"
          }
       }

   If your response return a bad request code (400), please refer to [this section](#wiki) to get further information.
   
### Commit your transaction 

Once you received the request response, you will need to commit your transaction.

Most partners are required to digitally sign their transaction when committing. This requirement is in place to guarantee [non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation).
Follow the [How-to sign and commit a transaction](#how-to-commit-a-transaction-with-a-signing-key) guidelines to implement the *commit* request.
#### Auto-commit
If it's agreed with *La Maison Simons* that you can commit transaction without signing them, you can use the auto-commit feature and receive the generated card(s) information in a single call. Follow [How-to auto-commit a transaction](#how-to-auto-commit-a-transaction) guidelines to implement this functionality.


@ToDo: Show results when transaction is successfully commited

## How-to reload a virtual gift card
The *reload virtual gift card(s) (reloadVGC)* request can be used to reload a previously activated virsual gift card with a user specified amount

In order to invoke the *reloadVGC* feature you should use this URI:

     /api/v1/transaction/vgc/reload/{pan}

Make sure to replace **{pan}** with the gift card number that you are trying to reload.

The https action must be a **POST**
The https header must include:

    Accept: application/json 
    Content-Type: application/json

Use the body of the requests to specify the amount and the currency.

Here is an example of request to reload a card:

    {
    "amount": 20,
    "currency": "CAD",
    "autoCommit": false
    }
You are strongly encouraged to add additionnal information in your request. Here is how you can do it:

    {
      "amount": 0,
      "currency": "CAD",
      "autoCommit": false,
      "additionalInfo": {
        "merchName": "string",
        "partnerRefNumber": "string",
        "merchRefNumber": "string",
        "storeInfo": {
          "storeId": "string",
          "termId": "string",
          "storeLocation": {
            "address1": "string",
            "address2": "string",
            "city": "string",
            "state": "string",
            "zipPostalCode": "string"
          }
        }
      }
    }


### Response 
If the request response is successful (HTTP 200 OK), you will receive the the transaction number, the date of creation and transaction informations such as:
 - The currency
 - The transaction hash
 
Here is an example of request response you will get from the *reloadVGC* call:

       {
         "noTransaction": "b0314d8a-dc87-4ff6-9b95-3c0b221a0df3",
         "createdDate": "2022-10-17T14:59:17.013038",
         "transactionInfo": {
         "txnHash": "b0314d8a-dc87-4ff6-9b95-3c0b221a0df3-25.0",
         "currency": "CAD"
          }
       }

   If your response return a bad request code (400), please refer to [this section](#wiki) to get further information.
### Commit your transaction 

Once you received the request response, you will need to commit your transaction.

Most partners are required to digitally sign their transaction when committing. This requirement is in place to guarantee [non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation).
Follow the [How-to sign and commit a transaction](#how-to-commit-a-transaction-with-a-signing-key) guidelines to implement the *commit* request.
#### Auto-commit
If it's agreed with *La Maison Simons* that you can commit transaction without signing them, you can use the auto-commit feature and receive the generated card(s) information in a single call. Follow [How-to auto-commit a transaction](#how-to-auto-commit-a-transaction) guidelines to implement this functionality.

## How-to commit a transaction

Each transaction needs to be [digitally signed](https://en.wikipedia.org/wiki/Digital_signature) in order to be committed. This ensures that the party requesting the transaction cannot repudiate a transaction later.

Transaction information is hashed and then signed with a private key. The signature is then validated by Simons using your matching public key. You may choose to reuse the same key pair that you created when setting up mTLS, or use a different key pair for signing. Make sure that you tell us which one you want to be configured in your account. If you use a different key pair, please send us the 

Signing keys have expiration dates that are determined by the partner. When sending your public key, please tell us what expiration date you want for this key. If you ever need to replace the key, send us a new one and we'll configure it for you. Multiple signing keys can be active at the same time, with different expiration dates.

#### Transaction hashing

When you create a transaction, we return a value called "txnHash". This value is the concatenation of the transaction number and the transaction total amount.  Transactions objects are immutable.


       {
         "noTransaction": "b0314d8a-dc87-4ff6-9b95-3c0b221a0df3",
         "createdDate": "2022-10-17T14:59:17.013038",
         "transactionInfo": {
         "txnHash": "b0314d8a-dc87-4ff6-9b95-3c0b221a0df3-25.0",
         "currency": "CAD"
          }
       }


The transaction hash must be signed by your private key using the **SHA256withRSA** algorithm.


In order to commit a transaction, use this URI:

     /api/v1/transaction/{txn}/commit

Make sure to replace **{txn}** with the transaction number that you are trying to commit.

The https action must be a **POST**
The https header must include:

    Accept: application/json 
    Content-Type: application/json
    

Here is an example of request to activate a new physical gift card:

    {
    "signature": "uxYEDgzRmdd+xNMuHmvrSzsHI+hqOz/1xIIbOxO0TH7frCT8i4+dTypr4qqdni4Z88Ca0CBROM5uTiMPQgNP/zgWOhqDY8m4C93FO0eB6yIul1E6oRsCuSSzw+Dh11rsPyWiVUP6PHWgihN+FbBnuOd/vQAqJAj9HTLDJ3Bc+lk="
    }
    
If the operation is successful, you will get one of these [response body](#response-body).


### How-to auto-Commit a transaction
If applicable in your partnership with *La Maison Simons*, you will have the option to auto-commit your transaction request.
This functionnality is available for these requests:

 - [Activate virtual gift card(s) (*activateVGC*)](#how-to-activate-virtual-gift-cards)
 - [Activate a physical gift card (*activatePGC*)](#how-to-activate-new-physical-gift-card)
 - [Reload a virtual gift Card (*reloadVGC*)](#how-to-reload-a-virtual-gift-card)

**In the call Body**, set the following value:

    "autoCommit": true

Instead of getting the *activateVGC*, *activatePGC* or *reloadVGC* usual response body, you will get the following response body.

### Response Body
Example of a successful request body response for each transaction once it is commited: 
#### ***activateVGC***

    {"noTransaction": "e61e0f7b-079e-4f51-a419-e5d5ded798ba",
    "createdDate": "2022-09-14T19:18:23.732+0000",
    "transactionInfo": {
    "totalCost": 6,
    "quantity": 2,
    "currency": "CAD",
    "txnHash": "e61e0f7b-079e-4f51-a419-e5d5ded798ba-6.0",
    "cards": [
        {
        "pan": "5241458148750295",
        "pin": "0221"
        },
        {
        "pan": "5183203138980180",
        "pin": "6669"
        }
        ]
    }
    }


#### ***activatePGC***

    
    {
      "noTransaction": "e61e0f7b-079e-4f51-a419-e5d5ded798ba",
      "createdDate": "2022-10-18T14:53:34.681Z",
      "transactionInfo": {
        "txnHash": "e61e0f7b-079e-4f51-a419-e5d5ded798ba-6.0",
        "currency": "CAD"
        }
    }

#### ***reloadVGC***

    {
        "noTransaction": "2afc9076-f044-4b67-b119-4f56139b8f51",
        "createdDate": "2022-06-22T17:29:49.282+0000",
        "transactionInfo": {
        "txnHash": "2afc9076-f044-4b67-b119-4f56139b8f51-6.0",
        "currency": "CAD"
        }
    }

** If your response return a bad request code (400), please refer to [this section](#wiki) to get further information.
## How-to reverse a transaction
### Prerequisites
The endpoint *reverse* will only be successful if:

 - Your request is sent prior to the rollback maximum delay (determined when signing up your partner account)
 -  Your transaction was commited
 - Generated or updated card(s) were not redeemed between the time that the transaction was commited and the time at which you call the*reverse* endpoint
 - This request is only available for these transaction:
     -   [Activate virtual gift card(s) (*activateVGC*)](#how-to-activate-virtual-gift-cards)
     - [Activate a physical gift card (*activatePGC*)](#how-to-activate-new-physical-gift-card)
     - [Reload a virtual gift Card (*reloadVGC*)](#how-to-reload-a-virtual-gift-card)

In order to call the *reverse* endpoint you should use this URI:

    /api/v1/transaction/{txn}

Make sure to replace **{txn}** with the transaction number that you are trying to reverse.

The https action must be a **DELETE**
The https header must include:

    Accept: application/json 
    Content-Type: application/json
### Request Body
There is no body for this request.
### Response
If the request response is successful with HTTP 200 code, you will receive the the transaction number, the date of creation and transaction informations. Rollback transactions have different transaction numbers than regular transactions.

Here is an example of request response you will get from the *activateVGC* call:

    {
    "noTransaction": "a109b604-7daf-4716-90b8-a35d1a243707",
    "createdDate": "2022-08-16T14:33:54.995+0000",
    "transactionInfo": {}
    }
If your response return a bad request code (400), please refer to [this section](#wiki) to get further information.
## How-to get a card balance

In order to call the *cardBalance* feature you should use this URI:

     /api/v1/card/{pan}

Make sure to replace **{pan}** with the gift card number that you are trying to activate.

The https action must be a **POST**
The https header must include:

    Accept: application/json 
    Content-Type: application/json
### Request Body
There is no body for this request. 

### Response
If the request response is successful with a HTTP 200 OK, you will receive the card amount and the currency in the response body.

    {
    "amount": 20.00,
    "currency": "CAD"
    }

If your response return a bad request code (400), please refer to [this section](#wiki) to get further information.

## How-to disable a card
Before starting a request to disable a card, your partner account must have been configured to allow for disabling a card

In order to call the *disableCard* endpoint you should use this URI:

     /api/v1/card/{pan}/disable

Make sure to replace **{pan}** with the gift card number that you are trying to activate.

The https action must be a **POST**
The https header must include:

    Accept: application/json 
    Content-Type: application/json
    
### Request Body
There is no body for this request. 

### Response
If the request response is successful with a 200 code, there will be no body for this response.  If your response return a bad request code (400), please refer to [this section](#wiki) to get further information.

## Wiki 
### Wiki Bad Request Global
| Code | Properties                      | Description                                                     |
|------|---------------------------------|-----------------------------------------------------------------|
| 400  | INVALID_AGREEMENT_CONFIGURATION | Bad agreement   configuration. Please contact Simons support                    |
| 400  | IP_ADDRESS_REQUIRED             | Please contact Simons support                            |
| 400  | RELATE_ERROR                    | An error has occurred in our backend system. Please contact Simons support.                       |
| 400  | RELATE_UNAVAILABLE              | Our backend system is unavailable. Please try again later.                                              |
| 401  | INACTIVE_PARTNER                | Your partner account is inactive                                             |
| 403  | ACCES_DENIED                    | Your account is missing permissions to execute this action.              |
| 404  | CARD_NOT_FOUND                  | Card not found or owned by partner |
| 405  | INVALID_FORMAT                  | Input has invalid format                                        
------
### Wiki Bad Request
| Code | Properties                         | Description                                                         |
|------|------------------------------------|---------------------------------------------------------------------|
| 400  | AUTO_COMMIT_NOT_SUPPORTED            | Auto commit is not supported when a signature is required       |
| 400  | BALANCE_MAX_REACHED                    | The partner's maximum balance has been reached                         |
| 400  | CARD_ALREADY_ACTIVATED              | The card is already activated                                       |
| 400  | CARD_DISABLED                       | The card is disabled                                                |
| 400  | CARD_USED_SINCE_ACTION_TO_ROLLBACK  | A purchase has been made on this card, transaction can't be reversed             |
| 400  | INACTIVE_CARD                        | The card is inactive or not yet activated                           |
| 400  | INVALID_SIGNATURE                    | The signature is invalid or expired                                 |
| 400  | INVENTORY_LIMIT_MAX_REACHED            | The maximum number of cards has been reached for this agreement   |
| 400  | NO_VALID_AGREEMENT                  | No matching agreement is found                                      |
| 400  | SIGNATURE_REQUIRED                  | A signature is required                                           |
| 400  | TIME_LIMIT_ROLLBACK_REACHED            | The age of the transaction exceeded the time limit for a rollback |
| 400  | TRANSACTION_ALREADY_COMMITTED        | The transaction is already committed                                |
| 400  | TRANSACTION_ALREADY_REVERSED          | The transaction is already reversed                                 |
| 400  | TRANSACTION_NOT_COMMITABLE          | The transaction is not committable                                  |
| 400  | TRANSACTION_NOT_COMMITTED            | The transaction must be committed                                   |
| 400  | TRANSACTION_NOT_REVERSIBLE         | This type of transaction canot be reversed      |
| 400  | VGC_BALANCE_MAX_REACHED                | The maximum amount for a card has been reached                      |
| 404  | CURRENCY_NOT_FOUND                 | This currency is not configured in your partner account                              |
-----

