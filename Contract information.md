
This document contains confidential information about IT systems and intellectual properties of the customer, as well as information about potential vulnerabilities and methods of their exploitation. This confidential information is for internal use by the customer only and shall not be disclosed to third parties

Document


<table>
  <tr>
   <td>Name : 
   </td>
   <td>Token Contract Code Review and Security Analysis Report for eosDAC
   </td>
  </tr>
  <tr>
   <td>Date : 
   </td>
   <td>04/12/2018
   </td>
  </tr>
</table>


## Table of Contents

- [x] [Reviewer Introduction](#ReviewerIntroduction)
- [x] [Scope](#Scope)
- [x] [Executive Summary](#ExecutiveSummary)
- [x] [Contract Overview](#ContractOverview)
- [x] [Actions and Parameters](#ActionsandParameters)
- [x] [Severity Levels and explanation](#SeverityLevelsandexplanation)
- [x] [Audit Overview](#AuditOverview)
- [x] [Informational statements](#Informationalstatements)
- [x] [Conclusion](#Conclusion)
- [x] [Disclaimers](#Disclaimers)
- [x] [Technical Disclaimer](#TechnicalDisclaimer)
- [x] [Appendix A. Evidences](#AppendixA)
- [x] [Appendix B. Automated tools reports](#AppendixB)

<a name="ReviewerIntroduction"></a>
## Reviewer Introduction

*Add reviewer introduction here*

<a name="Scope"></a>
## Scope

The scope of the project is eosDAC token contract, which can be found on the link below

#### eosdactokens [https://github.com/eosdac/eosdactoken](https://github.com/eosdac/eosdactoken)

We have scanned the **eosdactoken** contract for commonly known and more specific vulnerabilities. Here are some of the commonly known vulnerabilities that are considered. \
(Audited commit `b12d92020a4d097e73eca08bce88653dd467e543`)



*   **Numerical Overflow** All numerical calculations use the eosio::asset class which are protected against potential overflow and underflow in numeric operations.
*   **Authorization Checks** - All actions in the contract require authorisation using the `require_auth()` built in checks.
*   **Apply checks** - Uses the standard `EOSIO_DISPATCH` for the apply dispatching. This has checks for the correct contract id and and ensures only declared actions are responded to by the contract. 
*   **String Parameter length** - To ensure string parameters to actions are not unbounded in length the string parameters for all the relevant actions have been capped in length to 256 characters.
*   **RAM fill up protection (RAM DoS)** - storing all items in the user's RAM space rather than the contract RAM except for actions requiring the contract's _self permission.
*   **CPU bandwidth protection (CPU DoS)** - All actions in the contract are initiated by the caller of the contract therefore the caller would need to have enough staked CPU resources to perform the called action.
*   **Source code version** - The contract code has been updated to compile with a recent version of the eosio compiler tools targeting CDT 1.4.1. This provides several optimisations in source code to reduce boilerplate code and compiled with the more optimised WABT compiler.


<a name="ExecutiveSummary"></a>
## Executive Summary

The code has been reviewed by peers and available as open source code on GitHub since before it was first deployed to the main net in June 2018. The original code was based on the eosio.token source code as written and tested by Block One. It has also been kept up to date with changes to the eosio.token contract since then. The eosio.token contract has been thoroughly tested and exercised through replication by many tokens on the network and through our own automated and manual testing on the Jungle test. We have maintained the code in that time performing incremental updates to improve performance, security and functionality. As yet we have had cause for concern about security issues and no report of any loss of value through a security vulnerability. The customised features we have added to the contract include locking the ability to transfer at the time of issue and functionality related to member registration. The features are believed to be low risk in relation to security concerns in the code since they are not involved token transfers other than preventing all transfer while a token is locked.

<a name="ContractOverview"></a>
## Contract Overview

`eosdactokens` contract is built up on eosio.token contract with additional custom parameters and functionality for EOSDAC

`eosio.token` Token Contract https://github.com/EOSIO/eosio.contracts/tree/master/eosio.token

*   Contract name – eosdactokens
*   Symbol – EOSDAC
*   Decimals – 4

<a name="ActionsandParameters"></a>
## Actions and Parameters

**`eosdactokens` contract has 10 Actions**

### **create**

Parameters:

*   issuer : is a type of eosio account_name
*   maximum_supply :  is a type of asset
*   transferred_locked : is either 1 (true) or 0 (false)

Intent: The intent of {{ create }} is to create a new token with a {{ maximum supply }} as indicated. The asset is created in the format of {{ number of tokens to 4 decimal places token symbol }} for example 1000.0000 DAC will create a DAC token with a maximum number of units of 1000.0000. If {{ transfer_locked }} is set to true then only the issuer may use the transfer action.

Term: The act of creation of this asset will expires at the conclusion of code execution. The asset will then persist on the deployed contract as long as it is active.

### **issue**

Parameters:

*   to :  is a type of eosio accountname to issue tokens to
*   quantity : is a type of eosio asset
*   memo : is a string with a maximum of 256 characters

Intent: The intent of {{ issue }} is to issue tokens {{ quantity }} and send them to the account specified {{ to }} using the transfer action. This requires a privileged account. A note {{ memo }} can be sent to the receiver.

Term: The action lasts for the duration of the processing of the contract.

### **unlock**

Parameters:

*   unlock : is a type of asset

Intent: The intent of {{ unlock }} is to unlock a token to allow transfers from accounts other than the token creator. The parameter passed in must be a standard asset in the form of "1000.0000 ABC". Although the amount component eg. 1000.000 is ignored for the logic it is required for the format of an asset parameter. This can only be done once to unlock a token and cannot be reversed to lock a token again.

Term: The act of unlocking transfer on this asset will expires at the conclusion of code execution. Transfer functionality for the asset will then persist on the deployed contract as long as it is active.

### **burn**

Parameters:

*   from : is a type of eosio account_name for the owner of the tokens to burn.
*   quantity : is a type of eosio asset

Intent: The intent of {{ burn }} is to allow a user to burn {{ quantity }} tokens that belong to them.

Term: The burn action lasts for the duration of the processing of the contract. The reduction in token supply persists on the deployed contract as long as it is active.

### **transfer**

Parameters:

*   from : is a type of eosio account_name
*   to : is a type of eosio account_name
*   quantity : is a type of eosio asset
*   memo : is a string with a maximum of 256 characters

Intent: The intent of {{ transfer }} is to allow an account {{ from }} to send {{ quantity }} tokens to another account {{ to }}. A note {{ memo }} can be sent to the receiver.

Term: The transfer action represents a change in the asset balances of the accounts involved in the transaction.

### **memberreg**

Parameters:

*   sender : is a type of eosio account_name
*   agreedterms : is a hash reference to a document contained in a string with a maximum of 256 characters

Intent: The intent of memberreg is to indicate that the account has agreed to the terms of the DAC. It will update an internal database of member accounts. This action must supply the hash of the agreement in {{ agreedterms }}, it will hold the most recently agreed to, and can be called multiple times to update the hash.

Term: This action lasts for the duration of the processing of the contract. The member registration will persist on the deployed contract as long as it is active or superseded by an updated memberreg or memberunreg action.

### **memberunreg**

Parameters:

*   sender is a type of eosio account_name

Intent: The intent of memberunreg is to allow an account {{ sender }} to unregister its membership.

Term: This action lasts for the duration of the processing of the contract. The action will persist on the deployed contract as long as it is active or superseded by an updated memberreg action.

### **updateterms**

Parameters:

*   termsid : is a number id of the terms reference stored in the contract.
*   terms : is checksum hash of the updated terms.

Intent: The intent of {{ updateterms }} is to change the URL link specifying where the terms of a pre-existing record of member terms are located, associated with the given version {{ termsid }}.

Term: The update terms action lasts until it is superseded by a subsequent action.

### **updateconfig**

Parameters:

*   notifycontr : is a contract to also be notified of all transactions in this token contract.

Intent: Notify a listening contract so that it can update it's internal state based on transactions in here.

Term: The updateconfig action lasts for the duration of the processing of the contract.

### **close**

Parameters:

*   owner : is the owner of a balance entry for the token.
*   symbol : is the symbol for the currency entry to close

Intent: Close a balance entry for a token. This allows for a 0 balance to be kept open while still holding a valid entry in the token table.

Term: The updateconfig action lasts for the duration of the processing of the contract.

### **newmemterms**

Parameters:

*   terms content for new member terms so that they can accessed on the front end clients.
*   hash a checksum hash to verify the contents that would be be agreed to on the client has not been changed.

Intent: Add an updated member terms entry after content has been changed in the member agreement.

Term: The newmemterms action lasts for the duration of the processing of the contract.

<a name="SeverityLevelsandexplanation"></a>
## Severity Levels and explanation

<table>
  <tr>
   <td><H4>Risk Level</H4>
   </td>
   <td><H4>Description</H4>
   </td>
  </tr>
  <tr>
   <td>Critical
   </td>
   <td>Critical vulnerabilities are severe as they are easy to exploit and can result in loss of tokens etc.
   </td>
  </tr>
  <tr>
   <td>High
   </td>
   <td>High level vulnerabilities are hard to exploit but can have an impact on smart contract execution, e.g. open access to an important function
   </td>
  </tr>
  <tr>
   <td>Medium
   </td>
   <td>Medium level vulnerabilities are important to fix but can't lead to loss of tokens
   </td>
  </tr>
  <tr>
   <td>Low
   </td>
   <td>Low-level vulnerabilities are mostly outdated code, unused code related. They have very low to no impact on smart contract execution
   </td>
  </tr>
  <tr>
   <td>Lowest/Code/Style/Info
   </td>
   <td>Lowest level of vulnerabilities, code style violations and info statement can't affect smart contract execution and can be ignored
   </td>
  </tr>
</table>


 <a name="AuditOverview"></a>
## Audit Overview


### **Critical**

No critical severity vulnerabilities were identified


### **High**

No high severity vulnerabilities were identified


### **Medium**

No medium severity vulnerabilities were identified


### **Low**

No Low severity vulnerabilities were identified.


### Lowest / Code style / Info

No code style issues were identified.


<a name="Informationalstatements"></a>
## Informational statements

None

<a name="Conclusion"></a>
## Conclusion

During the audit the contract was manually reviewed, the auditor found no security issues that required fixing.

<a name="Disclaimers"></a>
### Disclaimers

The audit does not give any warranties on the security of the code. A single audit may not be enough, we highly encourage several independent audits and public bug bounties to ensure optimum security of the smart contracts. 

<a name="TechnicalDisclaimer"></a>
#### Technical Disclaimer

Smart contract built on the top of EOSIO blockchain allow for a lot of features to be covered by tests, but the Turing completeness of C++ programming language and flexibility of the language leaves some space for unexpected runtime exceptions.

<a name="AppendixA"></a>
#### Appendix A. Evidences

none

<a name="AppendixB"></a>
#### Appendix B. Automated tools reports

none
