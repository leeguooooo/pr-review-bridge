# Building a review service that respects developer attention and model budgets

## The original mismatch

A review that starts immediately but finishes after the PR merges cannot inform that merge. Polling with an LLM and repeatedly reviewing the same source make the mismatch expensive. An error banner is also not a code defect.

## Separate five decisions

1. **Intent:** the author declares none, spark or deep, with a reason.
2. **Scheduling:** coalesce events, wait 60 seconds for a stable head, and cancel obsolete work.
3. **Reasoning:** give a bounded diff and request only missing snippets. The author chooses the review level; the service does not classify complexity or upgrade models.
4. **Reuse:** key source analysis by repository, head, merge-base and engine policy; key human acknowledgement separately by the current PR revision and declaration.
5. **Delivery:** publish high-confidence defects, edit one status comment, privately notify the author, and optionally wake the exact development session.

## Failure is a separate outcome

Quota exhaustion, malformed model output and timeouts are unavailable/incomplete, never a clean pass. Manual review can continue during an outage. Previously reported defects still need resolution or an evidenced false-positive decision.

## What caches can and cannot do

A complete local cache hit makes zero model calls. Provider prompt caching may reduce repeated prefix work, but does not mean an entire project is remembered for free. The cache pins the whole source head to avoid reusing a result when an unseen dependency changed. It does not certify an untested merge result.

## Validation

Use unit tests for queue migration, cancellation, declaration parsing, identity isolation and notification deduplication. Use controlled real runs to verify SDK/browser behavior. Distinguish tested code from a working live provider: a quota refusal can prevent end-to-end verification even when all unit tests pass.

## Next: chatgpt-use

The intended adapter is a read-only `ask` transport, not an autonomous code-editing loop. Requirements are strict structured output, durable submission receipts, resumable reads and confirmed cancellation. See upstream [structured output issue](https://github.com/leeguooooo/chatgpt-use/issues/3) and [request lifecycle issue](https://github.com/leeguooooo/chatgpt-use/issues/4). This adapter is not yet shipped here.

Suggested talk/demo: declaration → coalesced queue → cache hit with zero model calls → bounded review → author notification → exact-session wakeup → human decision. Demonstrate an unavailable provider too; do not present only the happy path.
