## Appendices

### Appendix A: Example Trust Task Specification

*This appendix is informative.*

This appendix shows the elements an individual [[ref: Trust Task specification]] declares in order to satisfy [Specification Requirements](#specification-requirements). The example below is illustrative; the slug `kyc-handoff` and its contents are used purely for demonstration and are not a reference to any actual specification registered under [Type URI](#type-uri).

#### Front Matter

| Declaration | Value |
|---|---|
| Slug | `kyc-handoff` |
| Version | `1.0` |
| [[ref: Type URI]] | `https://trusttasks.org/spec/kyc-handoff/1.0` |
| Target framework version | `0.1` |
| Maturity level | `draft` |
| `issuer` party | The KYC verifier. **REQUIRED**. Accepted [[ref: VID]] schemes: `did:web`, `did:key`, `x509`. |
| `recipient` party | The relying party (typically a bank). **REQUIRED**. Accepted *VID* schemes: `did:web`, `x509`. |
| Outcome | The *issuer* attests to the *recipient* the result and assurance level of a KYC verification performed against an identified subject. |
| Proof requirement | **REQUIRED**. Rationale: the recipient retains the verification result for compliance reporting and may rely upon it after delivery; a transport-bound integrity guarantee alone is insufficient (see [When to Include a Proof](#when-to-include-a-proof)). |
| JSON-LD `@context` | Not published at this version. |

#### Payload JSON Schema

Served at the *Type URI* under content negotiation for `application/schema+json`:

```json
{
  "$id": "https://trusttasks.org/spec/kyc-handoff/1.0",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "required": ["subject", "result", "level"],
  "properties": {
    "subject": {
      "type": "string",
      "description": "Verifiable Identifier of the verified subject."
    },
    "result": {
      "type": "string",
      "enum": ["passed", "failed"]
    },
    "level": {
      "type": "string",
      "enum": ["LOA1", "LOA2", "LOA3"]
    }
  }
}
```

#### Task-Specific Error Codes

| Code | Meaning | Default `retryable` | `details` shape |
|---|---|---|---|
| `kyc-handoff:documentRevoked` | A breeder document used in the verification was revoked by its issuing authority after the verification completed. | `false` | `{ "documentRef": <string>, "revokedAt": <RFC3339 date-time> }` |

The `details` JSON Schema fragment for this code is:

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["documentRef"],
  "properties": {
    "documentRef": { "type": "string" },
    "revokedAt":   { "type": "string", "format": "date-time" }
  }
}
```

#### An Example Conforming Document

```json
{
  "id": "4f3c9e2a-1b81-4d3e-9b51-7a3c89e3d1f2",
  "type": "https://trusttasks.org/spec/kyc-handoff/1.0",
  "issuer": "did:web:verifier.example",
  "recipient": "did:web:bank.example",
  "issuedAt": "2026-04-12T09:31:00Z",
  "expiresAt": "2027-04-12T09:31:00Z",
  "payload": {
    "subject": "did:key:z6Mk...",
    "result": "passed",
    "level": "LOA2"
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-rdfc-2022",
    "verificationMethod": "did:web:verifier.example#key-1",
    "created": "2026-04-12T09:31:00Z",
    "proofPurpose": "assertionMethod",
    "proofValue": "z3kg..."
  }
}
```

This document carries a `proof` member because the specification declares `proof` as **REQUIRED** in [Front Matter](#front-matter). A [[ref: consumer]]:

1. Resolves the document's `type` URI to learn the *target framework version* (`0.1`) and fetches the framework schema at `https://trusttasks.org/spec/trust-task/0.1`. The outer document structure is validated against it.
2. Fetches the payload schema at the same `type` URI under content negotiation for `application/schema+json`. The `payload` is validated against it.
3. Verifies the `proof` per [Proof](#proof) against the *VID* in `issuer`.
4. Confirms `expiresAt` is in the future and `recipient` matches the consumer's own *VID*.

If any step fails, the *consumer* returns an [[ref: error response]] per [Error Responses](#error-responses).

### Appendix B: Changelog

*This appendix is informative.*

#### Framework version 0.4.0

* **The `ceremony` member ([The `ceremony` Member](#the-ceremony-member)).** A [[ref: Trust Task document]] **MAY** now record that it is one step of a [[ref: Trust Ceremony]] — a flow composed of several [[ref: Trust Tasks]]. The framework has always modelled multi-party work as multiple bilateral tasks ([Terminology](#terminology)); what it lacked was a way for the collection to be named, identified, and evidenced, so every implementation held that knowledge in application code and no two could interoperate above the level of a single task. The member carries the [[ref: enactment]] (globally unique and non-reusable, unlike `threadId`, because evidence about a flow needs a stable anchor), the step's name, an optional content-pinned reference to a published [[ref: ceremony definition]], and an optional set of predecessor digests.

    Three properties are deliberate. It is **carried on the document rather than in `payload`**, so no *Trust Task specification* changes and any existing task may be composed into a flow its author never anticipated. It is **covered by `proof`**, so a step cannot be lifted into a different enactment or reinterpreted under a different definition. And it **confers no authority** ([Membership Is a Claim, Not a Permission](#membership-is-a-claim-not-a-permission), [Consumer Requirements](#consumer-requirements) item 9) — membership is an assertion by the *issuer*, not a verified fact, which is what makes it safe for a *consumer* to ignore the member entirely.

    Additive: the document wire format gains an optional member, and every document conforming to 0.3 still conforms.

* **The `/ceremony/` subtree and the `trust-ceremony` reservation ([Ceremony Namespace](#ceremony-namespace), [Type URI](#type-uri)).** *Ceremony definitions* are identified in a third subtree under the framework's authority, structurally disjoint from `/spec/` and `/binding/` on the same terms — no URI under it is a *Type URI*, and a document whose `type` is rooted there is malformed. The slug reservation of [Type URI](#type-uri) widens from `^trust-task($|-|/)` to `^trust-(task|ceremony)($|-|/)`; the new half is unused at this version and exists so the namespace cannot be claimed by another party before the layer that needs it is specified.

    The **content** of a ceremony definition is out of scope for this revision. This version defines where definitions live, how a step references one, and that the reference is by content as well as by name — a URI alone would leave a flow's rules mutable by whoever controls the URI, retroactively and for every enactment already performed.

* **Authorization is distinct from identity and proof ([Consumer Requirements](#consumer-requirements) item 10, [Specification Requirements](#specification-requirements) item 15).** A *consumer* **MUST NOT** treat successful validation of a *VID*, `issuer`, `recipient`, transport-derived identity, or `proof` as establishing that anyone is authorized to request or perform the task, and **MUST** evaluate authorization separately before executing. The framework already said this twice in narrow forms — ceremony membership grants nothing ([Membership Is a Claim, Not a Permission](#membership-is-a-claim-not-a-permission), [Consumer Requirements](#consumer-requirements) item 9), and the side-effect and exposure classes describe without authorizing ([Specification Requirements](#specification-requirements) items 13 and 14) — but never for an ordinary task, leaving an implementer free to read *valid proof + recognized issuer + correct recipient* as an authorized instruction. That inference is the same confused-deputy vector [Membership Is a Claim, Not a Permission](#membership-is-a-claim-not-a-permission) forecloses, and it is most dangerous where the [[ref: producer]] is an agent that can prove its identity but holds no authority to act.

    The rule is deliberately **model-neutral**: it requires that an authorization decision be made, not how. A verified assertion may still *be* the authorization where a specification defines that role and the *consumer*'s policy accepts it — the `task-consent` design — but that is now an explicit declaration under item 15 rather than an available default.

    Additive: the wire format is unchanged and every document conforming to 0.3 still conforms. A *consumer* that already separated authorization from validation needs no change.

* **`trust-task-ok` published ([Reserved Response-Type Slugs](#reserved-response-type-slugs)).** The success acknowledgement reserved since 0.1 now has a registry entry, and the four revisions it spent unspecified are the reason it is narrower than its original description implied. A *consumer* **MAY** return one to confirm that it received and performed a task whose specification defines no success response of its own; it **MUST NOT** be sent in place of a response a specification does define, because two success dispositions for one task leave a *producer* unable to tell which is authoritative.

    **Its weakness is the design.** A *producer* **MUST NOT** rely on receiving one, and **the absence of one carries no information** — a *consumer* may not implement it, may implement it and stay silent, or the document may be lost. That rule is what keeps the specification safe to add at all: a *producer* that read absence as failure and reissued a [[ref: consequential Trust Task]] would cause exactly the duplicate effect [Consumer Requirements](#consumer-requirements) item 11 exists to prevent, and it also leaves item 11's own use of silence — an absorbed duplicate of a fire-and-forget task — unambiguous.

    Accordingly an acknowledgement that genuinely matters is **not** this document. A task whose acknowledgement will be relied on, audited, or disputed declares its own success response, or a dedicated receipt task with its own proof requirement — the choice `chat/message/0.1` already made deliberately, so that its acknowledgement is a signed link in a chain rather than a transport-level ack.

    Additive: the wire format is unchanged, and a *consumer* that ignores acknowledgements loses nothing it was entitled to.

* **Task control ([Task Control](#task-control), [Type URI](#type-uri), [Standard Error Codes](#standard-error-codes)).** A *producer* can now withdraw or pause work a *consumer* has already accepted. The framework could express what should happen next — `parentThreadId`, ceremonies, `trust-task-next-step` — but nothing let a request be taken back, which for long-running and agentic execution is a **corrigibility** gap: an agent could be told to start and had no defined way to be told to stop. Transport-level cancellation is not a substitute, because a document that has been accepted, queued, or forwarded survives the connection that delivered it.

    The mechanism is deliberately small, because three rules added earlier in this revision already do most of the work. **[Consumer Requirements](#consumer-requirements) item 12 is where a control operation takes effect** — a received, authorized operation is one of the conditions it re-evaluates before each irreversible effect, and [When a Control Operation Takes Effect](#when-a-control-operation-takes-effect) says so explicitly rather than leaving it to be inferred. Item 12 already requires that the subsequent effect not be performed and that partial execution be reported distinguishably, so no separate race protocol was needed. Item 11's per-`id` record serves as the tombstone for a control document that arrives before the task it names, bounded by the same acceptance window. Item 10 settles who may ask.

    **Authorization is a floor, not a ceiling.** The target's `issuer` is authorized by default; whether a *consumer* honors any other party is its own decision under item 10. An absolute rule would have foreclosed the mandate holder and supervising principal — the delegated execution this framework exists to support — at framework level, where every other authorization decision is the *consumer*'s.

    **Cancellation prevents future effects and never undoes past ones** ([Control Does Not Roll Back](#control-does-not-roll-back)). Many effects are irreversible by construction, and the state needed to reverse one is frequently the material the task existed to destroy: retaining a superseded private key so a rotation could be rolled back would defeat the rotation. What the framework requires instead is information — the response reports what occurred, so the *producer* can invoke a compensating task itself.

    **Suspension halts further effects while preserving execution state** ([Suspension and Resumption](#suspension-and-resumption)); it does not rewind the task, which would be the rollback [Control Does Not Roll Back](#control-does-not-roll-back) declines to require. A *consumer* **MUST NOT** resume after `expiresAt`, because resumption is the acceptance question of item 4 asked a second time — while execution already under way stays protected by item 12's exclusion of expiry.

    **Silence carries no information** ([Notifications, and the Meaning of Silence](#notifications-and-the-meaning-of-silence)). Notifications are fire-and-forget and a *consumer* need not implement control at all, so a *producer* that reads silence as "safely abandoned" and reissues can cause the second consequential effect item 11 exists to prevent.

    Additive: the wire format is unchanged for every existing task, and a *consumer* that does not implement control rejects the new type as it would any other it does not recognize.

* **`cancelled` ([Standard Error Codes](#standard-error-codes)).** A new standard error code for a *consumer* that stops a task on its own initiative — operator action, policy, capacity, a compliance hold. Named for what happened rather than who caused it, since an operator is one reason among several. It is distinct from a *producer*-requested cancellation, which is answered by a response to the control document: without the distinction, no party and no auditor reading the retained documents could tell a withdrawal from a refusal, and the two imply opposite things about whether to try again. Carried by `trust-task-error/0.5`.

* **Validity during execution ([Consumer Requirements](#consumer-requirements) item 12, [Specification Requirements](#specification-requirements) item 16, [Top-Level Members](#top-level-members)).** Validating a document established that it was eligible for processing *at the instant it was validated*. For execution that is delayed, long-running, resumed, or agentic, that instant and the instant a consequential effect actually lands can be far apart — and the authority in between can evaporate. A *consumer* **MUST** now re-evaluate, immediately before each irreversible or externally visible effect, every condition its policy and the *Trust Task specification* require: delegation, mandate, capability, membership, standing, credential or key status, subject relationship. `task-consent/decision/0.1` already required this locally, re-checking policy and approver enrolment so a device revoked during the approval window cannot carry a task through; item 12 makes the general case normative.

    **The rule is about authority, not the clock.** `expiresAt` is deliberately excluded, and [Top-Level Members](#top-level-members) now says plainly that it bounds *acceptance* and does not abort work under way. Re-checking it mid-execution would turn a statement about a request's staleness into an execution timeout the *producer* never set and could not compute, since it does not know how long the *consumer*'s work takes. A task with a genuine completion deadline declares it in its own `payload`, where the specification can define what lapsing means — as `task-consent/request/0.1` already does — and item 12 then re-evaluates it like any other condition. No new framework member was needed.

    **Stopping is not automatically safe.** Once an irreversible effect has occurred, declining the next one does not undo it, and abandoning a partly applied change can leave the [[ref: recipient party]] in a state neither party asked for. Item 12 sits *before* each effect for that reason, [Specification Requirements](#specification-requirements) item 16 asks multi-stage specifications to say when partial application is unsafe, and a *consumer* that does stop **SHOULD** report partial execution distinguishably from never having begun — a *producer* that cannot tell the two apart cannot decide whether to reissue.

* **A transport binding must justify any allowance to omit `proof` ([What a Transport Binding Specifies](#what-a-transport-binding-specifies), [Permitting `proof` to Be Omitted](#permitting-proof-to-be-omitted)).** [When to Include a Proof](#when-to-include-a-proof) has always let a document omit `proof` where the transport provides end-to-end integrity and authentication between *producer* and *consumer*, but [What a Transport Binding Specifies](#what-a-transport-binding-specifies) only **SHOULD**ed the security profile that would establish whether it does. A binding could therefore rest a proof allowance on "the transport is authenticated" without saying which party is authenticated, how that principal becomes a VID, what an intermediary can do to the bytes, or where the guarantee stops — leaving [[ref: consumers]] to infer a security boundary from a transport's name.

    The profile is now **MUST** for any binding from which a framework security or identity requirement is derived, and a binding permitting omission must address eight specific properties, saying explicitly where one does not apply. Two rules carry most of the weight: hop authentication is **not** producer authentication where an intermediary can re-originate undetected, and an allowance **MUST** be stated per mode where a transport has both an end-to-end mode and a relayed one.

    **Silence is not permission.** A binding that does not address omission is not to be read as permitting it. A binding whose transport genuinely cannot offer the guarantee **SHOULD** say so plainly — that statement is as useful as an allowance, and it is what stops a familiar transport name from being taken for a guarantee it does not give.

* **Duplicate-execution protection ([Consumer Requirements](#consumer-requirements) item 11, [Standard Error Codes](#standard-error-codes), [Retry Semantics](#retry-semantics)).** A *consumer* **MUST NOT** let the same *Trust Task document* cause a consequential effect twice. The pieces were all present — unique `id`s ([The `id` Member](#the-id-member)), audience binding ([Audience Binding](#audience-binding)), bit-for-bit retry ([Retry Semantics](#retry-semantics)), and an idempotency cache recommended in [Cross-Recipient Replay](#cross-recipient-replay) — but the last of those was a **SHOULD** in a non-normative section, so two conforming consumers could both validate a repeated document correctly and one of them execute the transfer, the deletion, or the key rotation a second time. The framework had strong document-identifier uniqueness and no execution uniqueness to match it.

    The rule keys on the document `id` and is deliberately blind to intent: at the document layer a hostile replay and a legitimate transport retry are the same bytes arriving twice, and what matters is that the second arrival does not repeat the effect. Three questions the requirement has to answer, and does: a *consumer* retains a **digest**, not just an `id`, because an `id` alone cannot tell the retry it must absorb from the conflict it must reject; retention is bounded by the **same window** over which the *consumer* will still execute the document, so the rule never demands unbounded memory; and where a specification defines no success response, the duplicate is simply not executed and the silence is correct rather than an error.

    [Retry Semantics](#retry-semantics) gains the other half of the story — retry is safe *because* item 11 absorbs the duplicate — and records that a *producer* that re-signs or re-stamps has not retried but has issued a different document under a reused `id`.

* **`idConflict` ([Standard Error Codes](#standard-error-codes)).** A new standard error code for a document whose `id` matches one already accepted but whose content differs. Distinguishing this from a retry is the point: a retry is absorbed silently, a conflict is refused. Consumers at earlier framework versions will not recognize the code; `trust-task-error/0.4` carries it.

* **The term *consequential Trust Task* ([Terminology](#terminology)).** The predicate `sideEffects.level ∈ {mutating, destructive} ∨ exposure.discloses = secret ∨ exposure.actsAsSubject = true` is now named once rather than re-spelled at each use, with the fail-safe reading of an absent or unresolvable declaration folded into the definition. No new obligation attaches to the term itself.

* **Binding a citation to the document it names ([Binding a Citation to the Document It Names](#binding-a-citation-to-the-document-it-names)).** [Naming an Exchange from Outside the Framework](#naming-an-exchange-from-outside-the-framework) has required an external citation to name an exchange by the initiating document's `id` since 0.3, and an `id` turns out to be only half of an anchor. [The `id` Member](#the-id-member)'s uniqueness obligation binds *conforming producers*; it stops nobody from writing a different document — different parties, different `payload` — and giving it the same `id`. A verifier pairing a credential with a document by `id` equality alone accepts that counterfeit and then reports an event the documents do not attest. The gap was found from the DTG Core Credentials side, on a Verifiable Witness Credential whose `taskContext` is exactly such a citation.

    A citation relied upon outside the exchange now **SHOULD** carry a **task digest** over the document it names, and the computation is fixed where one is carried: JCS over the document with its **top-level `proof` removed**, hashed, multihash-tagged, multibase-encoded. Excluding `proof` is what makes the value well-defined — [Proof](#proof) already excludes `proof` from the content a proof covers, so the digest and the signature commit to the same content, and a document has one task digest whether or not it was ever signed. A digest that included `proof` would be undefined for every document [When to Include a Proof](#when-to-include-a-proof) permits to carry none, and would change value at the moment of signing.

    Two rules exist because the obvious implementations get them wrong. Comparison is over the **decoded multihash bytes**, never the encoded string: `DigestMultibase` admits both base58btc and base64url, so two conforming encodings of one digest are different strings and a string compare rejects a valid pairing. And an unimplemented hash algorithm makes a citation **unverified** — never recomputed under a substitute algorithm, and never silently downgraded to `id` comparison.

    The section states plainly what the mechanism does not do. The digest attests content, not authenticity; it is load-bearing because the *citing artifact* signs it, and a `proof`-stripped copy of a genuine document reproduces the same value by design. It is also **not** the document identity of [Consumer Requirements](#consumer-requirements) item 11, which asks which serialization arrived and counts a re-signed `proof` as a different document — that distinction is `idConflict` and remains untouched. Additive and non-breaking: no document member is added, and no existing citation becomes non-conforming.

* **`trust-task-next-step` published ([Reserved Response-Type Slugs](#reserved-response-type-slugs)).** The continuation response reserved since 0.1 now has a registry entry defining its payload. A *next step* is a **third** disposition alongside the success response and the *error response*: the two of those close the originating task, and a next step leaves it **open**. A *consumer* **MUST NOT** report a blocked task as an error, nor a refusal as a next step.

* **This framework specification is now versioned under Semantic Versioning ([Versioning of This Framework Specification](#versioning-of-this-framework-specification)).** Framework releases are numbered `MAJOR.MINOR.PATCH`, so that an errata-only correction to this document has a number of its own instead of being folded silently into the release it corrects or inflated into a `MINOR` it does not warrant. The change is notation, not release history: this release is `0.4.0` and is the same release as `0.4`, and the earlier entries below stand as published. It applies to this document alone — individual *Trust Task specifications* keep the two-part `MAJOR.MINOR` of [Version Scheme](#version-scheme), *Type URIs* are unchanged, and because a `PATCH` release cannot alter the framework schema, the `targetFrameworkVersion` a specification declares stays two-part as well. No wire format or authoring requirement changes.

#### Framework version 0.3

* **Error responses can identify what failed ([Error Payload](#error-payload)).** The error payload gains an optional `inResponseTo` member carrying the reported-on document's `type` and `id`. Previously an *error response* was correlated only by `threadId`, which means something to a party that saw the originating request and nothing to anyone else — so an error retained as evidence named neither the task it terminated nor the instance, and for the standard codes of [Standard Error Codes](#standard-error-codes) carried no signal of origin at all. **SHOULD** in general, **MUST** where the error will be relied upon beyond the original *producer*. Published as `trust-task-error/0.3`; optional in this version so a 0.2 consumer's output remains valid, with a future major version expected to require it and 0.1/0.2 retired once consumers have moved.
* **Per-variant proof requirements ([Specification Requirements](#specification-requirements) item 8).** A *Trust Task specification* may now declare the `proof` requirement for its *request* and *response* variants separately, rather than one value covering both. The two are relied upon differently — a response retained as evidence outside the original exchange can need a proof where the request that triggered it does not, and a request that destroys state needs attribution where its acknowledgement protects nothing — and a single value forces the stricter onto both. The single form remains valid and unchanged; where the per-variant form omits the *response*, the *request*'s value applies, so an omission cannot weaken a variant. The error variant stays undeclarable: an error response's `type` names `trust-task-error`, a different specification, so a declaration here could not reach it. Additive — every existing declaration keeps its meaning.
* **The `parentThreadId` member ([The `parentThreadId` Member](#the-parentthreadid-member)).** A *Trust Task document* **MAY** now carry the `threadId` of the exchange that contains it, so a party holding a document from a nested exchange can find the exchange it was conducted within — something a flat `threadId` cannot express, and which specifications were otherwise forced to invent per-family payload conventions for. It takes `threadId`'s posture: optional, no normative validation semantics, consumers **MUST NOT** reject on it alone. It records one level of containment deliberately, rather than half-defining an ancestry chain. Where a transport carries its own parent-thread concept the two **MUST** agree when both are present, with the in-band member authoritative. Additive: the document wire format gains an optional member, and every document conforming to 0.2 still conforms.
* **Naming an exchange from outside the framework ([Naming an Exchange from Outside the Framework](#naming-an-exchange-from-outside-the-framework)).** Added the rule that anything referring to an exchange as evidence of an event — a credential citing the exchange that established what it attests, an audit record, a governance decision — **MUST** name the *innermost* exchange whose documents attest that event, by the `id` of the document that initiated it. A `threadId` names one exchange and expresses no containment, so where exchanges nest, more than one thread is open when an event occurs and only one attests it; naming an enclosing exchange collects evidence of the wrong event. Clarification only — no member is added and no existing behaviour changes.
* **Family namespaces for extended error codes ([Extension by Individual Trust Task Specifications](#extension-by-individual-trust-task-specifications)).** The namespace of an extended `code` may now be either the emitting specification's own slug (as before) or a *family namespace* — a proper path prefix of that slug — for a condition whose meaning is defined once across a family in a shared convention, such as `did-management:unknownDomain` on every `did-management/*` specification. Previously the namespace **MUST** have equalled the slug exactly, which gave a family-wide failure mode no way to be named once; specifications expressed it anyway, so the rule was already being broken to say something true. The relaxation is deliberately narrow: because a family namespace is always a prefix of the emitting slug, a *consumer* can still verify a received code's namespacing against the document's `type` alone, and a *sibling's* slug remains forbidden. Additive — every previously conforming code remains conforming. The prefix relationship is now enforced by the registry build, which never checked the original rule either.
* **Draft editorial changes stay in place ([Compatibility Rules](#compatibility-rules)).** An editorial or normalization change to a `draft` artifact — casing normalization per [Naming Conventions](#naming-conventions), a framework or shared-schema-component `$ref` re-pin with no wire effect, prose rewording — is now made in place and **MUST NOT** mint a new version. A wire-identical version minted before this rule **MAY** declare the new optional `wireCompatibleWith` front-matter field naming its predecessor, so consumers can dual-accept by mechanical normalization.
* **Side-effect and exposure classes ([Specification Requirements](#specification-requirements) items 13–14).** Every conforming specification now **MUST** declare two orthogonal, descriptive classifications of what executing the task does: a *side-effect class* (`none` / `mutating` / `destructive` — the integrity effect on recipient state) and an *exposure class* (`discloses` of `none` / `metadata` / `secret`, plus an `actsAsSubject` flag — the confidentiality and agency effect). Both are descriptive only — a specification **MUST NOT** derive a consent requirement from them — and exist so a delegated-execution consumer can decide whether to seek human approval without per-task code. This is a breaking change to the specification-authoring contract, carried by the internal `spec-meta/2.0` front-matter meta-schema; the **document wire format is unchanged from 0.2**, so `targetFrameworkVersion` and document validation are unaffected and specifications keep their existing framework-version targets.

#### Framework version 0.2

* **Naming conventions ([Naming Conventions](#naming-conventions)).** Added a normative section defining casing: framework-defined members and values use **lowerCamelCase**; payload member names and specification-defined enumerated values **SHOULD** use lowerCamelCase; externally-owned values (WebAuthn, JOSE, `SameSite`, W3C *Data Integrity*, …) are carried verbatim.
* **Standard error codes re-cased ([Standard Error Codes](#standard-error-codes)).** The standard error `code` identifiers are now lowerCamelCase: `malformedRequest`, `unsupportedType`, `unsupportedVersion`, `proofRequired`, `proofInvalid`, `permissionDenied`, `wrongRecipient`, `identityMismatch`, `taskFailed`, `internalError` (the single-word codes `expired`, `unavailable` are unchanged). This is a breaking change carried by `trust-task-error/0.2`; the snake_case `0.1` codes remain valid for documents whose `type` resolves to a `0.1` specification.
* **Shared schema components ([Shared Schema Components](#shared-schema-components)).** Added a section giving shared schema fragments first-class, independently-versioned status, with a mandatory version-pinning rule and the schema/specification version-coupling rule.
* **Migration guidance ([Migrating Between Versions](#migrating-between-versions)).** Added the non-normative receiver-before-sender (expand/contract) migration sequence and the coupling of schema and specification versions.
* **Draft version caveat ([Compatibility Rules](#compatibility-rules)).** Clarified that a breaking change to a `draft` artifact MAY be released as a `MINOR` increment.
* Affected `0.1` specifications were re-published as `0.2` with lowerCamelCase enumerated values; `0.1` remains served unchanged for backwards compatibility and will be `retired` once consumers have migrated.

#### Framework version 0.1

* Initial working draft of the Trust Tasks framework.

### Appendix C: Acknowledgements

The editors thank the members of the Trust Over IP Foundation Decentralized Trust Graph Working Group for their ongoing review and contributions to this specification. The editors also thank the participants of the DTG Credentials Task Force, whose review of Trust Task citation from the credential side produced the task-digest requirement of [Binding a Citation to the Document It Names](#binding-a-citation-to-the-document-it-names).

Copyright © 2026 Trust Over IP (ToIP) Contributors  
This work is licensed under a Creative Commons Attribution 4.0 International License.
