##  Technical architecture of X Predict Market
The technical architecture of this Grant proposal mainly consists of 5 modules.

### Proposal management
The proposal is created using create，then the proposal is stored on-chain through the structure of storage Struct Proposal

Data structure of proposal
```rust
Struct Proposal {
    type:string,
    title:vec<u8>,
    optionDescription, 
    currencyOption, 
    settlement: id,
    period:u32, 
    time: u32,
    proposalDescription: vec<u8>,
}
```
Call for proposal creation
**Create**

Storage
all the proposal
**Map<id, Proposal >**
The currency required for voting is specified during the creation of proposal
**Currency ProposalId**

Users need to stake the underlying asset governance token of X Predict Market to participate in the market voting once the proposal is created.
**Doublemap <id, account, vote>**
accounts for the storage of the voting related data of the user, displaying their votes on each individual proposal.
**Map<id, vote>** 
applies to the counting of the total votes in the proposal.

When the proposal acquires enough votes to enter the formal proposal area, users can start voting on the options of the proposal.


### Transaction
Transactions can only be enabled after the proposal enters the formal area

This module is the implementation design for the user to employ the vote function
User votes by **bet (proposal ID, optionID, amount)**
If the user wants to quit the vote. **revoke(proposal ID)** can revoke the vote.

### Result(OCW)
Implementation of this module relies on the user governed OaaS （Oracle as a service.
This step applies the **Unsigned transaction with signed payloads** transaction type that does not generate mining fee.

**Doublemap(pid, account, oid)**  Result counting
**Doublemap(pid, did, vote)** Proposal votes counting,Storage of the result statistics

Trigger the report mechanism
When the result is published, anyone can trigger the report mechanism once malicious acting node is discovered.

**Vec<oid>** Staking or not
**Doublemap <account, pid, did>** 

This node will be recorded when the report succeeds
Proposal ends if no report filed

Implementation of this module relies on the user governed OaaS（Oracle as a service）, we use **Result (payload)** to submit the Unsigned transaction with signed payloads transaction, which does not generate mining fee.
When the voting ends, we use **Doublemap(pid, account, oid)** to count the result and the **Doublemap(pid, did, vote)** to count the votes of the proposal, the statistics storage of the result is accomplished by the above methods.

When the result is published, anyone can trigger the report mechanism once evil acting node is discovered.
Whether anyone file a report by staking is recorded through **Vec<oid>**
The report is evaluated by the community, the successful report result will be recorded by **map<pid, oid>**
The report cycles using Loop 
Proposal ends if no report filed

### Liquidation
This module is used to withdraw assets (for winners in the prediction)
Data strucure is recorded in the result
Exchange the proposal related governance token
**map(pid, oid)**  result
The prediction proposal winner triggers the liquidation(transaction fee required in the withdrawal)

This module is used to withdraw assets (for winners in the prediction). The formal result is published using **map(pid, oid)**. According to the settings, the wrong prediction tokens are slashed while the correct prediction token will be cashed in the settlement currency at a 1:1 ratio, and the correct prediction will be charged a 2% transaction fee at the reward withdrawal. This transaction fee is set by the creator at the beginning of the proposal creation.
The token transfer employs **transfer()**  token, balanceof(oid)


### Asset Modules for multiple asset management
**Transfer:**  Vote as transfer from to amount
In the final asset module, we use (from to amount) to manage multiple assets.