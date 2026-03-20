# CONDITIONAL: Payments

Only load if payment dependencies detected (stripe, paddle, lemonsqueezy, paypal).

| Check | What to search for |
|-------|-------------------|
| Webhook signature verification | Search for `constructEvent`, `webhook_construct_event`, `Webhook.construct_event`. If Stripe is in deps but no webhook verification found: HIGH |
| Client-side amount | Search for amount/price in frontend POST requests to payment endpoints. Amount must be set server-side |
| Idempotency keys | Search for `idempotencyKey`, `Idempotency-Key` in payment creation. Missing = potential double charges |
