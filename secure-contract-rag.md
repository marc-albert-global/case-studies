# Case Study — Secure, Local-First Contract RAG

> **Status:** Private working repository.
> This write-up describes my own design and implementation. No third-party or confidential data is involved.

## The problem

Retrieval-augmented generation over sensitive documents (employment contracts, legal agreements) runs straight into a privacy wall: to make documents searchable you normally embed them, and embedding usually means **sending the text to a cloud API**. For a document class full of names, salaries, and personal data, that's a non-starter in a lot of real environments.

I wanted to prove out a RAG system that a privacy-conscious organization could actually run: **useful retrieval and Q&A over legal documents, with data sovereignty as the default, not an afterthought.**

## The approach

The system is **air-gapped by default** and treats cloud as strictly opt-in:

1. **PII masking before embedding** — personal data is detected and masked *before* any text leaves the ingestion stage, so even the vector representation never carries raw PII.
2. **Local-first inference & storage** — a local model runtime and a local vector store mean the default configuration sends nothing to any external service. Cloud providers can be enabled explicitly, but they're off out of the box.
3. **Role-based access at retrieval time** — retrieval is filtered by the requesting user's role, so the answer is constrained by *who's asking*, not just what's in the corpus.
4. **Legal-aware chunking** — documents are split along their actual structure (clauses, sections) rather than naive fixed windows, which materially improves retrieval quality on this document class.
5. **A routed query graph** — queries flow through a small state machine that classifies the request and routes it to the right retrieval and answering path.

## Architecture (described)

```
document ─► [ PII masking ] ─► [ legal-aware chunking ] ─► [ local embeddings ] ─► [ local vector store ]
                                                                                         │
user query ─► [ classifier / router ] ─► [ role-filtered retrieval ] ◄────────────────────┘
                                                  │
                                                  ▼
                                    [ grounded answer + sources ]
```

Built as a modular Python system: separate ingestion, storage, retrieval, and inference layers; configuration-driven so local vs. cloud is a setting, not a rewrite; with a test suite covering the parts that matter for trust — masking, access control, retrieval, and routing.

## Outcome

A working, packaged system that demonstrates the thesis: you can have genuinely useful RAG over sensitive documents **without** giving up data sovereignty. The privacy properties (mask-before-embed, air-gapped default, role-filtered retrieval) are enforced by the architecture rather than left to operator discipline.

## Why the code stays private

It's a personal working repository I'm continuing to develop, and the contract-handling logic is something I'd rather discuss than publish. Happy to walk through the masking pipeline, the access-control model, and the chunking strategy in a conversation.
