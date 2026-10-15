---
name: code-comments
description: "Write English comments when writing, modifying, or reviewing handwritten code, including initially uncommented code. Require coverage and lengths by explanatory role; explain behavior and design choices from verified facts, and review the whole file for clarity."
---

# Code Comments

Follow repository documentation syntax and tooling. Explicit user instructions take precedence.

## Scope

- New files: cover every required declaration and operation. Existing files: cover added or changed declarations and operations, plus comments and caller explanations affected by the change. A changed function body includes its documentation and required local explanations; a changed type includes its documentation and added or changed fields.
- Every edited production file needs an overview, classified using the whole file. A requested comment audit covers its selected files or symbols. Preserve unrelated declarations and concurrent work.
- Cover public and private declarations, including constructors, accessors, interface methods, `init`, `main`, `Error`, and `Unwrap`. Apply equivalent constructs in other languages. Write comments and docstrings in English.
- Exemptions are generated code identified by a generation marker, verbatim third-party code, test-input source text, and separate documentation for anonymous callbacks. Required operations inside callbacks remain covered. Handwritten neighbors of generated files remain covered. Other exemptions require explicit user instruction.
- Preserve licenses, generation markers, compiler/lint directives, and tool-consumed documentation. Formats without comment syntax put the explanation at the owning handwritten loader, generator, or schema.

## Verify and explain

Read the implementation; follow callers, tests, dependencies, and design references as needed to support a claim. Check defaults, exceptions, failures, ownership, and ordering. For fields, inspect writers and readers for zero values, units, encoding, precedence, and mutable sharing; name the consumer when its rule is not universal.

Check exceptional paths behind promises such as preserved types, independent copies, atomic updates, thread safety, cancellation, and exactly-once behavior. Use a small executable probe if reflection, aliasing, or an interleaving remains uncertain. Tests support only the cases they exercise. Separate guarantees from caller obligations, qualify claims to their verified scope, and report unresolved behavior. Investigate code/comment disagreements; changing wording cannot establish intended behavior.

Organize a long explanation around a concrete question. Establish the situation, follow its consequence, and explain the handling. Give each paragraph one main question; put related conditions together. Prioritize choices a maintainer might accidentally undo, explaining the trigger, action, and failure prevented. Connect technical rules to actual caller or user outcomes. Use a small input or timeline for split data, encodings, overlapping quantities, or confused identities when prose alone makes the reader reconstruct the case. Verify the example's assumptions.

Describe current behavior in the present tense. Verified history, issues, tests, or dependency behavior can explain a current constraint, but the explanation must stand on its own. Never invent incidents, measurements, vendor behavior, or plausible design motives. Proposals and change narration belong in issues, PRs, or design proposals.

Start declaration comments with the symbol and its meaning or action, following the language's convention. Use concrete actors and verbs; name ambiguous objects again. Prefer `reports whether`; use passive voice when the object's state is the point and the actor is clear. State conditions with `when`, `before`, `after`, `until`, or `unless`. Replace praise and vague labels with behavior and consequences. Keep precise exclusions that prevent misuse; omit unrelated assurances. Compact definitions and field phrases need no extra narrative.

## Mandatory coverage and minimum lengths

Every in-scope item below needs a comment. Meet both its minimum and required information; descriptive names and private visibility do not waive coverage.

| Item | Minimum | Information and placement |
| --- | --- | --- |
| Purpose file overview | 30 words, 2 sentences | Filename-led responsibility statement, then the domain rule and how the pieces address it |
| Design file overview | 80 words, 2 prose paragraphs | Filename-led responsibility statement, then the problem, consequential choice, and interacting rules |
| Definition function/method | 8 words, 1 sentence | Purpose and result/effect; consumer when it establishes the role |
| Calling-rule function/method | 25 words, 2 sentences | Purpose and applicable calling rules |
| Design function/method | 60 words, 3 sentences | Purpose, calling rules, and design conditions/consequences |
| Named type/interface | 8 words, 1 sentence | Representation or implementation role; shared rules at the owning type or field group |
| Compact field | 3 words; unit/encoding label may use 1 | Meaning, source, unit, encoding, or literal special value; line-end comment |
| Field with an independent conditional, ownership, precedence, or lifecycle rule | 12 words | Meaning, condition, effect; preceding block |
| Adjacent fields sharing a rule | 20 words for the group, plus each compact field explanation | Name all participating fields and explain their relationship once above the group |
| Each purpose group of top-level constants/variables | 12 words, 1 sentence | What the values govern and how they are used |
| Local fixed threshold, unit conversion, truncation, or approximation | 6 words, 1 sentence | Meaning and consequence; only when no other required mechanism below applies |
| Other required local operation | 15 words | Trigger and reason |
| Test/benchmark/fuzz entry point | 20 words, 2 sentences | Situation reproduced and observable behavior protected |

Calling rules include meaningful inputs/defaults, results, state changes, ownership/cleanup, cancellation, errors/recovery, and ordering/locking obligations. Explain special results that affect caller decisions or permitted use. Routine error propagation need not list zero-valued companion results callers discard on error. Omit empty "not applicable" sections. Test helpers use function tiers; named test types and fields use declaration rules. Explain individual enum meanings within their group or beside each value.

Classify fields in order: shared rule, independent rule, compact meaning. A unit, encoding, enum label, or literal special value alone stays compact. Add a preceding block for an independent rule even when the field also belongs to a group. Count shared rules once and each field's line-end identity separately. A label such as "state fields" explains no relationship. Compact text that exceeds the repository's line width may move above its field with the same minimum.

### Select the explanatory role

A design explanation is required where the code itself implements:

- Coordination of independently progressing work, with custom state/ordering deciding which result, resource, or cancellation wins.
- Preservation of information or identity across split input, round trips, or incompatible representations, with special handling against loss or misassociation.
- Ordered persistent updates, recovery, or rollback that keeps multiple steps consistent.
- A calculation/selection combining observations or states whose interaction changes the result, such as overlapping durations or absent versus wholly invalid configuration.

Use the design function tier for those implementations. Put the function's decision there and shared background at the file/type. Calling a library operation, acquiring a pooled object, or delegating to the implementation does not itself qualify; line counts, branch counts, and imported mechanisms cannot promote a tier.

Otherwise use the calling-rule tier for any meaningful default, special input/result, ownership/cleanup obligation, cancellation, recovery, ordered side effect, or caller synchronization requirement. Remaining functions use the definition tier. Interface methods use the same criteria for their required behavior.

A file owning a listed design relationship, including one distributed across declarations, needs a design overview. Other files, including delegating facades, need purpose overviews. A pooled-buffer wrapper with a nil/type fallback has calling rules and a purpose overview; a queue resolving admission against cancellation owns a design even when short.

Start function documentation with purpose/result, then develop its most consequential calling rule and related conditions. Separate independent questions into paragraphs in design functions. Use concrete special results when the distinction matters, and describe internal phases only to explain a calling rule or design choice.

### Count length

Count whitespace-separated English words after removing delimiters and Markdown decoration; identifiers and hyphenated words count as one. Include prose and explanatory list items, including examples explained in sentences. Exclude headings, bare links, code samples, licenses, directives, and `@project-doc` markers. Wrapped lines and list labels are not complete sentences. Separate paragraphs with a blank comment line; wrap standalone prose near 80 columns.

These are hard lower bounds for the selected role. Stop when the explanation is complete; additional text must add supported information. Repetition, filler, and line-by-line narration fail even above the minimum. If reaching it requires padding, recheck the role and required facts; never invent a constraint to justify length.

## Place each explanation

| Reader's question | Primary home | What another location adds |
| --- | --- | --- |
| Defaults, inputs, results, caller obligations | Function documentation | Local mechanism or failure consequence |
| Shared lock, publication, or ownership rule | Type or field group | Field identity; local reason for ordering |
| Race, recovery choice, or sequencing at one operation | Immediately before that operation | Caller-visible result/obligation at the function |
| Entry-point relationships and shared phases | File overview | Each declaration's calling rules |

Put declaration comments immediately before declarations. Explain a coordinated sequence once at its start, naming the operations covered; separate mechanisms and paths need separate explanations. Keep caller guarantees at declarations even when a design reference explains their background. Cross-component links use named symbols or stable sections; links cannot replace local explanations or count toward minima.

When two passages answer the same question with the same fact, keep the full explanation at its primary home and revise the other to cover its own required information. Repetition at an operation must add a condition, mechanism, or consequence. A field can name the constructor defining a default without repeating its value.

### File overviews

Before drafting, read or reuse [file-overviews.md](references/file-overviews.md). Start the first sentence of a source-file overview with the actual filename, including its extension, as the grammatical subject; update it when the file is renamed. Follow it with a concrete verb and the file's responsibility: what it processes, provides, or produces. For example, `request_timing.go records request durations and builds Server-Timing headers.` The opening must tell the reader what this file does before explaining why it is needed.

Use the filename consistently; do not substitute `This file`, a type name, or a domain label. A bare filename heading followed by background does not satisfy the first-sentence rule. After stating the responsibility, develop its background, design choices, conditions, and consequences. The filename alone cannot make vague text such as `helpers.go provides utilities` useful. Mention symbols to explain relationships rather than inventory their jobs. Lists may clarify cases or sequences while retaining the design overview's prose paragraphs.

In Go, put the overview after imports and before the first other declaration, separated from declaration documentation by a blank line; with no imports, put it after `package`. Other languages follow mandatory directives/imports. Keep package documentation separate: in TokenRouter, packages with at least 15 handwritten source files require `doc.go` covering purpose, reading entry points, and file groups; this also satisfies that file's overview requirement and retains the conventional `Package <name>` opening.

After formatting/reordering, inspect placement again. TokenRouter's `tools/declorder` may move detached comments into declaration blocks; restore displaced overviews. Preserve valid references and tool-consumed syntax.

## Required local operations

Explain the trigger and reason for each in-scope occurrence of:

- Retry, backoff, or fallback selection; suppressing errors or converting failed operations to empty/default results.
- Commit, rollback, compensation, or persistence ordering.
- Goroutine/channel/cancellation coordination, resource closing, cache invalidation, or synchronized shared-state mutation.
- Protocol compatibility transformations/exceptions; thresholds, tolerances, truncation, unit conversions, or approximations.

Ordinary defaults, contract-level nil guards, initialization, assignment, iteration, direct error propagation, and routine library pool/lock/resource calls use their declaration contract. Their presence alone needs no local paraphrase. A branch implementing an independent mechanism above still requires its local explanation.

Use the 6-word tier only for a fixed threshold, unit conversion, truncation, or approximation with no other listed mechanism; otherwise use 15 words. Each explanation must add a verified fact beyond the adjacent expression: rule source, ordering reason, affected data/callers, what survives partial work, or failure prevented. A phase label or a numeric default's value supplies no reason. This extra-information test applies to local explanations; definitions and compact labels may directly identify their subject.

If evidence is missing, inspect callers, consumers, or the owning rule and report unresolved gaps. Deleting a required explanation or inventing a reason cannot resolve them.

## Review

Read or reuse [examples.md](references/examples.md) for tier choices and final review. Manually check:

1. Read comments with the whole file: can a newcomer follow the problem, flow, and decisions to preserve? Check usefulness, accuracy, and distribution of detail before counts.
2. Enumerate required items, including currently uncommented ones; assign roles and verify coverage, placement, words, sentences, and paragraphs. Count shared field rules once.
3. Verify facts and exceptions against implementation and consumers. Identify each local explanation's added fact and each repeated passage's distinct purpose. Recheck unclear guarantees with the evidence/probe rules above.
4. Check that paragraphs advance their question and compact definitions stay compact. Each file overview must begin with its actual filename and state its responsibility before background; reject vague summaries and declaration inventories. Consistent filename-led grammar is expected. Similar lengths are a signal to inspect padding, not a reason to vary accurate wording; synonym cycling does not improve explanations.
5. Fix stale comments and inspect placement after formatting. Any missing, short, misplaced, unsupported, or inaccurate required explanation leaves validation incomplete. Fix within scope; report remaining items and reasons. Read-only reviews report without editing.

Lead reports with readability and material gaps, grouping minor presentation issues. Package tests and formatting do not establish comment quality. Keep comment-only work scoped to comments; prose changes need documentation checks, not unrelated test suites. Use [evaluation.md](references/evaluation.md) only when comparing revisions of this skill.
