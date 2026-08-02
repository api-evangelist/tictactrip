---
name: Book a trip end to end
description: Turn a selected Tictactrip itinerary into a confirmed, ticketed booking via cart, order and book.
api: openapi/tictactrip-openapi-original.json
operations: [CreateCart, UpdateCart, GetCart, CreateOrder, CreateBook, GetOrder, DownloadTicket, GetDiscountCards]
---

# Book a trip end to end

Use this skill to convert a chosen search result (see `tictactrip-search-trips.md`) into a booked e-ticket.

## Auth
`Authorization: Bearer <JWT>` with the `API_BOOK_PARTNER` role.

## Steps
1. **(Optional) List discount cards.** `GetDiscountCards` (`GET /booking/v3/discountCards`) to attach a railcard to a passenger.
2. **Create a cart.** `CreateCart` (`POST /booking/v3/carts`) with the selected trip(s), passengers and customer. Returns `201` with a `cartId`.
3. **Refine the cart.** `UpdateCart` (`PATCH /booking/v3/carts/{cartId}`) to set passenger details / options; re-read with `GetCart` (`GET /booking/v3/carts/{cartId}`).
4. **Create an order.** `CreateOrder` (`POST /booking/v3/orders`) from the cart. Returns `201` with an `orderId`.
5. **Book.** `CreateBook` (`POST /booking/v3/orders/{orderId}/book`) — this is asynchronous and returns `202 Accepted`. Poll `GetOrder` (`GET /booking/v3/orders/{orderId}`) until the order/ticket status settles.
6. **Fetch the ticket.** `DownloadTicket` (`GET /booking/v3/orders/{orderId}/ticket`) once booked.

## Rules
- No idempotency key exists; because `CreateOrder` / `CreateBook` are state-changing, do NOT blind-retry a request whose outcome is unknown — reconcile with `GetOrder` first.
- Booking is async (`202`); treat success as "accepted", confirm via `GetOrder`.
- Errors are `{ errorKey, errorMessage }` JSON; `404` = unknown cart/order, `400` = invalid body, `500` = retry after reconciling.
