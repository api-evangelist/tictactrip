---
name: Cancel a booked ticket
description: Check cancellation conditions for a Tictactrip order ticket and cancel it.
api: openapi/tictactrip-openapi-original.json
operations: [PartnerCancellationConditions, PartnerCancelBooking, GetOrder]
---

# Cancel a booked ticket

Use this skill to cancel a ticket within a confirmed order.

## Auth
`Authorization: Bearer <JWT>` with the `API_BOOK_PARTNER` role. You may only act on tickets belonging to your partner scope (else `403`).

## Steps
1. **Read cancellation conditions.** `PartnerCancellationConditions` (`GET /booking/v3/orderTickets/{orderTicketId}/cancellation/conditions`) — returns whether the ticket is cancellable and any refund/fee terms.
2. **Cancel.** `PartnerCancelBooking` (`PUT /booking/v3/orderTickets/{orderTicketId}/cancellation`) to perform the cancellation.
3. **Confirm.** Re-read the parent order with `GetOrder` (`GET /booking/v3/orders/{orderId}`) to verify the new ticket status.

## Rules
- `403` = the ticket is not in your scope; `404` = unknown ticket; `409` = the ticket is not in a cancellable state (check conditions first).
- Errors are `{ errorKey, errorMessage }` JSON. Always call the conditions endpoint before cancelling to avoid `409`.
