# Trust Tasks — Framework Specification

This repository is the **canonical source** for the *Trust Tasks* framework specification, developed under the [Trust Over IP Foundation](https://trustoverip.org) (ToIP) Decentralized Trust Graph Working Group (DTGWG).

A **Trust Task** is a self-contained, transport-agnostic, JSON-based description of an outcome that two parties agree to achieve. This specification defines the document structure, version scheme, namespace, and conformance requirements that every individual Trust Task specification is expected to satisfy.

- **Rendered specification:** <https://trustoverip.github.io/dtgwg-trust-tasks-spec/>
- **Registry of individual Trust Task specifications:** <https://trusttasks.org/>

## What lives where

| Repository | Contents |
|---|---|
| **this repo** (`dtgwg-trust-tasks-spec`) | The framework specification only — the normative document that individual specifications conform to. |
| [`dtgwg-trust-tasks-tf`](https://github.com/trustoverip/dtgwg-trust-tasks-tf) | The per-task spec/schema registry (`specs/<slug>/<version>/`), the transport bindings (`bindings/`), the registry website, and the generated Rust and TypeScript client libraries. |

Individual Trust Task specifications — `kyc-handoff`, `acl/grant`, `trust-task-error`, and the rest — are **not** in this repository. They stay in the registry repository, which consumes the framework specification published here.

## Authoring

The specification is written with [Spec-Up-T](https://trustoverip.github.io/spec-up-t-website/) and the [ToIP specification template](https://github.com/trustoverip/toip-template). Sources live under `spec/`:

| File | Contents |
|---|---|
| `spec/header.md` | Title, version, document status, editors, abstract, IPR terms. |
| `spec/intro.md` | Introduction, design goals, requirements language. |
| `spec/terms-and-definitions-intro.md` | Terminology preamble. |
| `spec/terms-definitions/*.md` | One file per defined term (`[[def: …]]`), referenced elsewhere as `[[ref: …]]`. |
| `spec/body.md` | The normative body, followed by the security, privacy, governance, internationalization, accessibility, conformance, and references sections. |
| `spec/appendix.md` | Appendix A (worked example), Appendix B (changelog), Appendix C (acknowledgements). |

The order in which these are concatenated is set by `markdown_paths` in `specs.json`.

```sh
npm install     # one-time
npm run render  # render to ./docs (gitignored)
npm run menu    # interactive Spec-Up-T menu (PDF, DOCX, freeze, xrefs, …)
```

Section headings carry **no manual numbering** — Spec-Up-T generates the table of contents and the `§` anchors. Cross-references are ordinary markdown links to the target heading's slug, e.g. `[Audience Binding](#audience-binding)`.

`docs/` is build output and is never committed; `.github/workflows/render-and-deploy.yml` renders on every push to `main` and publishes to the `gh-pages` branch.

## Contributing

- Open an [issue](https://github.com/trustoverip/dtgwg-trust-tasks-spec/issues) for substantive comments on the draft.
- Every commit must carry a `Signed-off-by` trailer per the [Developer Certificate of Origin](https://developercertificate.org/) — use `git commit -s`.
- A change to the framework that affects the wire format or the specification-authoring contract needs a corresponding changelog entry in `spec/appendix.md` and, once merged, a sync into the registry repository so the website and generated libraries follow.

## Licence

This specification is provided under the [JDF charter](https://cdn.platform.linuxfoundation.org/agreements/ToIP.pdf) for ToIP. Text is licensed [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); source code is licensed [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).
