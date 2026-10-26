---
name: aaj-kya-bana
description: Runs the Aaj Kya Bana household loop for an Indian home with a cook. It plans meals, fills the grocery cart on Swiggy Instamart or Zepto before the cook comes, writes the cook card in the cook's language, plans Zomato takeout when the cook is off, and records orders and pantry. Use whenever the person mentions meal planning, the week's menu, "aaj kya banega", the cook's schedule or day off, a grocery cart, Instamart, Zepto, Zomato or Swiggy ordering, syncing orders, the pantry, or booking cleaning help, or when a scheduled run or Aaj Kya Bana trigger wakes you to work through due tasks.
---

# Aaj Kya Bana

Aaj Kya Bana holds the household's meal plan, cook timings, pantry and order ledger. You do the work in the vendor apps through their own MCP servers, signed in as the person. Aaj Kya Bana never orders or pays, and neither do you unless the person is in the conversation and asks.

Entity shapes (Household, Meal, Task, Cart, TakeoutPlan, Order, CookCard) are in the Aaj Kya Bana tool schemas. Money is integer paise. Dates are `YYYY-MM-DD` and times `HH:MM` in the household's timezone.

## Is the person here?

Decide this first. It sets what you may do.

- **Present**: the person is chatting with you now and can answer.
- **Absent**: a schedule, routine, cron job, API trigger or webhook started you, or the prompt says it is unattended. Treat a run as absent when unsure.

Absent runs fill carts and record things. They never place an order and never ask questions. When a step needs the person's choice, fail or note it and move on.

## The loop

1. `get_household` once per run. Keep `people`, `diet`, `tastes`, `jobs`, `apps`, `staples`, `groceryApp`, the pantry, and each helper's `language`.
2. `next_tasks` returns due open tasks with `context` (date, slots, meals, visit start). No tasks means nothing to do. Say so in one line and stop.
3. `claim_task` each task before touching a vendor. If the claim errors, another agent holds it. Skip it.
4. Do the task by `kind`:
   - `fill_cart`: build the shopping list, fill the vendor cart (below), then `complete_task` with the Cart and `set_cook_card` for that helper and date.
   - `plan_takeout`: search the food app, `complete_task` with a TakeoutPlan of two or three suggestions. Build no food cart unless the person asks.
   - `check_pantry`: `get_pantry_check`, look at every photo, `update_pantry` with level, `amount` and `confidence` for each item you can see (out = none or empty container; low = under 3 days of planned meals, or under a quarter of capacity for a staple). Then `complete_task` with `{ changed, unreadable }`. Never guess an item you can't see.
   - `sync_orders`: see step 5, then `complete_task` with `{ synced: <number of orders you recorded> }`.
5. Read each vendor's file first (links under "Filling the cart"), then `get_orders` from each vendor you are connected to. For Instamart pass `orderType: "INSTAMART"`; the default returns nothing. Then `record_orders` with what they show, including each grocery order's `items` (plain ingredient name, product, pack, count, whole-line amount, price) from the vendor's order detail (recording twice is a no-op), and `update_pantry` to `have` for groceries that arrived.
6. If the vendor errors or nothing usable comes back, `fail_task` with a reason the person can act on ("Instamart doesn't deliver to the saved address"). If you must stop midway, `release_task`. Never leave a task claimed.

`log_activity` covers anything the other tools don't record, such as a cart you filled in a second app.

## Building the shopping list

- Start from each meal's `dishes`. Servings are `meal.servings`, or `household.people` when that is null. Scale every recipe quantity to that number.
- Respect `household.diet`. Don't add eggs for a veg home.
- Never buy, plan or order anything in `tastes.avoid`. Follow `tastes.notes`.
- Do only the jobs in `household.jobs`. Skip a task whose job isn't enabled and say so.
- Never buy a staple for a recipe. Skip pantry items at `have`. Buy items at `low` or `out`, staples included, as restock lines (`reason: "restock"`). Also restock any item whose `estimate.runsOutOn` is on or before this cart's delivery plus 2 days. Size restocks to about two weeks of `estimate.usePerDay`, capped at the spot's `capacity`, preferring the product and pack the household bought most in the last 90 days. An ingredient the pantry doesn't list is needed: buy it unless it is a staple.
- Merge the same ingredient across dishes and record every dish it serves in `forDishes`.
- Pick the pack, or count of packs, that covers the recipe quantity with the least left over. 600 g of tomatoes is one 500 g pack plus one 250 g pack, or a 1 kg pack if that is cheaper, not three 500 g packs.
- Prices come only from the vendor's tool results. A price you didn't see is `null`. Never estimate one.

## Filling the cart

Read the vendor file before you call its tools. Each has the tool names, the cart semantics and the confirmation rule.

- Swiggy Instamart: [references/swiggy-instamart.md](references/swiggy-instamart.md)
- Zepto: [references/zepto.md](references/zepto.md)
- Zomato (food only): [references/zomato.md](references/zomato.md)

Use `household.groceryApp` first. Items it doesn't have get `product: null` in the Cart. Then search the other grocery app for those items only. If you find them there, fill a second cart, `log_activity` for it, and name the app in `Cart.note` ("Curry leaves not on Instamart. Added to Zepto cart"). If neither app has it, say so in the note.

The Cart you send to `complete_task` lists every item: its `name`, the scaled `quantity`, `forDishes`, the vendor's `product` and `pack`, `count`, and `pricePaise` from the vendor. `totalPaise` is the vendor cart total, or `null` if you didn't see one.

## Placing an order

Only when the person is present and asks you to place it. Then follow the vendor's confirmation rule exactly, as the vendor file says:

1. Read the cart back from the vendor.
2. Show the items, the total and the full delivery address. Ask one question: "Order 14 items for ₹612 to Flat 302, Jubilee Hills?"
3. Wait for an explicit yes. "ok", "go ahead", "place it" count. Silence, an earlier instruction, or "sure, also add milk" do not.
4. Call checkout once. If it times out or errors, check the vendor's order list before anything else. Never retry blind. A second checkout can be a second order.
5. `record_orders` with the vendor's order id.

## The cook card

One card per helper per day, through `set_cook_card` with `helperId` (it is in the task key `fill_cart:<helperId>:...`), the date, and the helper's `language`.

- Write in the helper's `language` (`en`, `hi`, `hi-Latn` Hinglish, `mr`, `te` or `kn`) and its script, never in the app's language.
- Greet by name with that language's respect form. One line per meal with the slot and dishes, then what's soaked, in the fridge, or arriving.
- Dish names as homes say them: "rajma chawal", "bhindi". Never translate a dish into English.
- At most 6 lines, 60 words, 400 characters. It is read, or played as a voice note, in the kitchen.

Read [references/cook-languages.md](references/cook-languages.md) before your first card in a run. It has the respect forms, one example per language, and which language your notes to the person use.

## How you write to the person

- Plain, specific, local. Concrete numbers: "14 items, ₹612, arrives by 7:10".
- Lead with what happened: "Cart's ready on Instamart. 14 items, ₹612. Place it before Sunita comes at 8:00."
- Name what's missing and what you did about it: "Bhindi isn't on Instamart near you. Removed from the cart."
- No emoji, no exclamation marks, no "yummy", no "Oops". Say "Place order", not "Submit".
- Activity summaries, `Cart.note` and `fail_task` reasons go in the person's app language (see the cook-languages reference).

## Other requests

- **Plan the week**: read `get_plan` for the coming 7 days and `get_household`. Propose dishes for the slots each cook visit `covers`, matching `diet` and drawn from `tastes.cuisines` and `tastes.likes`, never `tastes.dislikes`, then `set_meals` once the person agrees. In an absent run, fill only empty slots.
- **Cook is off**: call `set_absence` for that helper and date (or the person marks it in the app). That creates `plan_takeout` tasks and skips the cart. Run those.
- **Cleaning**: you can't book Snabbit, Pronto or Urban Company InstaHelp. Tell the person the Aaj Kya Bana app opens their cleaning app in one tap.
- **Scheduling this loop**: [references/scheduling.md](references/scheduling.md).
