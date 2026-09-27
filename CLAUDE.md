# Moonpetal — Claude Code project brief

Moonpetal (moonpetal.eu) is a curated giftware store on Shopify (Basic plan, EUR, Luxembourg). Products are dropshipped/resold from UK wholesaler **Something Different Wholesale (SDW)**. The owner (Loubna) curates; Claude executes. She is fluent in EN/FR/DE/ES/LB/PT/IT and does all copywriting review and localisation herself — Claude drafts, she approves.

This repo is the single durable home for project state: this brief, the decisions log below, and any scripts/data files built along the way. Keep it current — update the decisions log whenever Loubna makes a call or a milestone completes.

## Access
- **Shopify**: use the Shopify MCP connector (verified working). Store: Moonpetal, www.moonpetal.eu, Basic plan, EUR, Luxembourg.
- Custom-app alternative: Shopify Admin → Settings → Apps → Develop apps; scopes: read/write products, inventory, publications, online_store_navigation, content, themes. `write_legal_policies` may not be grantable — legal policy pages are pasted manually by Loubna. GraphQL Admin API 2025-07+.

## Files Loubna provides (uploaded per session, not committed unless she says so)
- `TradeStockFull.txt` — tab-delimited SDW **UK trade file, GBP**. 3,189 products. Columns include Item No, Item Name, Price (GBP trade), RRP (GBP), Barcode, Material, Height/Width/Depth, Individual Weight, Packed Weight, Commodity Code, Country of Origin, Outer (case qty), Discontinued, URL Links (xlarge image), Description.
- `StockLevelsEU-2.csv` (or newer) — **EU feed, EUR**: Item Code, Stock Level, EU Price (real EU wholesale cost). ~3,120 SKUs.
- `book_lover_alcove.csv` — the completed first collection (48 products) with all final fields; reference format.
- `critter_selection.xlsx` — 129-product animal selection. **Round-trip in progress — see decisions log.**

## Non-negotiable content rules (auto-screen EVERY product name + description)
- BLOCKED (never list): Amy Brown artwork; pentagram, pentacle, satan, baphomet, occult, ouija/talking board, tarot, pagan, wicca(n), altar, ritual, witchcraft, sabbat, triple moon, horned god, athame, rune, divination/pendulum.
- REVIEW (surface to Loubna, do not auto-include or auto-reject): "book of spells", spell, magick, grimoire, witch, familiar, coven. Playful/whimsical spellbook giftware is allowed; genuinely occult items are not. Loubna decides edge cases. Match plurals too (witches, spells).
- Profanity in product names: flag for her decision.

## Pricing (EUR)
- **Cost** = `EU Price` from the EU feed. NEVER use the GBP trade Price as cost (it is ~1.25× lower; using it would underprice everything).
- **Retail** = EU RRP × 1.17 (Luxembourg VAT) rounded UP to next €.99.
- The trade file RRP is GBP. Where no EU RRP is known, estimate EU RRP = GBP RRP × 1.30 (median measured uplift), then apply ×1.17 and .99 rounding — but recompute from real EU figures before creating products if available.
- Multipacks/displays sold as singles: unit cost = display cost ÷ units; unit retail from per-unit RRP via the same formula. Split SKUs as `PARENT-SUFFIX`, keep `supplier_sku` = parent for ordering, per-unit weight/dims (NOT the display's), NO barcode on split units (the EAN belongs to the display; duplicated EANs break Google/Amazon).

## Naming & copy
- Whimsical Moonpetal name in `title`; the literal search phrase in `seo.title` + first line of body in `<strong>`. Example: title "Self-Love Book Vase", SEO "Pink Book Shaped Vase | Ceramic Bookshelf Vase | Moonpetal".
- Colour sets share a theme name across product forms. Existing sets: Myths & Legends (green), Story of Serenity (white), Self-Love (pink), Dark Academia (black/spellbook). Pattern: "<Set> <Form>" e.g. "Myths & Legends Storage Box".
- Licensed-artist products: keep artist attribution in the name.
- Voice: warm, flowing, a little wry; SDW-adjacent but original. NEVER copy SDW descriptions (duplicate content). Open on a scene, not specs. 60–120 words. Cross-reference set siblings by name. Specs go in a separate bullet list after `<hr>`: Material, Capacity (if any), Dimensions, Weight, Barcode.
- SEO titles ≤60 chars; meta descriptions ≤160 chars. Loubna has authorised all SEO improvements without asking.
- Product names stay ENGLISH in all future locales; only descriptions get localised (by Loubna, later, via Translate & Adapt).

## Collection architecture (two layers, one product)
1. **Themed worlds** (curated): smart collections with rule TAG EQUALS "<collection name>". Existing: "The Book Lover Alcove" (gid://shopify/Collection/698725597568, handle book-lover-alcove). Planned: "The Cattery" (handle cat-lover-gifts), "The Menagerie" (handle animal-lover-gifts).
2. **Browse categories** (product-type): smart collections with rule PRODUCT_TYPE EQUALS "<type>". Types to standardise: Mugs, Vases, Storage Boxes, Bookmarks, Candles, Oil Burners & Wax Warmers, Keyrings, Jewellery, Cushions & Textiles, Stationery, Trinket Dishes, Doorstops, Tote Bags, Seasonal.
- **FIX REQUIRED**: the 48 live products currently have MATERIAL in productType ("Ceramic", "MDF"...). Batch-update productType to the category noun above; material already lives in the specs bullet list.
- Navigation: main menu = themed worlds; "Shop by" submenu = categories. Handles for categories = the search phrase (e.g. /collections/trinket-dishes).
- 18 legacy collections exist from the old store (Black Like My Soul, Retro Funhouse, Critter Cuddles, Spooky Season, Gift Sets, ...). Decide with Loubna per collection: repurpose or delete. Do not leave three overlapping animal collections.

## Store state
- 48 products live (status ACTIVE), all in The Book Lover Alcove, one 1000×1000 image each from SDW xlarge URLs, full SEO, tags, cost, barcode, weight, HS code, country of origin, live inventory. 4 items at qty 0 (pending SDW restock).
- **NOTHING is published to the Online Store channel yet** (verified 2026-09-27: publishedCount 0, activeCount 48; storefront password protection ON; theme: Dawn). Publish everything in ONE pass only when all collections are built and Loubna says go.
- Location: gid://shopify/Location/115997475200. Vendor: "Moonpetal".
- Tags in use: collection name, set name, "Best Seller", "New In", "Back In Stock", "Out of Stock" (static — storefront sold-out state should come from live inventory; keep the tag only as an internal filter).

## Technical patterns that work
- Create products with `productSet` (one call = product + variant + price + inventoryItem incl. cost/weight/HS code/origin + inventoryQuantities + files + collections + SEO). Batch 4–6 per mutation via aliases. Zero errors across 48 so far.
- Images: pass SDW xlarge URL (`https://www.somethingdifferentwholesale.com/Images/Product/Default/xlarge/<SKU>.jpg`) as `files.originalSource`; Shopify fetches and rehosts. Verify `mediaCount` after each batch.
- SDW listing pages saved as Safari `.webarchive` contain the full post-JS DOM incl. all products and images — parse with plistlib + BeautifulSoup.
- Trade file has control characters (e.g. \x1c) — strip \x00-\x1f before writing to xlsx/Shopify.
- `inventoryPolicy: DENY` everywhere; tracked: true.
- Cloud-session network egress may block moonpetal.eu and other external hosts; the Shopify connector still works (server-side). Storefront screenshots need the user to allow the domain in the environment's network settings.

## Decisions log (keep appending)

### 2026-08/09 — critter_selection round 1 screening (129 rows)
Loubna's first returned `critter_selection.xlsx` was the WRONG FILE (all 129 rows intact, no deletions/marks). She is resending the corrected version. Screening of the full 129 was completed and stands:

**BLOCKED — 6, excluded automatically, never list:**
- CU_21326 Cute and Creepy Bat Cat Talking Board with Planchette (talking board, ritual)
- MK_26227 Black Cat Magick Black Fig Candle (ritual in description)
- MK_27227 Black Cat Magick Book Shaped Flower Vase (ritual in description)
- MK_26327 Black Cat Magick Curled Cat Tealight Holder (altar; also zero stock)
- MK_26427 Black Cat Magick Oil Burner (ritual in description)
- MK_26727 Black Cat Magick Pendulum Divination Kit (ritual, divination, pendulum)

**PROFANITY — 2. Loubna's decision: KEEP BOTH, RENAME with Moonpetal-friendly titles (no profanity in title or SEO); supplier SKU keeps the mapping:**
- DK_28824 Bat Shit Crazy Keyring (€6.99 est)
- DK_90122 Bat Shit Crazy Mug (€9.99 est)

**UNSELLABLE right now — 10 (skip; pages can come later when stocked/fed):**
- Zero stock: CH_25024 Bat Tealight Candle Holder; CN_22727 Pastel Bat Brew Cauldron Oil Burner; CN_22627 Pastel Bat Cat Oil Burner
- Not in EU feed (no real cost — never price off GBP trade file): HA_85827 Metal Black Cat Ornament with Witch Hat; HA_85727 Metal Skeleton Dog Ornament with Pumpkin; TX1501 Highland Cow Door Stop; TX1523 Strawberry Door Stop; FW_90225 Fawn Light Up Canvas Plaque; FW_90325 Sleeping Fox Light Up Canvas Plaque
- Discontinued: FW_89125 Green Fox Trinket Dish

**REVIEW-tier — 18, sent to Loubna as `witch_magick_picks.xlsx` (YES/NO in column A), AWAITING her picks:**
CU_20926 Bats, Cats and Witches Hats Fabric Wall Hanging; CU_20026 Cute and Creepy Bat Cat Mug; CU_41826 In My Witch Era Bat Cat Enamel Keyring; CU_21026 In My Witch Era Bat Cat Fabric Wall Hanging; CU_41926 In My Witch Era Bat Cat and Moon Enamel Keyring; MK_58127 Black Cat Book of Spells Oil Burner and Wax Warmer; MK_58027 Black Cat Magick Book Stack Vase; MK_27627 Black Cat Magick Book of Spells A5 Notebook with Pen; MK_27127 Black Cat Magick Book of Spells Book Shaped Mug; MK_27427 Black Cat Magick Book of Spells Shaped Storage Box; MK_26627 Black Cat Magick Trinket Dish; FR_34827 Frightful Folk Black Cat Lidded Mug; MK_26827 Magick Black Cat Face Mug; MK_26527 Magick Black Cat Face Trinket Dish; MK_27027 Sitting Magick Black Cat Mug; MK_27527 The Protector Magick Black Cat Keyring; HA_45626 Orange Metal Owl Ornament with Witch Hat and Ghost; HA_45426 Red Metal Owl Ornament with Witch Hat

**CLEAN — 93** (~20 cats → The Cattery, ~73 others → The Menagerie: bats, owls, foxes, fawns, dogs, doorstops). Final set = (her resent file's surviving rows) ∩ (clean + her YES picks + the 2 renamed profanity items), minus unsellable.

### 2026-09-27 — repo move
Project moved from lpop24/g01tawjsf (unrelated Java repo, abandoned) to onyxceo/moonpetal (this repo). Nothing was ever committed to the old repo. Loubna deletes the old repo herself (no API for repo deletion).

### 2026-09-27 — SDW order #3065 import + full stock/price sync (StockLevelsEU-3.csv)
Loubna's physical-shop order (65 lines, 13 Sep 2026, parsed from Safari webarchive → `data/order_3065_items.json`):
- **31 Satya-brand items excluded** on her instruction (all incense "by Satya").
- **33 products created** (of 34 non-Satya: LI_32127 already existed) — full house treatment: Moonpetal names, copy, SEO, image from SDW xlarge, cost, stock, DENY, ACTIVE, NOT published to any channel. Source data: `data/new_products.json` + `data/to_add.json`. New productTypes introduced: **Incense**, **Gift Bags**, **Candle Holders** (add to browse-category collections later). New set tags: Hollow Library, Black Rose, Yin & Yang, Frightful Folk.
- WI_40627 (Magic Toad cone holder) created at qty 0 — zero stock in feed, page kept for notify-me.
- **NEW PRICING RULE from Loubna (supersedes EU-RRP formula): retail = EU feed cost × 3 × 1.17 (LU VAT), rounded up to next €.99.** Store has taxesIncluded=true; per-country EU VAT display relies on Shopify's "include/exclude tax based on customer country" setting (verify in Settings → Taxes before launch).
- **All 48 existing products repriced** to the same rule from fresh feed costs (47 price changes; full before/after in `data/reprice_plan.json` for rollback). 10 stale unit costs refreshed (biggest: book mugs BC_352/3/4 26 6.63→9.28).
- **Inventory synced to StockLevelsEU-3** (30 corrections). Newly out of stock: LI_57827 (Hollow Library Wax Warmer), LI_33227 (Coffin Bookmark). Back in stock: PA_78327, PA_78427. Near-zero: LI_58927 (5), SE_42427 (45), LI_32927 (50).
- **HA_18026 (Shelf of Shadows Advent Calendar) is GONE from the EU feed** — cannot be reordered; qty set to 0, kept at stored cost. Flag to Loubna before publish.
- Split bookmarks SET_37426-*: parent feed stock 131 displays × 9 per design = 1179 each; unit cost 35.19/36 = 0.98.
- Content screen: no BLOCKED terms in the order's product names. Witch/magic-adjacent items (Magic Toad, Witching Hour) included — Loubna ordered them herself for the physical shop, which counts as her decision.
- Store total now: **81 ACTIVE products, 0 published to Online Store** (verified).
- Note: `moon-petal-11` in Loubna's admin URL = this same store (myshopifyDomain 7q0h34-1x.myshopify.com, moonpetal.eu); confirmed by catalog + "Moonpetal Redesign (WIP)" theme. Never call switch-shop from a non-interactive session (revokes the token).

## Outstanding work, in order
1. **Blocked on Loubna**: corrected `critter_selection.xlsx` (with her row deletions) + `witch_magick_picks.xlsx` (YES/NO). Then: re-screen, intersect with her deletions, price from EU feed, name (incl. the two profanity renames), write copy, create products unpublished, build The Cattery + The Menagerie smart collections. Report exact counts.
2. Batch-fix productType on the 48 live products; create browse-category smart collections; build navigation menus (themed worlds primary, Shop-by secondary).
3. Legacy collection cleanup with Loubna (repurpose vs delete, 18 collections).
4. Back-in-stock app: Loubna installs (SC Back in Stock / Notify Me! / similar, free tier). Then ensure sold-out product pages show the notify widget.
5. Homepage/theme: plum-dominant, mystical-adjacent, cosy. Collection landing sections per themed world. Current theme is stock Dawn.
6. Final pass: publish ALL products + collections to Online Store channel in one go on Loubna's word.
7. Ongoing: stock sync script — match EU feed by SKU, update inventory quantities, report new zero-stock items (their pages stay up for notify-me capture).

## Working style
- Loubna gives direct instructions and expects execution + a concise report after, not check-ins before. Ask only when a decision is genuinely hers (brand, taste, money, content-rule edge cases).
- Never present work as done when it is partially done — state exact counts (e.g. "14 of 48 created").
- She cannot see tool output. Anything she must read goes in the reply or a file.
