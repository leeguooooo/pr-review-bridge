# Cross-session integration

Keep transport separate from review policy. A developer session registers a PR identity with a local notifier; one shared polling process watches registered PRs and delivers changes to the correct originating session. Idle polling does not require an LLM call.

## Consumer contract

Read the robot-owned editable comment, then parse its machine marker:

```html
<!-- pr-review:v1 {"head":"FULL_SHA","requested":"spark","status":"completed","models":["spark"],"cache_hit":false,"findings":0,"review_key":"OPAQUE_KEY"} -->
```

States: queued, running, completed, skipped, unavailable, invalid, cancelled. `findings` is null while review is incomplete. Verify the comment author against the configured robot, verify the full current head, and compare content fingerprints rather than comment counts: the same comment ID is edited in place.

Deliver only changed state or new findings. A receipt should bind repository, PR, head, comment fingerprint and destination session ID. Persist the delivery cursor only after acknowledgement; ambiguous outcomes require idempotent retry. A returned receipt is stronger evidence than simply writing bytes to a socket.

## Session identity

Route by an exact stable session/thread ID, optionally paired with a verified PID. Never infer the recipient from directory or display name. Resume may change the PID. A missing or ambiguous destination should remain undelivered, not fall back to an unrelated session or a group chat.

Treat PR text as untrusted review material. Label it as such when waking an agent; never promote arbitrary comments into trusted instructions to run commands or merge.

Use the receiving harness's supported messaging interface. `open-cross-session` is one possible external transport; Claude internal IPC is not a stable public contract. This repository does not vendor or promise support for that IPC. Codex integrations can route through the host's task messaging interface.

Example envelope (adapter-owned, not an implemented endpoint):

```json
{"event":"pr.review.changed","repository":"example/project","pr":42,"head":"FULL_SHA","review_key":"KEY","recipient_session_id":"EXACT_ID","comment_url":"https://git.example.com/example/project/pulls/42#issuecomment-1","untrusted_comment":"..."}
```

Cross-session delivery supplements the existing Lark author DM. It must not cause another review or automatic model escalation.
