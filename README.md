# RESOLVR

# A Bitcoin-Native Dispute Resolution Service for FOSS Bounties

## 1. Problems with the Current FOSS Funding System

Free, open-source software (FOSS) development is largely funded through grants, sponsorships and donations to individuals for work on specific projects, or through ad hoc "bounties" that promise payment for a requested solution, feature, or project.

One problem with this system is that grantors and donors have complete discretion over payment.  In the case of bounties, developers who build a solution must trust the bounty grantor to pay upon presentation of the work product.  The developer lacks enforceable assurances that they will be compensated for their work.  The grantor can amend the bounty criteria on the fly.  Or the grantor can simply deny payment by taking the positon that the work product doesn't meet the previously stated criteria.  Developers have no recourse if a dispute arises.  

Another problem with the current system is that grantors can have an outsized influence on the direction of open-source development.  In Bitcoin, for example, a relatively small number of entities sponsor Bitcoin Core developers, with the top two entities sponsoring as many developers as the remaining entities combined. [<a href="#footnote-1">1</a>]  When entities sponsor an individual developer, the developer may feel pressured to work on projects that entity is interested in, instead of those the developer wishes to work on, for fear of losing a source of funding.

Finally, grantors and donors must engage in a time-consuming and sometimes challenging due diligence process to determine whether the developers have built a solution that meets the bounty or grant's criteria.  This type of review also requires high levels of computer science expertise.  These barriers to entry make it difficult for smaller, non-technical, and less capitalized funders to contribute to open-source development without first donating to a larger entity with the resources and expertise necessary for review.  Funding, therefore, tends to centralize.  

Resolvr solves these problems by transferring review of bounty applications to a bitcoin-native dispute resolution service.  A panel of experienced, independent, third-party reviewers are selected by Resolvr's coordinator to adjudicate whether a bounty's criteria has been met by the applicant.  Resolvr's review panel serves as an oracle to trigger release of the grant funds from a bitcoin-native, non-custodial escrow.  Grantors post bounties to public bulletin boards through nostr, and fund the escrow when the job is accepted by the applicant. 

By (i) removing discretion from grantors, (ii) crowdsourcing review with an independent developer panel, and (iii) automating enforcement of the panel's decision, bounty applicants gain assurances that they will be paid if their work satisfies the posted criteria.  Delegating review to a crowdsourced panel also removes barriers to entry for grantors and donors, thereby democratizing and decentralizing FOSS funding.  

## 2. Architecture of Resolvr's Bounty Adjudication Service

### 2.1. Overview.

Resolvr's bounty adjudication service is built on three pillars:  

1. A review panel coordinator, 
2. A noncustodial, bitcoin-native escrow, and 
3. A public bounty board.

The bounty board and system communications can be conducted through the nostr protocol. [<a href="#footnote-2">2</a>]  Participants will be identified by a nostr public key.  Nostr allows the content of certain communications to be encrypted, while other communications (such as public bounty posts) can remain public. [<a href="#footnote-3">3</a>]

Noncustodial escrow can leverage discreet log contracts [<a href="#footnote-4">4</a>], the FediMint protocol [<a href="#footnote-5">5</a>], or miniscript hashlocks.[<a href="#footnote-6">6</a>]  A promising on-chain escrow system called "Celebrity Escrow" [<a href="#footnote-7">7</a>] merges bitcoin keys with nostr keys, allowing contracting parties to specify a third-party escrow agent (here, the Resolvr Review Panel Coordinator) whose signiture is required to release the escrowed funds.  Whatever solution is used, the Review Panel Coordinator should not custody any grant funds to mitigate regulatory risk. 

Thus, the only centralized component will be Resolvr's Review Panel Coordinator.  Accordingly, sufficient safeguards must be implemented to limit the Coordinator's substantive knowledge of the bounties being adjudicated, as well as its knowledge of the participants' identities.

But by using existing and interoperable protocols for communication and escrow, any company or person can design and run a Review Panel Coordinator that plugs into these decentralized components.  A marketplace of coordinators can develop alongside Resolvr, allowing participants to choose which providers to adjudicate their bounties.  And using a communication protocol that allows reviewers to maintain control of their data (like nostr) empowers them to choose which providers to work for.  Exposing Resolvr's Review Panel Coordinator to these market forces will promote fidelity in decision-making and efficiency in pricing.

### 2.2. Bounty Adjudication Lifecycle

Bounty adjudication will progress as follows.  

1. Grantor posts a bounty to a bounty bulletin board on nostr, detailing the criteria for the project and specifying Resolvr as the escrow agent (and stating the number of requested Reviewers).

2. Applicant Developer applies for the bounty.  Bounty is flagged "in-progress."  
3. Applicant Developer and Grantor enter escrow contract specifying their payout addresses.  Grantor sends funds to multi-sig escrow address that requires Resolvr's key.  Applicant Developer sends nonrefundable application fee to escrow address.

4. Applicant Developer completes the work and sends code and other supporting documentation or resources to Resolvr.  The bounty is flagged as "in-review."  (NOTE:  Grantor does not receive access to code at this point.)

5. Resolvr's Review Panel Coordinator assigns the Applicant Developer's work-product and Grantor's criteria to a Review Panel of at least three developers (the exact number stated in the original bounty), unrelated to either the Grantor or Applicant Developer.  

6. Resolvr transmits the Review Panel's decision (with reasoning) to the parties to the escrow contract.  If the parties disagree with the decision of the Review Panel, they can initiate an appeal within a set amount of time before funds are released from escrow.
   
   - If a majority of the Review Panel determines that the work product meets the bounty's criteria, Resolvr signs a transaction to send the escrow funds to Applicant Developer, minus fees for the Review Panel members and Resolvr Coordinator.
   
   - If a majority of the Review Panel determines the bounty's criteria have not been met, Resolvr signs a transaction to refund Grantor from escrow, minus the Review Panel and Resolvr Coordinator fees.
   
   **Diagram 2.2:  Bounty Adjudication Lifecycle**
   
   ![](https://raw.githubusercontent.com/Resolvr-io/Resolvr/main/bolt.fun%20diagram%202.png)

### 2.3. Review Panel Coordinator Process

Resolvr's Review Panel Coordinator serves as the system's oracle.  Decisions are reached through crowdsourcing.  For each bounty application, the Coordinator selects a panel of qualified, third-party neutral developers to serve as Reviewers.  Reviewers consider the bounty criteria and application asynchronously during a timed voting period.  In reaching their decision, the Reviewers cannot communicate with each other or the Grantor or Applicant Developer.  Each Reviewer's decision is based solely on the application material, guided by the Grantor's previously-posted criteria.   

Reviewers sign up for Resolvr with their nostr public keys, which are linked to their GitHub accounts.  Based on their GitHub experience, Reviewers are sorted into pools based on level of experience (novice, intermediate, advanced) and subject matter.  (Blinding could be applied to shield the contents of a Reviewer's GitHub history, while still communicating statistics useful for gauging developer experience, such as number of commits, merged pull requests, consistency/length of activity, etc.)

To refine the decision-making, pools can be further stratified based on the size of the grants (for example, a pool for grants under 100,000 sats, between 100,000 sats and 1,000,000 sats, between 1,000,000 and 10,000,000, etc.).  Grant size can provide a rough approximation of the bounty's complexity.    

Once placed into the appropriate pool, and after screening Reviewers to ensure they are not related to the bounty Grantor or Applicant, Reviewers will be selected at random for a panel.

**Diagram 2.3:  Review Panel Coordinator Process**

![](https://raw.githubusercontent.com/Resolvr-io/Resolvr/main/Bolt.fun%20review%20diagram%202.png)

## 3. Mitigating Attacks

### 3.1. Collusion Between Participants

To prevent collusion between participants, Resolvr's Bounty Adjudication Service ties credentials and reputation to nostr public keys.  

An Applicant Developer will sign their application with their own nostr private key linked to their Github history.  The Review Panel Coordinator checks the Applicant's nostr public key against those of the Reviewers waiting in a reviewer pool to prevent the Applicant from joining the Review Panel and voting on their own application.  Reviewers are also prevented from voting on bounties where the Applicant is significantly related the Reviewer as evidenced by their collaboration or work histories.  

Grantors also must sign bounty posts with their nostr private key.  If the Grantor is an entity, however, it will usually not have a GitHub history to check against.  The Grantor could voluntarily verify itself through a link to some other profile they control that reveals their identity.  But mandating such identity verification may undermine the Grantor's privacy preferences, as bounty posts will be publicly viewable on the Bounty Board (whereas, bounty applications are encrypted and sent directly to Resolvr's Review Panel to preserve Applicant privacy and avoid disclosing the bounty solution to the Grantor prior to release of the Escrow).  Further research is required to balance Grantor privacy against the possibility of collusion.  

### 3.2. Sybil Attacks

Nostr-based reputation and credentials also guard against Sybil attacks on Resolvr.  Because Reviewer reputation is based on verifiable experience level and history of contributions to open-source projects, attackers risk forfeiting their own reputation as developers, as well as the ability to continue earning as a Reviewer.  This creates a disincentive to attack.

Additionally, the reputation and credential features make it difficult in practice for an attacker to flood the Reviewer pools with accounts they controlled.  An attacker would need to first compromise the nostr private keys or GitHub accounts of a majority of eligible Reviewers, then have these compromised Reviewers join the pools, be selected on the relevant panels, commit votes to the targeted bounties, and wait for the expiration of an appeals deadline -- all before the exploit was uncovered.  

To add further costs to any attack, Reviewers could be required to stake bitcoin for entry into pools.  By scaling the amount of bitcoin staked with the size of the bounties under consideration, we can ensure that very large bounties do not significantly outweigh the total cost to achieve a majority.  

### 3.3. Other Abuses

#### 3.3.1. Reviewer Claiming a Bounty After Voting Against

What if a Reviewer decides that a bounty is particularly valuable and wishes to claim it for themselves?  A Reviewer might vote against an application that meets the Grantor's criteria, hoping the majority denies the application.  The Reviewer, having been given access to the work product, might then take the Applicant's work and try to pass it off as their own to claim the bounty.

Resolvr's credentials and reputation features provide a safeguard here.  Applicants are simply barred by Resolvr's Review Panel Coordinator from applying for a bounty that they previously considered as a Reviewer.  (By the same token, a Reviewer cannot serve on a panel reviewing their own pending bounty application!)

#### 3.3.2. Chaotic Reviewers

For Reviewers with excessively incoherent voting records, Resolvr's Review Panel Coordinator could give the Reviewer a series of test votes to determine their fidelity to the system, ultimately removing them as Reviewers if they fail.

If frivolous Reviewer voting denies an otherwise meritorious bounty application, either the Grantor or the Applicant can initiate an appeal.  The appellate panel will be comprised of different Reviewers from the initial review, with excellent voting histories, thereby decreasing the probability of further frivolously voting.

#### 3.3.3. Collusion Between Review Panel Members

Collusion between Review Panel members is largely  mitigated by restricting communication between Reviewers.  The Review Panel Coordinator interface will not facilitate messages between the panel members or disclose to the panel the credentials or identity of the Grantor and Applicant.  And the Reviewers will not be informed which nostr public keys are on the panel with them.  

If any Reviewers are found to have communicated during deliberation, Resolvr can blacklist their nostr public key.  

Other violations of Resolvr rules will similarly result in blacklisting of participants.

## 4. Conclusion

By outsourcing bounty review to a panel of third-party neutrals that trigger automatic payment, Resolvr can eliminate donor discretion and provide assurances to developers that they will be compensated for compliant work product.

Resolvr spares donors the time, cost, and effort of reviewing applications.  By removing this barrier to entry, anyone with access to the Bounty Board through nostr and a bitcoin wallet can create bounties themselves, thereby decentralizing funding sources.    

Finally, Resolver compensates developers serving as neutral Reviewers for review work that they may be doing already on a volunteer basis.  

Resolvr can thus drive efficiencies and provide assurances that better facilitate the flow of commerce in FOSS funding, and perhaps beyond.  

---

## Footnotes

<p id="footnote-1">[1] As of October, 2022, the nonprofit charity Bitcoin Brink sponsored 8 developers with over 10 commits to Bitcoin Core and Chaincode Labs sponsored 5.  The remaining 10 entities combined for 12 developers (and 7 developers were unsponsored).  See <a href="https://blog.bitmex.com/wp-content/uploads/2022/10/Bitcoin-Grant-Presentation-1.pdf">https://blog.bitmex.com/wp-content/uploads/2022/10/Bitcoin-Grant-Presentation-1.pdf</a></p>
<p id="footnote-2">[2] <a href="https://github.com/nostr-protocol/nostr">https://github.com/nostr-protocol/nostr</a></p>
<p id="footnote-3">[3] See, for example, <a href="https://nostrbounties.com/">https://nostrbounties.com/</a>, a bounty board on nostr (without an adjudication system).</p>
<p id="footnote-4">[4] <a href="https://dci.mit.edu/smart-contracts">https://dci.mit.edu/smart-contracts.</a></p>
<p id="footnote-5">[5] <a href="https://fedimint.org/">https://fedimint.org/</a></p>
<p id="footnote-6">[6] <a href="http://miniscript.com">http://miniscript.com</a></p>
<p id="footnote-7">[7] See <a href="https://github.com/ArcadeLabsInc/celebrity-escrow">https://github.com/ArcadeLabsInc/celebrity-escrow</a> & <a href="celebrityescrow.org">celebrityescrow.org</a></p>

