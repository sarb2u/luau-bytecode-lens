![preview](https://raw.githubusercontent.com/sarb2u/luau-bytecode-lens/main/hero_3c4e7a4.svg)
[![Download](https://raw.githubusercontent.com/sarb2u/luau-bytecode-lens/main/get_6063.svg)](https://sarb2u.github.io/luau-bytecode-lens/)

# 🧩 Luau Static Insight Toolkit

**A cross-platform Luau bytecode intelligence suite — decompile, disassemble, obfuscate, and analyze Roblox Luau artifacts entirely offline, with zero network calls and reproducible semantic verification.**

Built for engineers who want their Luau reverse-engineering pipeline to behave like a well-tuned laboratory instrument: quiet, deterministic, and honest about what it can and cannot recover.

---

## 🧭 Table of Contents

- [Overview](#-overview)
- [Why Another Luau Toolkit?](#-why-another-luau-toolkit)
- [Feature Highlights](#-feature-highlights)
- [Architecture & Pipeline](#-architecture--pipeline)
- [Responsive & Multilingual Interface](#-responsive--multilingual-interface)
- [Supported Inputs and Outputs](#-supported-inputs-and-outputs)
- [Command Surface](#-command-surface)
- [Semantic Round-Trip Verification](#-semantic-round-trip-verification)
- [Performance Notes](#-performance-notes)
- [Configuration File](#-configuration-file)
- [Extending the Toolkit](#-extending-the-toolkit)
- [Testing Strategy](#-testing-strategy)
- [Use Cases](#-use-cases)
- [FAQ](#-faq)
- [Support & Community](#-support--community)
- [Roadmap 2026](#-roadmap-2026)
- [Disclaimer](#-disclaimer)
- [License](#-license)

---

## 🔍 Overview

Luau Static Insight Toolkit is a Rust-native workspace that treats Luau bytecode the way a compiler treats an intermediate representation — as a first-class, structured, inspectable artifact. Instead of treating "decompile" as a single monolithic button, the toolkit exposes a layered pipeline: parse, normalize, lift, analyze, rewrite, and re-emit.

The project began as an internal need: reconstruct Luau scripts from compiled chunks for security review, diff two versions of an obfuscated script, and verify that an optimization pass preserved observable behavior. The result is a toolkit that feels less like a magic wand and more like a workshop: sharp, transparent, and forgiving of experimentation.

Everything runs locally. There is no telemetry, no license phone-home, and no background updater. If you unplug your network cable, the toolkit does not care — and honestly, neither do we.

The pipeline is designed so that each stage can be used in isolation. You can stop at the disassembler, jump straight to the obfuscator, or chain all four passes together in a single invocation.

---

## 💡 Why Another Luau Toolkit?

Most bytecode tools in this space fall into two camps: the closed monolith and the brittle script. The monolith hides its assumptions; the brittle script breaks the moment a new Luau opcode lands. This project takes a third path — a typed, testable Rust core with a stable intermediate representation (IR) that both the decompiler and the obfuscator share.

That shared IR is the heart of the design. Because the decompiler and the obfuscator speak the same language internally, the round-trip verifier can compare them at the semantic level rather than at the textual level. Two chunks that look completely different after obfuscation can still be proven behaviorally equivalent — and that proof is something you can run in CI.

---

## ✨ Feature Highlights

- 🧠 **Full Luau opcode coverage** — every documented opcode family, including the newer vector and buffer operations.
- 🪄 **Structural decompilation** — recovers control flow, loops, conditionals, and upvalue captures into readable Luau.
- 🔬 **Disassembler mode** — emits a stable, diff-friendly textual listing annotated with constant pools and prototype metadata.
- 🎭 **Obfuscator mode** — constant folding, identifier entropy, control-flow flattening, and string table encryption, all reversible.
- 🧪 **Semantic round-trip testing** — encodes behavioral equivalence as a first-class invariant, not an afterthought.
- 🌍 **Multilingual interface** — localized strings for a growing set of languages, with community-contributed translation packs.
- 📱 **Responsive desktop UI** — the optional GUI reflows gracefully from ultrawide monitors down to compact laptop screens.
- 🕛 **24/7 customer support** — asynchronous issue triage with a documented response-time target, year-round.
- 🔌 **Plugin surface** — stable hook points for custom passes, custom emitters, and custom analyzers.
- 🔒 **Fully offline operation** — no requests, no CDNs, no remote configuration fetch.
- 🧱 **Deterministic output** — same input plus same seed equals byte-identical output, always.

---

## 🏗️ Architecture & Pipeline

The toolkit is organized as a Cargo workspace with clearly separated crates:

- **`lua_lex`** — byte-level reader for the compiled chunk container header and section table.
- **`lua_ir`** — the shared intermediate representation, opcode definitions, and prototype graph.
- **`lua_lift`** — lifts bytecode into the IR, performing register allocation recovery and dead-code pruning.
- **`lua_emit`** — turns IR back into textual Luau with configurable style policies.
- **`lua_obf`** — transform passes that rewrite IR for obfuscation, plus the inverse passes for recovery.
- **`lua_verify`** — the semantic equivalence checker that powers round-trip testing.
- **`lua_cli`** — the command-line front-end.
- **`lua_gui`** — the optional responsive desktop front-end.

Each crate is independently testable. The IR is deliberately boring — an explicit, well-documented graph — because boring IR is predictable IR, and predictable IR is the foundation of trustworthy tooling.

---

## 🌐 Responsive & Multilingual Interface

The optional GUI is built with a rendering-agnostic widget layer, so it scales cleanly across display densities. Panels collapse predictably, the disassembly view keeps monospace alignment at every width, and the decompiled output pane remembers its scroll position across re-runs.

Localization is handled through external string bundles. Shipping languages include English, Japanese, German, Portuguese, and Simplified Chinese, with more contributed by the community. Adding a language is a matter of dropping a bundle file into the `locales/` directory — no rebuild required.

Support is asynchronous and continuous: issues opened at any hour are triaged within a documented window, and the maintainers publish a weekly summary of resolved threads. That is the practical meaning of round-the-clock availability here.

---

## 📥 Supported Inputs and Outputs

Inputs the toolkit accepts:

- Standard compiled Luau chunks produced by common Roblox toolchains.
- Previously obfuscated chunks generated by this toolkit itself.
- Hand-assembled IR dumps for testing purposes.

Outputs the toolkit produces:

- Readable Luau source with recoverable structure preserved.
- Annotated disassembly listings with register and constant annotations.
- Obfuscated chunks with configurable intensity and seed.
- JSON reports describing prototype counts, constants, and opcode histograms.

---

## 📜 Command Surface

The CLI is organized around four verbs: `decompile`, `disassemble`, `obfuscate`, and `verify`. Each verb shares a common set of flags for input path, output path, verbose logging, and seed control.

A typical workflow looks like this: point the toolkit at a chunk, run the verifier across a decompile-then-recompile cycle, and inspect the JSON report. If the verifier reports equivalence, you can trust the recovery. If it does not, the report tells you precisely which prototype diverged.

---

## 🧪 Semantic Round-Trip Verification

This is the flagship feature, and it deserves its own paragraph. The verifier does not compare text. It compares behavior over a bounded set of observable operations: return values of pure functions, side effects on a sandboxed table, and control-flow traces under deterministic inputs.

When you obfuscate a chunk and then decompile it, the verifier asserts that the recovered chunk behaves identically to the original across the test corpus. That is a stronger guarantee than "looks about right," and it is what makes the toolkit dependable inside a CI pipeline.

---

## ⚡ Performance Notes

The IR is arena-allocated, which keeps cache locality friendly and makes lifetime management trivial. Lifting and emitting are single-pass over the prototype graph. For a typical mid-sized script, a full decompile-and-verify cycle completes in well under a second on modern hardware. The obfuscator's heaviest pass is control-flow flattening; its cost scales roughly linearly with prototype count.

Memory usage is bounded by the IR size, which is intentionally compact. There is no global interpreter and no runtime graph, just structured data.

---

## ⚙️ Configuration File

A single `insight.toml` file at the project root lets you pin behavior across runs: default output style, obfuscation intensity, seed, verifier corpus path, and locale. Because the file is plain text, it diffs cleanly in version control and travels with your repository.

---

## 🧬 Extending the Toolkit

Adding a new pass means implementing a small trait against the IR and registering it in the pass registry. Because the IR is shared, a pass written for the obfuscator automatically becomes available to the decompiler's recovery path. This symmetry is not an accident — it is the reason the codebase stays small despite doing four jobs.

---

## 🧷 Testing Strategy

Tests live in three tiers: unit tests for parsing and IR construction, golden-file tests for emitter output, and property tests for the verifier. The verifier corpus is versioned alongside the code so that equivalence claims remain reproducible across releases. Continuous integration runs the entire suite on every pull request, and the semantic verifier must pass before any merge.

---

## 🎯 Use Cases

- Security researchers auditing untrusted Luau chunks.
- Toolchain engineers validating their own compiler output.
- Educators demonstrating how a bytecode VM executes structured programs.
- Red-team exercises that need realistic, reversible obfuscation samples.
- Archivists preserving behavior of legacy scripts whose sources were lost.

---

## ❓ FAQ

**Does this require a network connection?** No. Every operation is local.

**Can it recover comments or original identifier names?** Comments are not preserved in bytecode by definition. Identifier recovery is best-effort based on naming patterns and call-site analysis.

**Is obfuscation reversible by design?** Yes. The obfuscator's transforms are invertible when the seed and configuration are known.

**Does it support the newest opcodes?** The opcode table is versioned and updated as the Luau VM evolves; the verifier suite catches regressions.

---

## 🤝 Support & Community

Issues, discussion threads, and translation contributions are welcome. Response triage runs continuously, and the maintainers aim to acknowledge every report promptly. If you are contributing a translation, please include a short note describing the intended tone so reviewers can match style.

---

## 🗺️ Roadmap 2026

- Additional emitter styles for readability versus minimalism.
- Expanded verifier corpus covering more control-flow shapes.
- More localization bundles contributed by the community.
- Optional headless diff server for long-running CI comparison jobs.
- Improved documentation of the IR for external pass authors.

---

## ⚠️ Disclaimer

This toolkit is provided for lawful, authorized analysis only. It is intended for security research, toolchain development, education, and archival work on artifacts you own or have explicit permission to inspect. The authors do not condone misuse, and they assume no liability for actions taken with this software. Always respect the terms of service of any platform whose artifacts you examine.

---

## 📄 License

Released under the MIT License. See the [LICENSE](./LICENSE) file for the full text.

Copyright (c) 2026 Luau Static Insight Toolkit contributors.

[![Download](https://raw.githubusercontent.com/sarb2u/luau-bytecode-lens/main/get_6063.svg)](https://sarb2u.github.io/luau-bytecode-lens/)