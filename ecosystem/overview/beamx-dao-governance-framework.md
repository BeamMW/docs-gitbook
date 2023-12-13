# BeamX DAO: Governance Framework

## **BeamX DAO: Governance Framework** <a href="#e032" id="e032"></a>

**TL;DR**

* Proposals can modify system parameters or add entirely new functionality to the protocol.
* Governance of BeamX DAO conducts through the voting of BeamX holders. There are two types of proposals **Regular Proposals** and **Special Proposals** -- each requires a vote, separate legal quorums, and a different processing path.
* Any BeamX stakeholder can draft a governance proposal; the Facilitator holds the vote.
* The standard voting cycle (voting epoch) lasts for two weeks or 20,160 blocks.
* To vote in the current epoch, users must have staked their BEAMX tokens before it begins and cannot withdraw their tokens until the voting epoch ends.
* As of this time, the Beam governance remains unchanged and is dependent on Facilitators and the Beam Foundation representatives responsible for interfacing between BeamX DAO and the Core Objectives of the Protocol.

### BIP (BeamX Improvement Proposal)

The Facilitators and Beam Foundation representatives will collect community feedback through an open and documented process called BeamX Improvement Proposals (BIP). The goal of BIPs is to reach the broadest possible consensus on how the BeamX DAO should evolve. Additionally, BIPs provide a mechanism for community members to define critical issues and suggest changes and improvements to the system, serving as a dynamic body of Law that regulates the network at any given moment. The BIPs process will offer the highest transparency and community input while minimizing undesirable results.

BIPs are standardized documents subject to community feedback and voting that, once enacted, define the behavior of the Governance and parameters in the Beam network.

There are two types of BIPs. One of which is **Technical** (this could be a proposal to alter or make additions to BeamX platform smart contract code, for example). All other types of submissions will be categorized as **General** BIPs.

All Beam proposals must follow the BIP templates.

You can find all submitted BIPs in the Beam Governance Section on Github.

* Technical BIPs must conform to the [Technical BIP Template](https://app.gitbook.com/o/-M\_G-3uaAbZMXViHLVZl/s/-M\_xT5mSJnNyG9Y96Eca/\~/changes/5iPJb9Ji3AqEU9prZ7tX/overview/technical-bip-template).
* General BIPs must conform to the [General BIP Template](https://app.gitbook.com/o/-M\_G-3uaAbZMXViHLVZl/s/-M\_xT5mSJnNyG9Y96Eca/\~/changes/5iPJb9Ji3AqEU9prZ7tX/overview/general-bip-template).

### Regular Proposals

Voting times for Regular Proposals run every voting epoch and last two weeks (or 20,160 blocks).

Regular Proposals include distributing a positive balance from the BeamX DAO income. DAO income distributes across three categories:

* Repurchase and burn BEAMX tokens.
* Reward voters who actively participate in BeamX governance.
* Replenish the DAO reserve pools, i.e., BEAMX/BEAM or BEAMX/stable pools.

### Special Proposals

Special Proposals are only available for voting during specific voting epochs. Special Proposals may include the following:

* **Type A Proposals:** Changes for specific parameters of the DAPPs controlled by BeamX DAO, e.g., fees, treasury management, grants.
* **Type B Proposals:** improvements to the BeamX DAO and governance mechanisms.

## Voting cycle for Regular Proposals

### Submitting Regular Proposals

The Facilitators, i.e., Beam Foundation and BeamX DAO members, will submit Regular Proposals for a vote in the [forums](https://forum.beam.mw/) seven days before the next voting epoch. The team will set the initial BRR rate, and then each element can be increased by proportionally decreasing the other.

**For example**, if the initial BRR set (Burn, Reward, Rebate) were 5%, 65%, and 30%, respectively, then the vote would include four options\*:

1. _No change,_ i.e., keep existing distribution (5% burn, 65% reward, 30% rebate).
2. _Pro burn_, i.e., lower reward and rebate (10% burn, 61.6% reward, 28.4% rebate).
3. _Pro reward,_ i.e., lower rebate and burn (4.3%, 70% reward, 25.7% rebate).
4. _Pro rebate_, i.e., lower reward and burn (4.6% burn, 60.4% reward, 35% rebate).

_\*Burn and rebate funds will be accumulated until the launch of the BeamX DAO reserve pools._

### Creating a voting session for Regular Proposals

The Facilitators draft a vote on Regular Proposals one day before each scheduled voting epoch.

### Voting for Regular Proposals

Voting sessions last for one epoch. Users must have their BEAMX tokens locked in the voting contract during the previous voting epoch to be eligible to vote in the current voting epoch. One BEAMX token equals one vote, and each proposal must pass via an absolute majority, regardless of the total amount of BEAMX currently staked.

Voters (someone who has staked their BEAMX in the voting contract) can freely increase or decrease their stake. If they reduce their stake in BEAMX, their voting power reduces to the minimum stake value during the current voting epoch. If they increase their stake, their voting power will increase during the next voting epoch (provided they don't reduce their stake during that time).

To earn voter rewards, voters must vote on all proposals up for voting in the current epoch.

### Implementation of the Regular Proposal

Once the voting epoch concludes, any surplus in DAO will distribute following the vote results:

* Surplus marked for burn will go towards purchasing BEAMX using BeamX DAO reserve pools that BEAMX stakeholders can initiate at their discretion. DAO surplus marked as voter rewards for fully participating in the voting epoch (meaning they voted for every proposal) can claim their rewards (which will equal the value of their minimal stake) during the voting process.
* Surplus marked for rebating will distribute across the reserve pools of BeamX DAO proportionally to the size of those pools that BEAMX stakeholders can initiate at their discretion.

## Voting cycle for Special Proposals

### Submitting Special Proposals

All BEAMX stakeholders can post a Type A BIP on the Beam [forum](https://forum.beam.mw). Each proposal must contain the following three elements:

* Description of a problem the proposal is aiming to solve.
* A proposed solution.
* Justification or reasoning of the proposed solution.

Special Proposals Type B require the stakeholders to publish a BeamX DAO Request for Comments. After rounds of community discussion and feedback, the final BIP may undergo revisions before being held for a vote.

### Creating a vote for Special Proposals

Following Faciliatory initiated [forum](https://forum.beam.mw) discussions, voting commences seven days before the voting epoch. All rules regarding voting that apply to Regular Proposals also extend to Special Proposals. In addition, the percentage of the total BEAMX supply will dictate quorum rules. The initial quorum ranges between 5%-20% of the current BEAMX circulating supply for Type A proposals and 15%-40% of BEAMX circulating supply for Type B proposals. The Facilitator will determine the exact percentage on a case-by-case basis. In the future, quorums will be open to proposed changes following the Type B format.

### Implementation of the Special Proposal

If a quorum is reached for a Special Proposal, the Beam Foundation and BeamX DAO Facilitator will honor the will of BEAMX stakeholders and carry out the proposal.
