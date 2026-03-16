# VAMP — Verified Authenticated Messaging Protocol

> [!TIP]
> This project is current in the early stages of ideation and prototyping. The design is not finalized, the implementation is not complete, and the roadmap is not set in stone. The goal of this repository is to explore the problem space and potential solutions in the open, with a reference implementation to validate the model. 
> Feedback, contributions, and discussion are very welcome.

VAMP is an open protocol for Identity-First Mail: a cryptographically verifiable, spam-resistant successor to legacy email.

This project explores Identity-First Mail (IFM), a messaging model in which identity is cryptographic, capability is discoverable before send, native traffic is encrypted by default, and suspicious scale carries **[sender-side cost](docs/sender-cost-mechanics.md)**. VAMP is the protocol and reference implementation intended to realize that model.

## Why This Project Exists

Email is one of the most important systems on the internet, and one of the most expensive to defend.

The problem is not that spam filters are weak. The problem is that the underlying model is upside down.

In the current email ecosystem, sending is cheap, identity is weak, trust is mostly inferred after the fact, and the receiver pays the bill. Every serious mail provider and every serious business ends up doing the same thing: stacking more controls on top of SMTP to compensate for assumptions that no longer match the modern internet.

Those controls help. They do not fix the root problem.

This project exists to explore a better default that places the burden of abuse on the sender, makes identity verifiable, and makes secure delivery the normal path instead of a special feature.

## The Problem

Legacy email was designed for a more trusting network. It still assumes a world where anyone can try to reach anyone else at almost no cost, and where identity is only loosely attached to the message.

That creates a terrible incentive structure:

- Attackers can send at scale very cheaply.
- Defenders must spend real money filtering, scoring, quarantining, training, reviewing, and recovering.
- Users still get spoofed, phished, and overwhelmed.
- Operators still carry the cost of abuse.

In other words, modern email works by making the receiver smart enough to survive a protocol that lets the sender behave cheaply and anonymously.

_**That is a bad bargain.**_

## How the World Handles This Today

The current answer is layered defense.

Organizations rely on some combination of:

- Hosted email platforms
- SPF, DKIM, and DMARC
- Secure email gateways
- Sandboxing and link rewriting
- Reputation systems
- Anti-phishing tooling
- User awareness training
- Incident response and fraud recovery
- AI and heuristic filters
- Massive machine learning pipelines

These measures are necessary, but they are all compensating controls. They improve outcomes without changing the basic economics of abuse. Its a constant battle where spammers and phishers only need to find one weak point, while defenders have to cover every possible attack vector.

The result is an industry that spends _enormous_ time, money, and compute trying to decide which messages should never have been cheap to send in the first place.

## Why Legacy Email Remains Expensive

Modern email security is built from layers of compensating controls: hosted mail platforms, SPF/DKIM/DMARC, secure gateways, sandboxing, link rewriting, reputation systems, awareness training, incident response, and fraud recovery. Those controls help, but they do not change the basic economics of abuse.

Legacy email is now doing three jobs badly at once: human communication, organizational identity, and machine notification. Its core failure is economic. SMTP lets senders push messages at near-zero marginal cost while forcing receivers to spend money on filtering, storage, machine learning, user training, incident response, and fraud recovery.

That cost is not theoretical. Large providers process enormous volumes of abusive traffic just to discard it. Organizations routinely pay for hosted mail, add-on security products, user training, and operational response just to keep legacy email usable. The direct fraud losses are only the visible part of the problem. The hidden cost includes false positives, user distrust, helpdesk load, security operations overhead, and the permanent complexity of defending a protocol whose default incentives are wrong.

VAMP starts from a different premise: the protocol should make identity verifiable, delivery capability discoverable before send, native traffic encrypted by default, and suspicious scale expensive for the sender rather than the receiver.

The goal is not to build a prettier spam filter. It is to change the economics of messaging so abuse becomes harder to scale and easier to attribute.

For a quantified 2026 snapshot of operator burden and spend, see [The 2026 Cost and Operational Burden of Spam](docs/spam-costs-and-volumes.md).

## What We Believe Instead

We believe a modern messaging system should work differently.

Identity should be explicit. Trust should be verifiable. Encryption should be normal. Abuse should be expensive for the sender, not the receiver.

That leads to a simple design goal:

> Make legitimate communication easy, and make bulk abuse uneconomic by design.

This project is an exploration of what that could look like.

## What We Are Proposing

At a high level, this project explores Identity-First Mail (IFM), an identity-first, capability-aware messaging model with email-like addressing. VAMP is the protocol intended to implement that model.

The user experience should still feel familiar:

- Enter an address
- Write a message
- Click send

But underneath that familiar surface, the system behaves very differently.

### 1. Domain-Authorized Identity

A domain should be able to publish a cryptographic trust anchor for its users.

That means a message is not trusted because it claims to be from `alice@example.com`. It is trusted because the domain has explicitly authorized the identity and the message can be verified against that authority.

This moves email away from string-based identity and toward actual, verifiable principals.

This isn't a re-invention of cryptography or a new management nightmare. Its a re-application of well-known public key infrastructure concepts to the problem of messaging identity. The domain is the root of trust (Just like a certificate authority), and users and devices are authorized under that root.

### 2. Capability Discovery Before Send

The client should not wait until after sending to discover how a message can be delivered.

Before sending, it should be able to determine whether the destination domain supports the native system or whether the message must fall back to legacy email.

That allows the client to make smarter decisions up front:

- Verify recipient capability
- Verify identity state
- Detect revocation
- Determine whether secure native delivery is possible

This is a major shift from "send first, discover failure later."

### 3. Encryption by Default for Native Traffic

If recipient keys are discoverable as part of normal address resolution, encryption stops being a special feature and becomes the default path.

Users should not have to trade usability for confidentiality. The secure path should be the ordinary path.

### 4. Sender Cost for Suspicious Scale

The current system makes large-scale abuse too cheap.

We want a system where normal human communication remains easy, but bulk or unknown sending behavior triggers explicit cost. That cost could be computational, economic, or policy-based. The exact mechanism can evolve later.

For a concrete, operator-aligned cost model and initial threshold ideas, see [MTA-Scoped Sender-Cost Mechanics](docs/sender-cost-mechanics.md) and [Email Volume Baselines for Threshold Design](docs/email-volume-baseline.md).

The important idea is simple:

> The sender should bear the marginal cost of suspicious scale.

That flips the current asymmetry.

### 5. Revocation as First-Class State

Identities and devices should have real lifecycle state.

If an account is disabled or a device is revoked, that should be visible to the system before a user sends a message. The client should be able to fail early instead of relying on delayed delivery errors or silent trust assumptions.

### 6. SMTP as a Compatibility Bridge, Not the Core Model

This project does not assume the world can switch overnight.

A migration path matters, which means compatibility with legacy SMTP matters. But SMTP compatibility should be treated as a downgrade path, not as the native design center.

That distinction matters. A system that stays fully constrained by legacy email will inherit legacy email's problems forever.

Just as importantly, fallback cannot become an abuse loophole. If a sender domain supports VAMP but chooses to send over legacy SMTP instead, it is bypassing the identity, encryption, and sender-cost controls that make the new system valuable in the first place.

That means receiving systems need downgrade-aware policy. At a high level:

- If a domain advertises native VAMP capability, native delivery is the expected path.
- SMTP from a VAMP-capable sender should be treated as downgraded traffic, extremely suspicious, or outright policy-violating.
- Repeated or policy-violating downgrade attempts should be rejectable, throttleable, or reputation-damaging.
- Legacy SMTP remains available for domains that have not adopted VAMP, but it should not be a loophole for domains that have.

In other words, interoperability matters, but downgrade abuse must not be allowed to hollow out the protocol. Compatibility is a bridge for migration, not an escape hatch from accountability.

For a deeper deployment/threat-model analysis of downgrade resistance and operator incentives, see [Threat Model, Downgrade Resistance, and Operator Value](docs/threat-model-and-value.md).

## Docs

- [Threat Model, Downgrade Resistance, and Operator Value](docs/threat-model-and-value.md)
- [MTA-Scoped Sender-Cost Mechanics](docs/sender-cost-mechanics.md)
- [Email Volume Baselines for Threshold Design](docs/email-volume-baseline.md)
- [The 2026 Cost and Operational Burden of Spam](docs/spam-costs-and-volumes.md)

## What Success Looks Like

### For Users

Success looks boring in the best possible way:

- Less spam
- Fewer spoofed messages
- Clear trust indicators
- Secure delivery when possible
- No manual key handling

### For Operators and Admins

Success looks even better:

- Less filtering burden
- Stronger sender identity
- Clearer policy boundaries
- Deterministic revocation
- More control over abuse
- A migration path that does not require a flag day for the whole internet

### For the Broader Ecosystem

Success means shifting email from a receiver-pays abuse model to a sender-accountable model.

## What This Project Is Not

This project is not trying to recreate PGP with a nicer logo.

It is not a product pitch dressed up as a protocol. The goal is an open, implementable foundation for a better messaging model.

It is not asking ordinary users to manage personal key material, compare fingerprints, or carry long-lived cryptographic identity between employers. It makes no expectation that users will understand or interact with keys at all, in fact the contrary is true: the system should be designed to make key management invisible for users in the normal case.

It is not trying to make anonymous bulk messaging easier, verified bulk messaging is a use case that has not yet been planned for, but the system should be designed to allow it in a way that does not undermine the core value of verifiable identity and sender accountability.

It is not assuming that perfect privacy, perfect openness, perfect interoperability, and zero friction can all be had at once. Adversarial systems do not work that way.

And it is not pretending that a clean-sheet protocol replaces global email by decree. Migration is part of the problem and must be designed explicitly.

## What a Reference Implementation Needs

A useful reference implementation should prove the model, not just sketch it.

At minimum, it needs:

- A domain identity service
- A native message delivery service
- Capability discovery
- Domain-authorized user and device identity
- Client-side pre-send resolution
- Native signing and encryption
- Revocation and status checking
- Sender-cost enforcement for suspicious behavior
- Clear trust indicators in the client
- SMTP gateway support for legacy interoperability
- Admin tooling for enrollment, revocation, policy, and audit

That is enough to validate the architecture without locking the project into final protocol details too early.

## MVP

The first milestone should be narrow and practical.

A real MVP should prove that a small number of domains can exchange messages natively with:

- Domain-managed identities
- Per-device keys
- Pre-send capability checks
- Default encryption for native delivery
- Message signing
- Revocation of users and devices
- A simple policy engine
- SMTP fallback where explicitly allowed
- Receive-side trust indicators

That is enough to answer the most important question:

> Does this model actually improve the economics and usability of messaging?

## Future Work

There is plenty that can come later:

- Stronger recovery models
- Richer trust and reputation systems
- Advanced encrypted search
- Mailing lists and groups
- Delegated send and shared inboxes
- Mobile and desktop clients
- Compliance and archival features
- Multiple interoperable implementations
- Standardization work

Those are important, but they are not the first hill to climb.

For deeper analysis, start with the [docs](#docs) folder links above.

## Why This Matters

Email is still one of the internet's most important shared systems. It is also one of its most abused.

The world has spent decades building defensive layers around a protocol that makes abuse too cheap and trust too fuzzy. That has produced a lot of clever engineering and a lot of operational pain.

We think there is a better direction.

Not a prettier spam filter.  
Not another bolt-on trust signal.  
A different set of defaults.

This project exists to explore that path in the open, with VAMP as a concrete protocol for implementing the broader Identity-First Mail model.
