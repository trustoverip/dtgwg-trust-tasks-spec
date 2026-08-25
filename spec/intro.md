## Introduction

*This section is informative.*

Two parties interoperate when they agree on the shape of the work they cooperate on. Today that agreement is reached ad-hoc: every onboarding flow, every consent receipt, every credential exchange is described in a vendor-specific schema, carried over a vendor-specific protocol, and validated by vendor-specific code. The result is a combinatorial explosion of pairwise integrations.

A *[[ref: Trust Task]]* is a single, finite description of an outcome between two parties — a KYC handoff, a consent grant, a payment commitment — that is portable across implementations because the task definition is decoupled from the transport that delivers it.

Three properties make a Trust Task portable:

1. **Self-contained** — the document carries everything needed to act on it: parties, criteria, schema, identifiers. No hidden context.
2. **Transport-agnostic** — the document makes no assumption about the protocol that delivers it. DIDComm, HTTPS, message queue, paper — the task is the task.
3. **JSON-based** — the canonical serialization is a single JSON object validated against a published JSON Schema.

The body of this framework specification defines the document structure, version scheme, and namespace shared by every individual Trust Task specification published under the registry.

This is a Working Draft prepared by the Trust Tasks Task Force of the Decentralized Trust Graph Working Group (DTGWG) of the [Trust Over IP Foundation](https://trustoverip.org). Publication as a Working Draft does not imply endorsement by the ToIP membership. Comments are welcome via the [issue tracker](https://github.com/trustoverip/dtgwg-trust-tasks-spec/issues). The editors expect the substantive sections — in particular [Minimum Requirements](#minimum-requirements), [Error Responses](#error-responses), [Transport Bindings](#transport-bindings), and [Security Considerations](#security-considerations) — to evolve as individual Trust Task specifications progress through the [Maturity Levels](#maturity-levels) and surface gaps in this framework.

The individual Trust Task specifications that conform to this framework, together with the transport bindings and the generated client libraries, are maintained in the [Trust Tasks registry repository](https://github.com/trustoverip/dtgwg-trust-tasks-tf) and published at <https://trusttasks.org/>.

### Design Goals

*This section is informative.*

The framework aims to solve four related problems that arise wherever two or more parties cooperate over a network.

1. **A common task vocabulary across any transport.** In a decentralized ecosystem there is no single message bus or RPC framework: parties speak DIDComm, HTTPS, message queues, paper, and anything else. [[ref: Trust Tasks]] let two parties agree on *what* they are doing without first agreeing on *how* the bits move between them. The same task specification works regardless of the transport carrying it.

2. **Security, privacy, and identity that scale to the transport.** A [[ref: Trust Task document]] can rely on the integrity, authentication, and party-identity guarantees already provided by the transport in use — for example, mutually-authenticated TLS or a signed DIDComm envelope — and where those guarantees are absent, the document's own `proof`, `issuer`, and `recipient` members ([Proof](#proof), [The `issuer` and `recipient` Members](#the-issuer-and-recipient-members)) supply them in-band. Implementers can match cryptographic work to the threat model in front of them rather than always paying the worst-case cost.

3. **Payload freedom, declared at the boundaries.** The framework defines the outer document shape and deliberately leaves the `payload` unconstrained. Each [[ref: Trust Task specification]] chooses its own payload structure, JSON Schema, and — where useful — JSON-LD context. The framework only requires that each choice be declared explicitly ([Specification Requirements](#specification-requirements)) and be machine-validatable.

4. **A standard family of response types.** Many tasks need a structured way for a [[ref: recipient party]] to report what happened. The framework reserves a small set of response-type [[ref: Trust Task specifications]] addressing the common cases — failure ([Error Responses](#error-responses)), success with metadata (`trust-task-ok`), and a recipient-suggested continuation (`trust-task-next-step`) — each itself a *Trust Task* so that one validation, signing, and transport pipeline serves both the task and its response. All three are specified as of this revision (see [Error Responses](#error-responses) and [Reserved Response-Type Slugs](#reserved-response-type-slugs)).

## Requirements Language

*This section is normative.*

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in BCP 14 [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals, as shown here.
