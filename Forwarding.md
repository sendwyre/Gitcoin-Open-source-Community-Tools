# Wyre Forwarding Tool #
## About this document ##
**Summary**: This document describes a tool that developers can use to further integrate Wyre to their blockchain-powered applications.

Date | Version | Author | Comments
-----|---------|--------|---------
2019-05-15 | 1.0.0 | [Emilio Silva Schlenker](mailto:e18r@disroot.org) | Initial version
## Definitions ##
**Dapp**: Partner's application used by the end user by means of Wyre's API.<br/>
**End user**: User of the partner's dapp.<br/>
**Ethereum blockchain**: The blockchain in which is deployed the original contract of the partner that Wyre interacts with.<br/>
**Forwarding tool**: Dapp developed by Wyre that lets partners deploy forwarding contracts for their original contract's functions.<br/>
**Forwarding contract**: A smart contract whose purpose is to act as a proxy between Wyre's API and a specific function in the original contract.<br/>
**Original contract**: The smart contract created by the partner and deployed on the Ethereum blockchain, whose functions the partner wants the end user to be able to interact with.<br/>
**Partner**: Developer who integrates Wyre's API in her dapp.<br/>
**Wyre's API**: A set of programmatic services that the partner integrates in her dapp.
## Problem Statement ##
![Current end user workflow](https://www.lucidchart.com/publicSegments/view/97106219-6f65-473d-8c37-1a53b4244dc7/image.png)

*Current end user workflow*

Wyre's API allows dapp developers to enlarge their user base by onboarding users who don't own cryptocurrencies. On a Wyre-enabled dapp, the end user is able to use her bank account to send funds to the dapp's smart contract. At present, however, the funds can only be sent to the smart contract's [fallback function](https://solidity.readthedocs.io/en/latest/contracts.html#fallback-function). In order to access all the features of many dapps, end users often need the ability to interact with other functions of each dapp's smart contract. Although Wyre allows end users to fund their own Ethereum accounts, many end users are discouraged from using dapps by the difficulty of learning to use Metamask or other dapp browsers. In this context, the ability to interact with a dapp only by means of a bank account can be very valuable.
![Current partner's worflow](https://www.lucidchart.com/publicSegments/view/090d78da-4210-4f15-a956-246dd43e474f/image.png)

*Current partner's workflow*

The partner's workflow in order to develop a Wyre-powered dapp consists in developing the dapp front end, creating and deploying a set of smart contracts, optionally a back end, and finally configuring Wyre's widget to include Wyre's API in its dapp, or including Wyre's API directly.
## Proposed Solution ##
![Proposed end user workflow](https://www.lucidchart.com/publicSegments/view/74eb5039-16e4-47a4-ac7c-fcf59157e85a/image.png)

*Proposed end user workflow*

With the help of an additional tool provided by Wyre, Wyre's partners can create forwarding contracts that allow Wyre to send end user funds to any function of the partner's original contract. To the end user, the experience remains unchanged, while the partner needs to perform an aditional configuration step. From Wyre's point of view, the payment flow will also remain unchanged, except for [transaction fees](#change-in-wyres-api).
![Proposed partner's workflow](https://www.lucidchart.com/publicSegments/view/bada6e7d-e12b-4d24-bf2d-faccf73d43f1/image.png)

*Proposed partner's workflow*

These forwarding contracts are created in such a way, that they are able to receive Wyre's payments through their fallback function, and then call a specific function in the original contract. Wyre's partners can deploy as many forwarding contracts as they need, at a rate of one per function, and need to configure them by specifying the original contract's deployment address on the Ethereum blockchain, as well as the name of the function to forward to, the amount of gas it needs in order to be executed successfully, and the function's arguments. These arguments [cannot be changed](#shortcomings) by the end user.

Even though the Forwarding Tool can be developed as a traditional application, we propose for it to be a Dapp. This grants it two important features: First, by using a dapp, the partner is in charge of the deployment of the forwarding contracts on the ethereum platform, which grants her a more granular control and, importantly, the ability to verify the integrity of each forwarding contract before and after deployment. Second, since Wyre would not be the entity in charge of the deployment, this protects it from potential legal issues resulting from the misuse of the forwarding tool.
### Shortcomings ###
Although this proposed solution addresses the problem of sending funds to specific contract functions, it doesn't address the more general problem of interacting with smart contracts. Indeed, dapps have broader uses beyond purely financial ones, and their functionality doesn't revolve solely around financial transactions. Besides from the amount sent, end users often need to be able to send specific data to contract functions. However, forwarding contracts don't allow end users to specify that data, which needs to be hard-coded by the partner at deployment time.

Another disadvantage of this approach is an increase in transaction fees that can become noticeable during network congestion peaks.
## Implementation ##
### Change in Wyre's API ###
Wyre's API needs to change the way it handles transaction fees on Ethereum.

There are [two types of accounts](http://www.ethdocs.org/en/latest/contracts-and-transactions/account-types-gas-and-transactions.html) in Ethereum: Externally Owned Accounts (EOA), i.e. regular user accounts, and Contract accounts. The fees for sending funds to EOAs are solely dependent on the fee market, as in Bitcoin. The fees for sending funds to Contract accounts, however, depend not only on the fee market, but also on the contracts themselves. Given that smart contracts behave like programs, the more complex they get, the more costly it is to execute them and the more fees they require. That's the reason why Ethereum fees are calculated as the product of the gas price (driven by the market) and the gas amount (dependent on the contract's complexity). EOAs have a fixed gas amount corresponding to 21,000 gas units, while contracts have usually higher gas amount requirements. Each contract function has a specific amount requirement, and it can't be easily estimated.

For these reasons, Wyre's API needs to allow partners to specify the gas amount that their contract functions need. Failure to do so will likely result in failed transactions. Given that it would be difficult for Wyre to estimate the amount of gas a certain contract needs, the best option is to leave that responsibility to Wyre's partners by adding a field in Wyre's transfer API. This field would allow Wyre's partners to specify the gas amount. The gas price can be obtained in the same way it is currently being obtained by Wyre.
### Forwarding Contract ###
A forwarding contract with a constructor and a [fallback function](https://solidity.readthedocs.io/en/latest/contracts.html#fallback-function) needs to be created. The constructor receives as parameters the original contract's deployment address on the Ethereum blockchain, the name of the original contract's function to forward to and a list of arguments. With these data, the fallback function uses an [external function call](https://solidity.readthedocs.io/en/latest/control-structures.html#external-function-calls) or a [low-level `CALL`](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#members-of-address-types) to invoke the original contract's specified function and send all the received ether to it, as well as the remaining gas and the specified arguments.

The resulting code should follow [security best practices](https://consensys.github.io/smart-contract-best-practices/). Also, by using the Truffle Suite or a similar smart contract development framework, this forwarding contract needs to be thoroughly tested in different testnets. The unit tests should be designed with a security mindset, considering [possible attacks](https://consensys.github.io/smart-contract-best-practices/known_attacks/).
### Dapp ###
A dapp that deploys forwarding contracts to different ethereum blockchains needs to be created. It should have a user interface that targets Wyre's partners and has an emphasis on usability. It should use the web3.js library to [deploy](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#deploy) the forwarding contracts, whose generated JSON file needs to be made available by the smart contract developer. The dapp provides a form in which the partner fills the following data:
* The original contract's deployment address on the ethereum blockchain.
* The name of the original contract's function to forward to.
* An arbitrarily-sized list of arguments to be passed to the above function.

These data need to be passed to the `deploy` function as arguments. As soon as the new contract is deployed, the dapp should [check](https://web3js.readthedocs.io/en/1.0/web3-eth.html#eth-chainid) on which blockchain it is currently operating, and if it's a blockchain supported by Etherscan, [publish](https://etherscan.io/apis#contracts) the source code of the contract. For this, the dapp needs access to a flattened version of the latest forwarding contract's source code.

The dapp might incorporate the look and feel of Wyre, for which it would be advisable to follow Wyre's style guidelines and/or obtain Wyre's stylesheets.
### Feature Addition Guidelines ###
The development needs to follow Wyre's Feature Addition Guidelines:
* **Extensibility**.
* **Code Style**.
* **Used establish patterns where applicable**.
* **Generally Readable**.
* **Tests**: Testable? Good coverage?
* **Security/Defensiveness**: Permission checks? Whitelisting? Exception handling?
* **Performance**: Correct algorithms selected? Edge cases?
### Deployment ###
The dapp can be hosted on any server under an accessible URL. It can be a JavaScript bundle created with Webpack and served as static content. It doesn't require a back end, and it also doesn't require to interact with Wyre's API in any way.
## Human Requirements ##
### Solidity developer ###
**Task**: create the solidity code of the forwarding contract, as well as the unit tests. Write the high-level documentation of the contract.<br/>
**Experience level**: 6/10<br/>
**Estimated time**: 35 hrs
### Front End developer ###
**Task**: create the dapp front-end using Wyre's look & feel and the web3.js library, as well as consume Etherscan's API for publishing contract sources. Use the web3.js library to deploy contracts on the ethereum blockchain.<br/>
**Experience level**: 7/10<br/>
**Estimated time**: 45 hrs
### Back End developer ###
**Task**: modify Wyre's transfer API in order to let partners customize the gas amount of each transfer when performing Ethereum transactions. Create the relevant unit tests.<br/>
**Experience level**: 5/10<br/>
**Estimated time**: 15 hrs
## Security Considerations ##
### [Reentrancy](https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy) ###
Reentrancy is an attack by which a contract can be completely depleted of its funds by means of a vulnerable paying external function call. If the contract's balance is updated *after* making the call, it is possible for an attacker to forge a smart contract that re-enters its calling contract and executes the paying function multiple times. Although worth mentioning, this attack is, however, not a problem for the current case, because:
* Both the caller and the callee are controlled by the same entity.
* The caller intends to spend all its funds on the call anyway.
### Malicious Forward Call ###
At some point between the moment in which the partner specifies her original contract's deployment address, and the moment in which the forwarding contract gets deployed, a modification to the address could occur, which would result in the diversion of end user funds to an attacker's wallet. From the point of view of Wyre's partner, this is probably the biggest concern by using this tool. This is mitigated by:
* Publishing the source code of the contract as soon as it gets deployed on any blockchain.
* Being a dapp, which allows partners to check the deployment code, which gets executed on their local machines with their own dapp browsers.
### Vulnerabilities in the forwarding contract ###
An unknown vulnerability could exist in the forwarding contract, which would allow an attacker to prevent it from working correctly, which could result in failed or delayed transactions, the locking of funds in the forwarding contract, or the diversion of funds to the attacker's wallet. This is mitigated by:
* Performing an audit by a well-known smart contract security firm such as [CoinMercenary](https://www.coinmercenary.com/) before making the dapp available to partners.
* Creating an ongoing bug bounty program in a platform such as [Gitcoin](https://gitcoin.co) or [HackerOne](https://www.hackerone.com/). This is very important since the Ethereum ecosystem evolves rapidly and new vulnerabilities appear every day.
* Having a legal disclaimer on the dapp, exempting Wyre from any responsibility related to the loss of funds or other damage to partners or end users. This, however, might deter some partners or end users from using the tool.
## Final Remarks ##
Wyre's forwarding tool goes one step in the direction of allowing end users outside of the crypto community to access Ethereum dapps. A further step would require a change in Wyre's API to allow for built-in execution of contract functions. This would work in two steps: first, partners would upload the contract source to Wyre's API, and then they would be able to refer to specific functions to be called by the transfers API. If this tool proves successful, I believe it would be the next step for Wyre.
