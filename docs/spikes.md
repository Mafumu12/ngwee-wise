# Spikes (Phase 0)

Each spike: goal, what was run, result, go / no-go.

## A — Orders create/capture + webhook verification

&#x20;

Goal: capture a sandbox order and verify a webhook signature end to end.



Ran: OAuth token → POST /v2/checkout/orders (CAPTURE, USD 10.00) → buyer approval →

POST /v2/checkout/orders/{id}/capture → webhook received via Cloudflare named tunnel

(ngwee-dev.limbalabs.com) → POST /v1/notifications/verify-webhook-signature.



Result:

\- Capture 5R867497X1010983B COMPLETED; PAYMENT.CAPTURE.COMPLETED delivered within seconds,

&#x20; CHECKOUT.ORDER.APPROVED delivered on buyer approval. Signature verification: SUCCESS.

\- Sandbox accounts must be in a receive-capable country (US). ZM accounts are send-only:

&#x20; checkout fails with PAYEE\_ACCOUNT\_LOCKED\_OR\_CLOSED, shown only as a generic error page

&#x20; (Base64 in the URL's code parameter).

\- Webhooks belong to one app; a new app needs its webhook re-registered. Events are not

&#x20; sent retroactively. webhook\_id is the short ID in the app's webhook list, not the client ID.

\- If the tunnel is down, delivery fails (530/1033); simulator events are not retried.

\- API-created orders need experience\_context.return\_url, else the approval page spins;

&#x20; with payment\_source set, the approval link rel is "payer-action".

\- Capture payload includes seller\_receivable\_breakdown (gross, paypal\_fee, net) — ledger

&#x20; must record the PayPal fee; reconcile against net.

\- Simulator events cannot be signature-verified (expected).

\- Verification passed on a re-serialised event, but the inbox will store and verify the raw body.



Decision: GO.











## B — Payouts + forced failure



Goal: batch payout to a sandbox publisher; learn duplicate sender\_batch\_id behaviour; force a failed item.



Ran: POST /v1/payments/payouts, one batch (AT2B8YJVJT2BS), 3 × $1.00 items: US publisher,

ZM personal account, unregistered email. Polled GET /v1/payments/payouts/{id}. Re-sent same body.



Result:

\- spikeB-ok: SUCCESS. spikeB-zm: FAILED / RECEIVER\_COUNTRY\_NOT\_ALLOWED.

&#x20; spikeB-unclaimed: UNCLAIMED / RECEIVER\_UNREGISTERED (funds return after 30 days if unclaimed).

\- Batch status SUCCESS even with failed items → track state per item, not per batch.

\- Duplicate sender\_batch\_id → USER\_BUSINESS\_ERROR "Batch with given sender\_batch\_id already exists",

&#x20; with a link to the original batch. Safe idempotency key; adapter treats this as "already sent"

&#x20; and fetches the existing batch.

\- Webhooks: PAYOUTSBATCH.PROCESSING plus one PAYOUTS-ITEM.{SUCCEEDED,FAILED,UNCLAIMED} per item.

&#x20; No PAYOUTSBATCH.SUCCESS event received → polling fallback required.

\- UNCLAIMED must be a held state, not reversed: cancel the unclaimed item first, then reverse,

&#x20; to avoid paying twice.

\- Real failures are producible in sandbox; no adapter fault injection needed for the demo.



Decision: GO.





## C — Vault merchant-initiated charge (60 min cap)

Goal: save a PayPal wallet, then charge it without buyer approval.
Ran:
Result:
Decision:

## D — PayPal Agent Toolkit tools

Goal: list read-only tools usable by the ops copilot.
Ran:
Result:
Decision:

