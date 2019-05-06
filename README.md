# Designing A Tool For Forwarding Funds To A Function

### Task

Design a smart contract structure for the following task. Whether it’s one contract, or many, we’re looking for the most secure, extensible, and scalable architecture. This task below is what you are designing for.

The gitcoin contract can be found [here](https://gitcoin.co/issue/sendwyre/Gitcoin-Open-source-Community-Tools/1/2939).

### User story

As a developer, we'd like the ability to specify a destination address and have that forward to a specific contract function.

### Current behaviour

Currently, developers are required to specify a destination address. This is great for allowing their users to convert and deposit funds into their non-custodial wallet. Currently, developers users go through multiple steps to get the deposit into their wallet, and then to select purchase, execute the transaction and then wait for confirmation on-chain. What it misses is the ability to go straight to executing the function on our smart contract.

### Wanted behaviour

We want a developer to build functionality that will allow other developers to:

1. Specify a contract (Upload source/ABI)

2. Returns known functions

3. Specify the functions that they would like accessible.

4. Deploy a contract which will execute a specific address forwarder for each function specified in step 3.

5. Returns one or more transaction id's.

6. Transactions confirm / Contracts are now setup to each individually execute a specific function.

Our purpose for this bounty is to simplify developer experiences working with our API. Ensuring that they can concentrate all their efforts on getting their project to market and/or serving their current users. We want a developer to build the functionality for Tool For Forwarding Funds To A Contract while adhering to our [feature-addition-guidelines]. Here are some more feature guidelines:

- Extensibility.

- Code Style.

- Used establish patterns where applicable.

- Generally Readable.

- Tests: Testable? Good coverage?

- Security/Defensiveness: Permission checks? Whitelisting? Exception handling?

- Performance: Correct algorithms selected? Edge cases?

### Definition of done

You have successfully completed this, when:

**Flowchart**
Can be scribbled on a whiteboard. Just needs to be legible. Use this as a check-in point for the competition. Can deliver feedback if it’s the right direction, etc…

**Schema**
Up to you to design.

**Design explanation**
Example breakdown (if helpful) Deliberately excluded & included with reasoning.

**Security concerns**
Dependencies, technical implementationrisks.

**Time to market**
How fast can it be executed on now to implement, AND for other developers in the future. Remember we’re targeting developers, and want to cater for the slowest member of the pack. Aka new developers to the space.

**Target developer experience level**
From 1-10. 1 being only use jsFiddle, 10 being Vitalik.

**Submission**
Please submit your work by making a pull request to the Github repository.

### Supporting Information

The best design will accommodate compatibility with centralized API’s. If it were a PayPal contract you’d find relevant to share account activity similar to the following:

**GET requests examples**

- Accounts: Activity changes on user “ACCO-123”
- Transfers: Updated status on the transfer “TRAN-123”
- Payment Methods: Card Expired “PAYM-123”
- Contacts: Lookup “CONT”.
