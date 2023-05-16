# Bitcoin-Native Dispute Resolution for FOSS Bounties

## 1. Problems with the Current FOSS Funding System

Free, open-source software (FOSS) development is largely funded through grants, sponsorships and donations to individuals for work on specific projects, or through ad hoc "bounties" that promise payment for a requested solution, feature, or project.

One problem with this system is that grantors and donors have complete discretion over payment.  In the case of bounties, developers who build a solution must trust the bounty grantor to pay upon presentation of the work product.  The developer lacks enforceable assurances that they will be compensated for their work.  The grantor can amend the bounty criteria on the fly.  Or the grantor can simply deny payment by taking the positon that the work product doesn't meet the previously stated criteria.  Developers have no recourse if a dispute arises.  

Another problem with the current system is that grantors can have an outsized influence on the direction of open-source development.  In Bitcoin, for example, a relatively small number of entities sponsor Bitcoin Core developers, with the top two entities sponsoring as many developers as the remaining entities combined. [<a href="#footnote-1">1</a>]  When entities sponsor an individual developer, the developer may feel pressured to work on projects that entity is interested in, instead of those the developer wishes to work on, for fear of losing a source of funding.

Finally, grantors and donors must engage in a time-consuming and sometimes challenging due diligence process to determine whether the developers have built a solution that meets the bounty or grant's criteria.  This type of review also requires high levels of computer science expertise.  These barriers to entry make it difficult for smaller, non-technical, and less capitalized funders to contribute to open-source development without first donating to a larger entity with the resources and expertise necessary for review.  Funding, therefore, tends to centralize.  

As a solution to these problems, we can transfer review of bounty applications to a bitcoin-native dispute resolution system.  A panel of experienced, independent, third-party reviewers are selected by a coordinator to adjudicate whether a bounty's criteria has been met by the applicant.  The review panel serves as an oracle to trigger release of the grant funds from a bitcoin-native, non-custodial escrow.  Grantors contemporaneously fund the escrow when posting bounties to a public bulletin board. 

By (i) removing discretion from grantors, (ii) crowdsourcing review with an independent developer panel, and (iii) automating enforcement of the panel's decision, bounty applicants gain assurances that they will be paid if their work satisfies the posted criteria.  Delegating review to a crowdsourced panel also removes barriers to entry for grantors and donors, thereby democratizing and decentralizing FOSS funding.  

## 2. Architecture of the Bounty Adjudication System

### 2.1. Overview.

The Bounty Adjudication System is built on three pillars:  

1. A review panel coordinator, 
2. A noncustodial, bitcoin-native escrow, and 
3. A public bounty board.

The bounty board and system communications can be conducted through the nostr protocol. [<a href="#footnote-2">2</a>]  Participants will be identified by a nostr public key.  Nostr allows the content of certain communications to be encrypted, while other communications (such as public bounty posts) can remain public. [<a href="#footnote-3">3</a>]

Noncustodial escrow can leverage discreet log contracts [<a href="#footnote-4">4</a>], the FediMint protocol [<a href="#footnote-5">5</a>], or miniscript hashlocks.[<a href="#footnote-6">6</a>]  Ideally, however, bitcoin transactions should be conducted over the Lightning Network to reduce the on-chain footprint and increase privacy.  The Review Panel Coordinator should not custody any grant funds to mitigate regulatory risk. 

Thus, the only centralized component will be the Review Panel Coordinator.  Accordingly, sufficient safeguards must be implemented to limit the Coordinator's substantive knowledge of the bounties being adjudicated, as well as its knowledge of the participants' identities.

But by using existing and interoperable protocols for communication and escrow, any company or person can design and run a Review Panel Coordinator that plugs into these decentralized components.  A marketplace of coordinators can develop, allowing participants to choose which providers to adjudicate their bounties.  And using a communication protocol that allows reviewers to maintain control of their data (like nostr) empowers them to choose which providers to work for.  Exposing the Review Panel Coordinator to these market forces will promote fidelity in decision-making and efficiency in pricing.

### 2.2. Bounty Adjudication Process

Bounty adjudication would progress as follows.  

1. Grantor posts a bounty to a bounty bulletin board, detailing the criteria for the project.  At the same time, Grantor funds the bounty by depositing bitcoin in escrow.

2. Developers and additional funders view the listed bounties, choosing those on which to work or contribute more funds.

3. Applicant Developer applies for the bounty by submitting the code and other supporting documentation or resources to the bounty Review Panel.  Applicant Developer pays a nonrefundable application fee that is placed into escrow.

4. The Review Panel Coordinator assigns the bounty application and Grantor's criteria to Review Panel of at least three developers, unrelated to either the Grantor or Applicant Developer.  
   
   - If a majority of the Review Panel determines that the work product meets the bounty's criteria, the funds will be released from escrow to Applicant Developer, less fees for the Review Panel members and Coordinator.
   - If a majority of the Review Panel determines the bounty's criteria have not been met, no funds will be released except the Review Panel and Coordinator fees.

5. The Review Panel transmits its decision to all participants.  If the participants disagree with the decision of the Review Panel, they can initiate an appeal within a set amount of time before funds are released from escrow.
   
   **Diagram 2.1:  Bounty Adjudication Process**
   
   ![](https://github.com/BitcoinBrief/BitResolve/blob/main/Bounty%20Adjudication%20Lifecycle.png?raw=true)

### 2.3. Review Panel Coordinator Process

The Review Panel Coordinator serves as the system's oracle.  Decisions are reached through crowdsourcing.  For each bounty application, the Coordinator selects a panel of three qualified, third-party neutral developers to serve as Reviewers.  Reviewers consider the bounty criteria and application asynchronously during a timed voting period.  In reaching their decision, the Reviewers cannot communicate with each other or the Grantor or Applicant Developer.  Each Reviewer's decision is based solely on the application material, guided by the Grantor's previously-posted criteria.   

Reviewers signal interest in participating on a panel by staking bitcoin.  After voting, if the Reviewer is in the minority, then their staked bitcoin will be forfeited and distributed to the Reviewers in the majority.  Staked bitcoin is separate from the grant escrow.  Reviewers in the majority are also rewarded with fees from the grant funds and application fee.  

Similar crowdsourced decision-making has proven successful in online marketplaces. [<a href="#footnote-7">7</a>]  The Review Panel Coordinator system employs the game theory concept of "Schelling points," similar to that used in the Kleros arbitration system on ethereum.[<a href="#footnote-8">8</a>]  Because Reviewers have a financial stake, they will seek to maximize their financial interests by voting coherently and sincerely.  

To refine the decision-making, pools of potential Reviewers can be created and stratified based on the size of the grants (for example, a pool for grants under 100,000 sats, between 100,000 sats and 1,000,000 sats, between 1,000,000 and 10,000,000, etc.).  Grant size can provide a rough approximation of the bounty's complexity.  The amount of bitcoin staked by Reviewers can be scaled based on which pool the reviewer seeks to join.  

In addition to amount of staked bitcoin, Reviewer qualifications can be considered for placement into pools.  Each Reviewer could create a nostr public key to identify them within the system.  A Reviewer can verify their qualifications by linking their public key to a GitHub account (or similar service) they control.  (Blinding could be applied to shield the contents of a Reviewer's GitHub history, while still communicating statistics useful for gauging developer experience, such as number of commits, merged pull requests, consistency/length of activity, etc.).

Once placed into the appropriate pool, and after screening Reviewers to ensure they are not related to the bounty Grantor or Applicant, Reviewers will be selected at random for a panel.

**Diagram 2.3:  Review Panel Coordinator Process**

![](https://github.com/BitcoinBrief/BitResolve/blob/main/Reviewer%20Panel%20Coordinator.png?raw=true)

## 3. Mitigating Attacks

### 3.1. Collusion Between Participants

To prevent collusion between participants, the Bounty Adjudication System ties credentials and reputation to nostr public keys.  

An Applicant Developer will sign their application with their own nostr private key linked to their Github history.  The Review Panel Coordinator checks the Applicant's nostr public key against those of the Reviewers waiting in a reviewer pool to prevent the Applicant from joining the Review Panel and voting on their own application.  Reviewers are also prevented from voting on bounties where the Applicant is significantly related the Reviewer as evidenced by their collaboration or work histories.  

Grantors also must sign bounty posts with their nostr private key.  If the Grantor is an entity, however, it will usually not have a GitHub history to check against.  The Grantor could voluntarily verify itself through a link to some other profile they control that reveals their identity.  But mandating such identity verification may undermine the Grantor's privacy preferences, as bounty posts will be publicly viewable on the Bounty Board (whereas, bounty applications are encrypted and sent directly to the Review Panel to preserve Applicant privacy and avoid disclosing the bounty solution to the Grantor prior to release of the Escrow).  Further research is required to balance Grantor privacy against the possibility of collusion.  

### 3.2. Sybil Attacks

Nostr-based reputation and credentials also guard against Sybil attacks on the system.  Because Reviewer reputation is based on verifiable experience level and history of contributions to open-source projects, attackers risk forfeiting their own reputation as developers, as well as the ability to continue earning as a Reviewer.  This creates a disincentive to attack.

Additionally, the reputation and credential features make it difficult in practice for an attacker to flood the Reviewer pools with accounts they controlled.  An attacker would need to first compromise the nostr private keys or GitHub accounts of a majority of eligible Reviewers, then have these compromised Reviewers join the pools, be selected on the relevant panels, commit votes to the targeted bounties, and wait for the expiration of an appeals deadline -- all before the exploit was uncovered.  

And requiring Reviewers to stake bitcoin for entry into pools further adds cost to any attack.  By scaling the amount of bitcoin staked with the size of the bounties under consideration, we can ensure that very large bounties do not significantly outweigh the total cost to achieve a majority.  

### 3.3. Other Abuses of the System

#### 3.3.1. Reviewer Claiming a Bounty After Voting Against

What if a Reviewer decides that a bounty is particularly valuable and wishes to claim it for themselves?  A Reviewer might vote against an application that meets the Grantor's criteria, hoping the majority denies the application.  The Reviewer, having been given access to the work product, might then take the Applicant's work and try to pass it off as their own to claim the bounty.

In this scenario, the incentives of the Schelling point game in which the Reviewers participate serves as a first line of defense.  The Reviewer would forfeit their stake by voting contrary to the merits if the majority of the Reviewers vote on the merits to approve the bounty application.  The Reviewer would lose funds and the opportunity to claim the bounty.   The nefarious Reviewer would have to believe it more likely than not that a majority of the other Reviewers are also planning to vote contrary to the merits, or that the bounty application was debatable enough that there was a significant chance the majority would vote against the application.

The credentials and reputation features provide a further safeguard where incentives fail.  Applicants are simply barred by the Review Panel Coordinator from applying for a bounty that they previously considered as a Reviewer.  (By the same token, a Reviewer cannot serve on a panel reviewing their own pending bounty application!)

#### 3.3.2. Chaotic Reviewers

For Reviewers with excessively incoherent voting records, the Review Panel Coordinator could give the Reviewer a series of test votes to determine their fidelity to the system, ultimately removing them as Reviewers if they fail.

If frivolous Reviewer voting denies an otherwise meritorious bounty application, either the Grantor or the Applicant can initiate an appeal.  The appellate panel will be comprised of different Reviewers from the initial review, with excellent voting histories, thereby decreasing the probability of further frivolously voting.

#### 3.3.3. Collusion Between Review Panel Members

Collusion between Review Panel members is largely  mitigated by restricting communication between Reviewers.  The Review Panel Coordinator software will not facilitate messages between the panel members or disclose to the panel the credentials or identity of the Grantor and Applicant.  And the Reviewers will not be informed which nostr public keys are on the panel with them.  

If any Reviewers are found to have communicated during deliberation, the coordinator will blacklist their nostr public key.  

Other violations of coordinator rules will similarly result in blacklisting of participants.

## 4. Conclusion

By outsourcing bounty review to a panel of third-party neutrals that trigger automatic payment, we can eliminate donor discretion and provide assurances to developers that they will be compensated for compliant work product.

Donors are spared the time, cost, and effort of reviewing applications.  And by removing this barrier to entry, anyone with access to the Bounty Board and a bitcoin wallet can create bounties or fund existing ones, thereby decentralizing funding sources.    

Finally, developers serving as neutral Reviewers can be compensated for review work that they may have been doing already on a volunteer basis.  

Private, bitcoin-native dispute resolution systems such as the one described here can drive efficiencies and provide assurances that better facilitate the flow of commerce in FOSS funding, and perhaps beyond.  

---

## Footnotes

<p id="footnote-1">[1] As of October, 2022, the nonprofit charity Bitcoin Brink sponsored 8 developers with over 10 commits to Bitcoin Core and Chaincode Labs sponsored 5.  The remaining 10 entities combined for 12 developers (and 7 developers were unsponsored).  See <a href="https://blog.bitmex.com/wp-content/uploads/2022/10/Bitcoin-Grant-Presentation-1.pdf">https://blog.bitmex.com/wp-content/uploads/2022/10/Bitcoin-Grant-Presentation-1.pdf</a></p>
<p id="footnote-2">[2] <a href="https://github.com/nostr-protocol/nostr">https://github.com/nostr-protocol/nostr</a></p>
<p id="footnote-3">[3] See, for example, <a href="https://nostrbounties.com/">https://nostrbounties.com/</a>, a bounty board on nostr (without an adjudication system).</p>
<p id="footnote-4">[4] <a href="https://dci.mit.edu/smart-contracts">https://dci.mit.edu/smart-contracts.</a></p>
<p id="footnote-5">[5] <a href="https://fedimint.org/">https://fedimint.org/</a></p>
<p id="footnote-6">[6] <a href="http://miniscript.com">http://miniscript.com</a></p>
<p id="footnote-7">[7] eBay India's Community Court is one example.  See Amy J. Schmitz & Colin Rule, Online Dispute Resolution for Smart Contracts, 2019 J. Disp. Resol. 103, 115 (2019).</p>
<p id="footnote-8">[8] See <a href="https://kleros.gitbook.io/docs/products/court">https://kleros.gitbook.io/docs/products/court</a></p>
