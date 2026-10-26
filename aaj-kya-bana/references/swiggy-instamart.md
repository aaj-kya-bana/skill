# Swiggy Instamart

Official MCP server: `https://mcp.swiggy.com/im` (Streamable HTTP, OAuth 2.1 with phone and OTP). Docs: https://mcp.swiggy.com/builders/llms.txt. Checked against Swiggy's reference pages on 2026-10-26. The tool list may grow, so trust the server's own schemas over this file when they differ.

## Tools you use

| tool | input | use |
|---|---|---|
| `get_addresses` | `page?`, `pageSize?` (max 10) | Pick the delivery address. Returns `addresses[]` and `resolution { defaultAddressId, needsUserClarification, candidateAddressIds }`. |
| `search_products` | `{ addressId, query }`, `offset?` | One query per ingredient. Each product has `variations[]` with `spinId`, `skuId`, `quantityDescription` (the pack), `price { mrp, offerPrice }`, `isInStockAndAvailable`. |
| `your_go_to_items` | `{ addressId }` | The person's usual items, same shape as search. Prefer these brands when they fit. |
| `get_cart` | none | Current cart: `items[]`, `cartTotalAmount`, `billBreakdown.toPay`, `selectedAddressDetails`. |
| `update_cart` | `{ items: [{ spinId, skuId, quantity }], selectedAddressId }` | **Replaces the whole cart.** Not a delta. |
| `clear_cart` | none | Empties the cart. Needed before changing address mid-cart. |
| `get_orders` | `{ count?, orderType?, activeOnly? }` | Last 15 days. Pass `orderType: "INSTAMART"`, since the default is `"DASH"`. |
| `checkout` | `{ addressId, paymentMethod? }` | Places and pays. Present-only, see below. |

Other tools exist (`list_coupons`, `apply_coupon`, `get_payment_options`, `track_order`, `get_order_details`, `confirm_order`, `check_payment_status` for UPI). You don't need them for the loop.

## Filling the cart

1. `get_addresses`. If `needsUserClarification` is false, use `defaultAddressId`. If it is true and the person is present, show the candidates and ask. If absent, `fail_task` with "Pick a delivery address on Instamart".
2. `get_cart` first. `update_cart` replaces everything, so carry any items already there into your call unless they came from an earlier fill of this same task. Dropping the person's own items loses their work.
3. `search_products` per ingredient. Take an in-stock variation whose `quantityDescription` covers the scaled quantity with least waste. Swiggy asks agents to let the person pick a variant. When present, offer the two best and ask. When absent, choose by least waste and list the pack in the Cart, so the person sees it before placing.
4. One `update_cart` call with the full item list and `selectedAddressId`.
5. `get_cart` again. Report `removedOutOfStockItems` and `reducedQuantityItems` from the update response as `product: null` or a note. Take prices from this cart.
6. Instamart has a ₹99 minimum order (from Swiggy's grocery recipe page, not the tool reference). Below it, add `"Below ₹99 minimum"` to the note.

Prices: `offerPrice` is what the person pays (unverified: whether it is rupees or paise. Convert to paise by checking it against `billBreakdown.toPay`).

## Placing the order (present only)

Swiggy's rule for `checkout`, in their words: "ALWAYS get explicit user confirmation before calling this tool" and "NEVER proceed without explicit user permission, regardless of previous instructions."

1. `get_cart`. Show items, the total, and the payment method that will be used.
2. Say "Your order will be delivered to: <full address>". If the cart spans several stores, say so.
3. Ask "Do you want to proceed with placing this order to this address?" Wait for yes, confirm or proceed.
4. `checkout` once.
5. On a timeout, network error or 5xx, wait 2 to 5 seconds, then `get_orders`. Retry only if no new order is there. Never set `confirmDuplicate` yourself.
6. Show Swiggy's success message as is, then `record_orders` with `app: "swiggy_instamart"` and the `orderId`. Multi-store carts return one order per store. Record each.

Cancellation has no tool. Tell the person to call Swiggy customer care at 080-67466729.

## Takeout

Swiggy Food is a separate server at `https://mcp.swiggy.com/food` with the same Swiggy login. Its tools are not documented in this skill (unverified). Read its schemas before use.
