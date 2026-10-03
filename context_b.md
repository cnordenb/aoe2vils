# aoe2vils — working context

_Working notes for Claude sessions. This file is gitignored, like `context_a.md`, which holds the full design chat log (don't edit it). Consolidated 2026-09-28 from that log and a full read of `index.html`, then updated after later edits. Line numbers match the 2,927-line version of 2026-09-28. Edits on 2026-09-29 (Xolotl Warrior, Castle order, pastures) shifted them by a few lines from about line 931 on; grep the name._

## What it is
- **"Villagers required · AoE2 DE"** is a single-file, zero-dependency static page (`index.html`). It makes no network requests (since 2026-10-01; it used to load Alegreya and Alegreya Sans from Google Fonts). The fonts are embedded as base64 woff2 in a `<script>` in `<head>` (before the `<style>`), which decodes them and adds them with `document.fonts.add(new FontFace(family, bytes, { weight, display: "block" }))` (user, 2026-10-01: DevTools' Network panel counted the earlier `@font-face url(data:…)` rules as four requests, so the site showed 6; now 2 when hosted, the page and `/favicon.ico`, 1 from a local file). The fonts: Alegreya 700 and Alegreya Sans 400 / 500 / 700, Latin subset only (Google's `unicode-range` for it: every character the page uses that the fonts have; ✓ ☐ → aren't in Alegreya at all and come from a fallback, as before). About 96 KB of font, 128 KB as base64: the page went from 278 KB to 407 KB. `display: "block"`. The comment above the font data carries the fonts' copyright lines and the SIL Open Font License 1.1 (required when redistributing; the files' own name tables hold the copyright and the OFL URL too). Italics are synthesized, as before (no italic face was ever loaded). There's no build, tests or CI: open the file in a browser.
- **Question it answers:** how many villagers per resource does it take to keep N buildings producing unit X non-stop, or to keep N copies of a building under construction non-stop? Per resource it computes cost ÷ time ÷ gather rate, rounded up to whole villagers.
- **Origin:** a from-scratch remake of the "Villagers required" page at aoe2-de-tools.herokuapp.com (github.com/PeterWhiteJavascript/aoe2-de-tools). It was built as a claude.ai artifact over one chat (Opus 5.5, 2026-09-26 → 09-28). The current `index.html` is the artifact as of the chat's last exchange.
- **Where the data comes from:**
  - aoe2techtree.net source data (updated 2026-09-22): unit and building costs, train and build times, civs, bonuses, tech trees, ages and upgrade lines.
  - The original tool's dataset: base gather rates and farm tables.
  - The user, who measured them by hand: alternative-income rates.
  - Bonus and tech texts on the page are short summaries, not the game's wording.

## The user's standing rules (from the chat)
- Static HTML + CSS + JS only.
- **No animations whatsoever** (strict, project-wide, 2026-09-28). No CSS `transition`, `animation` or `@keyframes`, no JS-driven motion, no smooth scrolling. State changes are instant: e.g. the dropdown arrow flips direction immediately. `requestAnimationFrame` is fine for layout timing.
- **No page scrolling.** Everything fits the viewport; only dropdown lists scroll.
- Units are picked from a searchable dropdown, with no images or unit buttons.
- Results appear as a spreadsheet: resources as columns, units as rows, and a Total row that is always visible.
- **Totals are whole villagers, always rounded up.** Row cells (units, Farm Reseeding, House Building) show one decimal, normally rounded (user, 2026-09-29; was rounded-up whole numbers). The exact value, to 2 decimals, appears in the hover tooltip.
- **aoe2techtree.net is the source of truth** for civs, bonuses, tech trees, ages and upgrade lines.
- **Layout stability.** Showing or hiding something must not move other UI; the user said so explicitly for the Stone column and the dropdown checkboxes. Hidden controls keep their space. The # Builders column is a deliberate exception: it collapses, and the user wants it left as is.
- **No redundant text.** The user had these removed: help texts, the original tool's link and description, the Conscription description, the "Page cleared" message, Undo ("I don't want an undo function"), and the whole status line (2026-09-28).
  - Don't add confirmation or status messages. The table itself shows what changed.
- **The user dictates UI wording.** Keep these strings verbatim:
  - "No applicable civ bonuses."
  - "No applicable team bonuses from allies."
  - "Civilization bonuses, team bonuses, and unique technologies are inhibited when Full Tech Tree is enabled."
  - the ally tag "(from Huns Ally)"
  - "Villagers required"
  - "Add a unit" / "Add a unit or building", and "Unit" / "Unit/Building" (both follow the Buildings box)
  - "# Buildings", "# Builders", "# Wood vils" and the other column headers
  - "Naval" (was "Include Dock", 2026-09-30); the fishing spots "Shore" / "Deep" / "Trap"; "+x w/min from Foragers"; the income sources "Trees" (was Lumberjack), "Gold Mine" (was Gold Miner), "Stone Mine" (was Stone Miner), "Custom Rate", and the Gurjara ones "Garrisoned Sheep" / "Garrisoned Cows" (were Gurjara Livestock / Gurjara Livestock (large)); "+x f/min from Lumberjacks"; "+x g/min from Stone Miners" (Claude's, after the heading, not dictated); column headings by source (`HEADS`): "# Lumberjacks" (Trees), "# Gold Miners" (Gold Mine; the user wrote "# Gold Miner", read as a typo), "# Stone Miners" (Stone Mine), "# Farmers" (Farm, also Khitan Pasture), "# Foragers", "# Shepherds", "# Hunters", "# Fishermen"
  - the adder footer tooltips "Take wood expense of farm reseeding into account" (since 2026-09-30 "Take wood expense of farm reseeding and fish trap rebuilding into account"; the user agreed to it mentioning fish traps, the words are Claude's) and "Take wood expense of house building into account"
  - the row name "Fish Trap Rebuilding"
  - "Idle TC Time" (checkbox) and "Villagers missing" (2026-10-01)
  - Transition Eco's fourth radio "After [] seconds", and its heading "After 45s" / "After 5m" / "After 5m30s" in place of "On reaching <Age>" (2026-10-01)
- **Bonus labels** (user, 2026-09-30): each `FX` label says what the number does, with one verb ("Foragers work +15% faster", "Castles cost −25%", "Villagers carry +3"), and no explanation after a comma. When in doubt, copy aoe2techtree.net's civ bonus list (`strings.json` civ help texts). All labels were rewritten that way; the app keeps "−" for minus. Berbers' farm-table label carries per-age numbers (+5% Dark, +10% from Feudal). Romans' one bonus is two labels (gather / build), since the app models them separately.
- **Custom income** (user, 2026-09-30): every source dropdown ends with "Custom Rate" (verbatim; was "Custom..."), so the wood / gold / stone dropdowns always show now (the "nothing to choose, hidden" rule and the selects' `hidden` attribute are gone). While it's picked, that box's heading rate becomes a number field (`#rate-in-<res>`, 0.1–999.9 per villager per minute, one decimal; as tall as the text, so the box doesn't grow). It starts from the rate the resource had before, and is remembered per resource (`state.customRate`, saved; Clear All empties it). The rate is taken as typed: no upgrades (hidden, like any non-standard source) or gathering bonuses (`rates()` sets it last). While typing, an empty or invalid field keeps the last rate. The column reads "# Villagers" (user wrote "# Villager", read as the plural). Farm Reseeding, Foragers and Varangian food gold don't apply to custom food; Shu lumberjack food doesn't apply to custom wood. Checked: 50/min wood halves the lumberjacks (4.79 → 2.43); Transition Eco uses it (10 × 45/min × 130 s = 975).
- **Column headings follow the source** (user, 2026-09-30): see the wording list. `fitHeads()` (in `update()` and after `fit()` sizes the text) shows each such heading only where it neither overflows nor makes the heading row taller than the plain form ("# Wood vils", "# Gold vils", "# Food vils") would; otherwise that heading shows the plain form (`th.plain`, spans `.hl` / `.hs`). The heading row is then held at the plain forms' height (inline `height` on the row), so a heading that fits in fewer lines (e.g. "# Villagers" vs "# Wood vils" at 1,000 px with Buildings on) doesn't move the table either. Results: wide layouts and landscape phones show every name; 600–1,150 px with Buildings off shows "# Wood vils" / "# Gold vils" (they'd wrap to two lines) but the food names; phones vary by what fits. So the heading row is never taller than before. `th.dataset.label` holds the name for the Transition Eco steppers' labels.
- **Varangian food gold** (user, 2026-09-30): replaces the Varangian Shepherd / Hunter / Fisherman gold sources (old saves fall back to Gold Mine). `FX.Varangians` = `foodgold` ("Shepherding, fishing, and hunting also generate gold", aoe2techtree): 10% of the food rate for Sheep, Hunt and Fisherman villagers, 1 gold per 20 food for Fishing Ships (user). While one of those is the food source, the Gold box shows "+x g/min from Shepherds / Hunters / Fishermen / Fishing Ships" ("from Ships" in narrow layouts) under Relics (`#food-gold`, same style as the Forager line; its line exists while Varangians, kept for other sources; user's choices). x = food gatherers (the food total rounded up, or Fishing Ships) × gold each. That gold, divided by the gold rate, comes off the gold total after the relics; the gold total's tooltip lists both ("after 2 relics cover 2.63 and 13 shepherds cover 1.13"). Transition Eco adds it to "On reaching" gold (10 hunters, Feudal: 53.3). Checked: 13 shepherds +25.7 g/min (19.8 food each), 10 Fishing Ships (deep) +12.7.
- **Shu lumberjack food** (user, 2026-09-30): replaces the Shu Lumberjack food source (old saves fall back to Farm). `FX.Shu` = `woodfood` ("Lumberjacks generate food in addition to wood", aoe2techtree), 6.859% of the lumberjack rate (upgrades and wood bonuses included), only while Trees is the wood source. The Food box shows "+x f/min from Lumberjacks" under the upgrades (`#wood-food`; line exists while Shu, kept for other wood sources). x = lumberjacks (the wood total rounded up, Farm Reseeding and House Building included) × food each; that food, divided by the food rate, comes off the food total (tooltip "after N lumberjacks cover X"). Farm Reseeding counts the farmers before this, so its lumberjacks are sized for slightly more farmers (about 0.08 per lumberjack): a small overestimate, accepted rather than iterating. Transition Eco adds the wood villagers' food to "On reaching" food (10 lumberjacks, Feudal: 34.6). Checked: 5 lumberjacks +8 f/min, covering 0.39 farmers.
- **Polish stone gold** (user, 2026-09-30): replaces the "Polish Stone Miner" gold source (old saves fall back to Gold Mine). `FX.Poles` = `stonegold` ("Stone Miners generate gold in addition to stone", aoe2techtree, capital M as there), 33% of the stone rate (upgrades and stone bonuses included; `stoneGoldShare()`), only while Stone Mine is the stone source. The Gold box shows "+x g/min from Stone Miners" under Relics (`#stone-gold`, the Varangian line's place; the two never coexist since both are civ bonuses; line exists while Poles, kept for other stone sources). x = stone miners (the stone total rounded up) × gold each; that gold, divided by the gold rate, comes off the gold total after the relics (tooltip "after N stone miners cover X"). Only buildings cost stone, so in Villagers Required the line (space kept) and the bonus-line entry hide while the Stone column is blanked (`relevant()` checks `showStone()`), like Inca stone. Transition Eco adds the stone villagers' gold to "On reaching" gold (10 stone miners, Feudal: 154.4). Checked: Poles Imperial, Knight-line + Castle: 10 stone miners +71.3 g/min, gold 6.58 → 3.45 → 4; with Stone Mining + Shaft Mining 7 miners +66 g/min.
  - Layout: "from Stone Miners" is longer than the Varangian line's narrow "from Ships", so `fitIncome()` gives it 2 lines at 768 px (the box 17 px taller than Varangians'), small type at 412 px, and 3 lines at 360 px (like Shu). Checked at 13 sizes × both tabs: nothing overflows, and the Gold box is one height across stone sources and with Buildings on or off.
- **Extra-income lines** (Forager, Shu food, Burgundian relic food, Varangian gold, Polish gold, Paper Money gold, Burgundian Vineyards gold; class `.extra-inc`): "+x … /min" and "from …" (`.inc-from`, the name in `.inc-who`). `fitIncome()` (in `applyLayout()`) measures the longest text a line can show ("999.9", every source name) and picks one line, two lines (`html.inc-2l`), two in smaller type (`html.inc-small`), or three, the name on its own (`html.inc-3l`; "from Lumberjacks" at 360 px), so the box height never depends on the numbers or the source. Checked at 14 sizes in both tabs: box heights equal across food sources, nothing overflows.
- **Building bonuses and the Buildings box** (user, 2026-09-30): a civ or team bonus that changes only buildings (`buildingsOnly(e)`: every UNITS entry its matcher changes is a building, taking the cost resource into account) is left out of the bonus line and the ally picker's preview while Buildings is off, unless it still reaches the table (`inReach()`): a building row in the table, Houses with House Building on, or Farms / Pastures with Farm Reseeding on and Farm food. E.g. Franks' Castles hide; Malians' "Buildings cost −15% wood" and Spanish builders stay (Houses, Farms); Teutons' Farms show with Farm food only; Inca stone hides (Houses and Farms cost no stone). The effects themselves still apply. `update()` re-renders the line and re-fits if its height changed.
- **Naval bonuses** (user, 2026-09-30): likewise, a bonus that changes only naval entries (`navalOnly(e)`: every UNITS entry its matcher changes passes `isDock`: Dock units, Dock, Harbor, Fish Trap) is left out of the bonus line and the ally preview while Naval is off, unless it reaches the table (`inTable()`, shared with the building rule: a row it changes, or the Fish Traps of Fish Trap Rebuilding). `buildingsOnly()` / `navalOnly()` share `onlyOf()`; `inReach()` needs both rules to pass. Hidden with Naval off: Koreans "Warships cost −20% wood", Vikings "Warships cost −20%" and team "Docks cost −15%", Italians "Fishing Ships cost −15%", Sicilians team "Transport Ships cost −50%", Malay's two Fish Trap bonuses (shown with Fish Trap Rebuilding or a Fish Trap row). Kept: gathering bonuses (Japanese "Fishing Ships work faster", Danes' drop-off) and mixed ones (Persians' Town Centers and Docks, Wei's Traction Trebuchets and Lou Chuans). Checked for every civ, own and as an ally.
- **Upgrades a source doesn't use are hidden, not greyed** (user, 2026-09-30): e.g. Heavy Plow unless Farm is the food source, Stone Mining while Feitoria is the stone source. Space and ticks are kept. Greying stays for age locks and techs the civ lacks.
- **Entry naming rules:**
  - "(Elite) X" or "(Heavy) X" for two-step Elite/Heavy lines, including Castle unique units whose Elite version has identical cost and time.
  - "X-line" for longer lines, named after the first unit: "Battering Ram-line" (user, 2026-09-30; was "Ram-line").
  - "A / B" when only part of a line merges (e.g. "Crossbowman / Arbalester").
  - Not a line when the only extra member is one civ's unique upgrade: named after the common unit, "Bombard Cannon" (Houfnice) and "Elite Skirmisher" (Imperial Skirmisher) (user, 2026-09-30).
  - A two-step line with different names: "Stone/Fortified Wall", "Armored/Siege Elephant" (user, 2026-09-30); with a unique extra: "(Heavy) Camel Rider" (Camel Scout, Gurjaras).
  - An entry may show its unique member where it can be had (`NAME_WITH_UNIQUE`, `displayName()`): "Elite Skirmisher" reads "Elite/Imperial Skirmisher" for Generic with Include Regional & Unique (Post-Imp too) and for Vietnamese outside Post-Imp (Post-Imp: "Imperial Skirmisher") (user, 2026-09-30).
  - A unique member's name finds its entry only where that member can be had (`uniqueShown()`: the civ has it, or Generic with Include Regional & Unique), Post-Imp included: e.g. "camel scout" finds (Heavy) Camel Rider only for Gurjaras or Generic with R&U, "houfnice" Bombard Cannon only for Bohemians or Generic with R&U (user asked for Camel Scout; applied to every unique member). Search strings are built per query (`searchOf()`), no longer the fixed `SEARCH`.
  - An entry whose unique member comes before the rest of its line (`EARLY_UNIQUE`; only "(Heavy) Camel Rider" → Camel Scout, Feudal Age; user, 2026-09-30: Gurjaras have Camel Scout in Feudal, Camel Rider only from Castle, the same here but for the name): where the member can be had (`uniqueShown()`: Gurjaras, or Generic with Include Regional & Unique) the entry reads "Camel Scout" until its own age (Castle), in the list and the table. `ageOf()` opens it in Feudal for Generic with Include Regional & Unique (was Castle) and for Gurjaras under Full Tech Tree (they keep their unique units; the entry isn't `U`-flagged, so it had gone to Castle). Gurjaras already had it from Feudal (per-civ age 2), named "(Heavy) Camel Rider". Row identity stays "(Heavy) Camel Rider", so saves and age changes just rename it. Checked: Gurjaras Dark / Feudal / Castle / FTT / Post-Imp, Generic with and without Include Regional & Unique, Berbers (± FTT), Hindustanis.
  - Renamed entries keep working in old saves through `RENAMED` in `load()`.
- The user routinely invites follow-up questions ("Ask me follow-up questions if you need to").

## UI & behaviour (current)
- **Header:**
  - The title, then the **tabs** (Villagers Required | Transition Eco), then **Clear All** and **Clear Civs** centred.
  - The header arrangement is measured, not tied to width thresholds (`fitHeader()`, see Conventions & gotchas). It uses one line when everything fits; otherwise the picker labels go above the boxes, then the pickers get their own line, then (phones and about 600 px) the tabs get their own line too.
  - **Allied Civilization(s)** picker: multi-select, 0–7 civs, each civ's team bonus shown on the right. The label carries a count, e.g. "Allied Civilization(s) 3/7" (`#ally-count`, set in `syncButtons()`). It always shows, including 0/7, so the label's width never changes.
  - **Your Civilization** picker: defaults to Generic.
  - Below them: the bonus line `#civ-fx` (clamped to 2 lines; full text on hover) and the Age radios (default Imperial).
- **Settings row.** It shares the table's column grid, so each resource's settings sit directly above its column.
  - **Saxons' TCs/Castles** (user, 2026-09-30; not Full Tech Tree): a −/+ count "TCs/Castles" (verbatim), 0–4 (`TC_MAX`), default 1 (`state.tcCastles`, saved; Clear All resets it to 1, `isCleared()` checks it), placed where the user asked: under Clear table, above the # Buildings column (`#tc-castles` in the adder, `align-self: flex-end; margin-top: auto`; `.gone` for other civs). Their bonus (aoe2techtree "Foot Soldiers cost -5% per Town Center or Castle controlled (maximum -20%)") is a `cost` FX whose `p` is a getter, 5 × the count. The bonus line always reads "Foot Soldiers cost −5% per Town Center or Castle controlled (maximum −20%)" whatever the count (user; aoe2techtree's text with the app's "−"; `fxText()` prints a label without "{p}" as it is), while a unit's cost tooltip says what applies ("Foot Soldiers cost −10%", the FX's `appliedLabel`, used by `effective()`). Foot Soldiers = infantry and foot archers (user's choice; `FOOT_SOLDIER` = military and (`I` or `A` without `M`)): Militia-line, Spearman-line, Archer-line, Skirmishers, Hand Cannoneers, Slingers…; not cavalry, cavalry archers, siege, ships, Monks, Villagers. (`fxText()` also no longer calls `findIndex` on a plain 0 `p`: it greys the label.) The count's −/+ redraws the rows (`counter(…, renderRows)`), since costs change. Layout: the adder had ~10 px spare, so Saxons' settings row is 13 px taller in the wide layout (149 vs 136 at 1,920); in narrow layouts the count is a line at the adder's right end (+23 px). Checked: 1 TC Militia-line 48f 19g, Archer 24w 43g, Skirmisher 33w 24f, Hand Cannoneer 43f 48g; 4 TCs −20%; Knight-line, Cavalry Archer, Monk, Villager unchanged.
  - **Adder:** its label, then the checkbox line, the unit combo, and a footer row: the **Farm Reseeding** and **House Building** checkboxes on the left (user, 2026-09-29, where the status line used to be) and a right-aligned "Clear table" link. The hidden link keeps its space (`.adder-foot .linkbtn[hidden]`), so nothing in the row moves when it appears. The checkbox line is Naval / Buildings / *slot*. The slot shows Full Tech Tree for a civ, or Include Regional & Unique for Generic.
  - **Wood / Food / Gold / Stone:** the rate per villager per minute, plus the eco-upgrade checkboxes.
    - Each resource has a plain source `<select>` (`.src-sel`) directly under its heading, so all four line up.
      - Food: Farm ("Pasture" for Khitans), Berries, Hunt, Sheep, Fisherman (villagers fishing from the shore; was "Fishing"), Fishing Ship, plus any available alternative incomes. Heavy Plow, Wheelbarrow and Hand Cart sit under it (`#food-ups`, was `#farm-ups`; the indented line to their left, `.nested`, was removed 2026-09-30) and show only while the source is Farm (hidden, space kept, otherwise: `laneShown()`). For Khitans' pastures (not under Full Tech Tree) Heavy Plow is invisible but keeps its space (`#food-ups .opt[hidden]`): its pasture version, Pastoralism, doesn't change what farmers carry, so it lives only in the Pasture Reseeding row.
      - **Fishing Ship** (2026-09-29; reworked 2026-09-30): Fishing Lines and Gillnets take Heavy Plow's and Wheelbarrow's places (each pair shares an `.opt-stack` grid cell, so nothing moves); Hand Cart's line stays empty. Ticks are kept when the boxes swap. With Fishing Ship the swapped-out Heavy Plow / Wheelbarrow stop reserving width (`#food-ups.ship`), leaving room for the radios.
      - **Fishing spot radios** (user, 2026-09-30): Shore / Deep / Trap (`state.fish`, default Deep, saved, reset to Deep by Clear All; `isCleared()` checks it) in a column right of the upgrades (`.food-row`, `#fish-spot`), same font and size as the checkboxes, one per upgrade line. Hidden (space kept) for other sources. Transition Eco uses the spot too.
        - **Trap from Feudal Age** (user, 2026-09-30; `TRAP_AGE`, `fishSpot()`, `spotTip()`): in Dark Age (Transition Eco: Researching Feudal) the Trap radio is greyed and disabled (`.opt.na`, tooltip "Available from Feudal Age. Fish Traps"), as is "Ship: Trap" in the split dropdown. A Trap choice is kept, like an age-locked tick: Deep shows and counts meanwhile (rate, no Fish Trap Rebuilding), and Trap returns in Feudal Age. Picking another spot in Dark Age replaces it. Radio tooltips are now set in `syncSettings()` from `SPOT_TIP`. Everything that computes reads `fishSpot()`; `state.fish` stays the saved choice (`isCleared()` still checks it).
        - **No room** (phones, and windows narrower than about 720 px, e.g. 600 px tablets and 640×360; user's choice): `fitFish()` (in `applyLayout()`) measures whether the radios fit beside Fishing Lines / Gillnets and otherwise sets `html.fish-dd`. Then the radios are gone and the Food dropdown lists the ship once per spot: "Ship: Shore" / "Ship: Deep" / "Ship: Trap" (values `fship:shore` etc., titled from the radios' tooltips; the change handler splits the value). Resizing across the line keeps the choice.
        - Room to spare where they fit: 10 px at 1,920, 6 px mid (1,151–1,320), 13 px at 768, 4 px at 730 and 700×400. The column reads "# Fishing Ships" ("# Ships" in the narrow layout, via `.wide-only`, because the long form wraps and pushes the table down at 600–1,150 px), and Fishing Ships don't count toward Villagers required.
      - Wood, Gold, Stone: the standard source plus any available alternative incomes. A dropdown with nothing to choose is hidden but keeps its space (`visibility: hidden`), so switching civ moves nothing. (Gold used to always show because of Relic; since 2026-09-29 it hides like the others.)
      - **Relics** (Gold box, below Gold Shaft Mining; user, 2026-09-29): a −/+ count 0–99 (`state.relics`, saved, reset by Clear All) and the relics' total gold per minute (`#relic-out`, e.g. "90g/min", whole numbers from 100 up; the tooltip gives the per-relic rate). Before Castle Age (`RELIC_AGE`) the count shows 0, greyed and locked ("Available from Castle Age"); the number set is kept and returns in Castle Age, like an age-locked tick (`relicsNow()`; locking is in `syncSettings()`). See How a number is computed. It fits the line Gold has spare below its two upgrades, so no box grows: checked at 11 sizes (1,920×1,080 to 360×640 and landscape phones) in both tabs, including 99 Aztec relics ("3950g/min"). Phones wrap it onto two lines (three at 360 px), inside the space Food's box already sets. Hidden (space kept) in Transition Eco, where relics don't count (user).
        - **Relic food line** (user, 2026-09-30: the food was only in the tooltip): with a Burgundian civ or ally (`relicfood`), the Food box shows "+x f/min from Relics" under the upgrades, after Shu's lumberjack line (`#relic-food`, `.extra-inc`, sized by `fitIncome()`). It exists while the bonus applies (`relicIncome(1).food`; gone under Full Tech Tree), reads "+0" with no relics, and is blank (space kept) before Castle Age and in Transition Eco. So a Burgundian civ or ally makes the Food box one line taller (+20 px at 1,920; the table moves down), and Shu with a Burgundian ally gets two lines. Checked at 13 sizes: no overflow, no page scroll, same height at 0 / 3 / 99 relics and in Feudal.
      - **Near Fortified Church** (Georgians, not Full Tech Tree; user, 2026-09-30; `church` FX, bonus line "Fortified Churches provide Villagers in a 9 tiles radius with +10% work rate", aoe2techtree): a checkbox "Near Fortified Church" (verbatim) at the bottom of each resource box (`[data-church]`, `state.church`, saved; Clear All resets it, `isCleared()` checks it). Ticked, that resource's villagers gather +10% (`rates()`, after the `rate` effects; not Fishing Ships; a niche or custom source replaces the rate anyway). Greyed and locked before Castle Age (`CHURCH_AGE`, the Fortified Church's age; tick kept), hidden (space kept) for a source that isn't villagers (`churchShown()`: Fishing Ship, Feitoria, livestock, Custom Rate). Shown in both tabs (in `TE_KINDS`). Layout: `html.church` (set in `applyLayout()`) makes the resource fieldsets flex columns with the box at `margin-top: auto`, so the four sit level whatever each box holds; without the class the boxes are `display: none` and the fieldsets unchanged (Britons and Generic pixel-identical to HEAD at 13 sizes). Georgians' settings row is one line taller (+22 px at 1,920; +20 at 768–1,250; the label wraps to two lines at 412–600 px, three at 360–390, so +35 to +51 there). Checked: rates 25.6 / 22.3 / 25.1 / 23.8 with all four; Transition Eco Researching Imperial 10 lumberjacks = 811.
      - With any source other than Lumberjack / Gold Miner / Stone Miner, that resource's eco upgrades are hidden (space and ticks kept; `#wood-ups`, `#gold-ups`, `#stone-ups` get `hidden` in `syncSettings()`, `.ups[hidden]` keeps the space). Was greyed until 2026-09-30.
      - **Foragers** (Wood box, under the upgrades; user, 2026-09-30): replaces the "Portuguese Forager" wood source. While Portuguese (own civ, not under Full Tech Tree) and Berries is the food source it reads "+x w/min from Foragers" (`#forage`; user: centred, italic, all in the muted colour, the number not highlighted, the plus to stress it's added), x = foragers × 5. Its line exists whenever the civ is Portuguese (`.forage.gone` otherwise) and is invisible but kept for other food sources, so picking Berries moves nothing and switching to or from Portuguese moves the table by one line (20 px, 37 px on phones where it wraps; user's choice). See How a number is computed.
  - **Over the Technologies column:** Post-Imp and Clear Techs. Narrow layouts show a copy inside the Technologies panel instead.
- **Minimum rows** (user, 2026-09-30; wide layout only): the table is at least four rows tall (`MIN_ROWS`). Empty filler rows (`tr.filler`, rendered after the automatic rows, each with a hidden stepper so it's as tall as a unit row) fill up as unit and automatic rows appear. `fillRows()` also shows extra fillers while Villagers required would cover the technologies above it: Portuguese get one, Franks two (three in the mid layout); everyone else four. The user wasn't asked about the extra ones.
- **Custom units and buildings** (user, 2026-09-30; `CUSTOM`, `customRowHtml()`, `addCustom(kind)`):
  - Dropdown: "Custom Unit..." is the first entry under the Units heading; with Buildings ticked, "Custom Building..." is the first under Buildings (both verbatim). Each shows with nothing typed or while the text matches "custom unit" / "custom building", reads "n/5 in table", and greys out (can't be picked) at 5 of its kind (`CUSTOM_MAX`).
  - Picking one adds "Custom Unit n" / "Custom Building n" (lowest free n) with 0 costs and 30 s, cursor in its first field.
  - Fields (costs 0–999 for a unit, 0–9999 for a building; time from 1 s up to the same maximum, one decimal; user) on a line under the name (`.cu-fields`), since the name's line is too short at every width (page ≤ ~1,440 px wide; Unit column 334 px at 1920, 218 px mid). One line at 1,400 px and up and on tablets; 2–3 on narrower mid widths and phones. A unit has wood, food and gold (user: a unit can't cost stone) and training time; a building has wood, gold and stone (no food) and build time, plus a # Builders stepper up to 53 (`CUSTOM_BUILDERS`, the largest building's limit, since its size is unknown; `maxBuilders()`). Its minimum is 0, the only building that goes below 1 (user, 2026-09-30): 0 builders counts none (# Builders total, Villagers required) but takes the same time as 1 (`row.builders || 1` in `rowTime()` / `costMeta()`; `update()` counts `row.builders` as is for custom rows). `load()` keeps 0, clamps below 0 to 0, and gives a missing value 1.
  - Saved in `state.rows` as `{ name, count, builders, custom: { wood, food, gold, stone, t } }`; `load()` keeps names 1–5 of each kind, zeroes the resource the kind can't have, clamps numbers and builders (`cleanCustom()`), drops duplicates. `rowUnit(row)` gives the UNITS-style row (group "Custom", flags "" or "B", every age), used wherever rows were looked up by name.
  - No type, so only bonuses for every unit / every building reach them (user): Portuguese gold for units; Malians' wood, Inca stone, Spanish and Roman builders, Treadmill Crane for buildings. The name's line then shows "→ cost and time after bonuses (and builders)" (`.cu-eff`). A custom unit takes 1 population (House Building); a building takes none, counts as a building row (# Builders and Stone columns show), and keeps building bonuses in the bonus line with Buildings off (`inReach()`).
- **How many rows fit** (answered 2026-09-30): no fixed number. `addUnit()` / `addCustom()` add the row, `fit()` shrinks the table text down to 9 px, and a row that still doesn't fit is taken back out silently. Measured with Farm Reseeding and House Building showing: 1920×1080 54 unit rows, 1600×900 38, 1366×768 30, 1280×720 23, 768×1024 34, 412×780 15, 390×740 10, 360×640 1.
- **Row order** (user, 2026-10-03): units first, buildings after them, each kind in the order added (custom units with the units, custom buildings with the buildings). E.g. adding Villager, Farm, Knight, Castle shows Villager, Knight-line, Farm, Castle. `orderRows()` stable-sorts `state.rows` itself at the start of `renderRows()`, so the saved order and the DOM order match (the remove handler's focus-the-next-row relies on that); old saves are sorted on load. The automatic rows (Farm Reseeding, Fish Trap Rebuilding, House Building) still come after all of them.
- **Table columns:** red × remove · Unit · # Builders (1 to the building's limit, building rows only; see `SIZE` / `maxBuilders()`) · # Buildings (0–99, square −/+ steppers) · # Wood / Food / Gold / Stone vils.
  - The name cell holds the display name, an **Upgrade** button (earlier line versions only) and cost/time meta. Modified values are underlined; hovering shows the base values plus every effect applied.
  - A row with # Buildings 0 has its whole "name cost / time" string greyed out (the coloured resource amounts too), in the same faded tone as its zero cells (`tr.idle`, set in `update()`; user, 2026-09-29). The Farm's "25s" box greys with it, label in that tone and the box at .45 opacity, but stays usable (user, 2026-09-30; not disabled, Claude's choice, like the rest of the row). A red "can't make" flag wins over the grey.
  - Rows the civ or age can't make stay in the table, flagged in red ("not in Goths tech tree" / "needs Castle Age").
  - The Total row sums builders, buildings and each resource.
- **Technologies panel** (right of the table), top to bottom:
  - the civ's relevant unique techs, each with a one-line effect (none for Generic)
  - Conscription
  - Treadmill Crane (only while Buildings is ticked)
  - Shipwright (only while Naval is ticked; 2026-09-30)
  - **Wide layout, fixed places** (user, 2026-09-30; `html:not(.narrow):not(.tab-te)`, so the mid layout too, not narrow or Transition Eco): top to bottom the Castle Age unique tech, the Imperial Age one (two `.ut-slot`s in `#ut-list`; no civ has two of one age), Conscription, Shipwright, Treadmill Crane (CSS `order`; the DOM and narrow order stay Conscription, Crane, Shipwright). A hidden tech keeps its space (Circumnavigation with Naval off, Shipwright, Crane); a missing unique tech leaves a checkbox-high gap. So ticking Naval or Buildings moves nothing. Switching civ can, since unique techs differ in height.
  - **Villagers required** (wide layout) is absolutely placed with its number centred on the Totals row (`placeGrand()`, from `fillRows()` in `update()` and `fit()`).
  - **One cell with Post-Imp** (user, 2026-09-30): the Post-Imp cell covers the settings row's bottom line (`margin-bottom: -1px` + background), and `placeTechs()` (in `fit()`) moves the column up (`#techs` inline `top`, negative) so the first unique tech's place is level with Gold / Stone Shaft Mining (anchor: Gold's, always laid out). The table area (`.sheet-wrap`) doesn't clip in the wide layout for this; `.board` still does. The column's top stays about 14 px below Clear Techs, so both stay clickable. With the extra room Franks fit in four rows at 1,366 px and up (mid layout still adds one or two).
  - the **Villagers required** grand total
  - **Idle TC Time** (user, 2026-10-01; wide and mid layouts only, `html.narrow .idle-tc` hides it; hidden in Transition Eco with the grand box): a checkbox "Idle TC Time" under the Villagers required number (`#idle-tc` inside `.grand-box`), default off. Ticked, it shows a 0–10:00 slider in seconds (`#idle-t`, `state.idleT`, default 0) and the time in a text field you can type in (`#idle-t-out`, "mm:ss"; user, 2026-10-01), up to 59:59 (`IDLE_MAX` = 3599). `parseIdle()` takes "12:30", "2:5" or plain seconds ("90"); over 59:59 becomes 59:59. While typing, an incomplete or invalid time keeps the last one; leaving the field or Enter writes it out as mm:ss; `syncIdle()` leaves the field alone while it has focus. A typed time past 10:00 puts the slider at its end and counts exactly (25:00 = 60, 59:59 = 143), and under it "Villagers missing" with its number (`#vils-missing`). Slider (moved above Villagers missing at the user's request), the time's text, label and number share one colour (the field's border stays `--rule`), `--idle` on `.idle-out` (user, 2026-10-01): green at 00:00 (`.idle-none`, new token `--good`: #2f7a3c light, #6cc07a dark), neutral `--ink` (white in dark mode) at 00:01–00:24 (`.idle-some`, no villager missing yet), red `--food` (the app's red) from 00:25 on. It switches at once, with no transition (checked: 0 s transition on every part). The slider takes the colour via `accent-color`, so its thumb and filled part change; the unfilled track stays the browser's grey. Villagers missing = ⌊idle seconds ÷ 25⌋ (`IDLE_VIL_T`): always the base 25 s, whatever the civ (user; Persians' faster TCs ignored), so 10:00 = 24. (The slider's extra 10:01 step meaning ">10:00" / "24+", added earlier on 2026-10-01, went when typing came in: the user gave the slider 00:00–10:00 and typing up to 59:59. An old save's 601 reads 10:01, 24.) The field is 3.3em wide, so the slider is 105 px at 1,920 and 81 px mid; the block's height (89 px) didn't change. Shown apart from Villagers required, not added to it (user). Both saved (`state.idleTc`, `state.idleT`); Clear All resets both (user), and `isCleared()` checks them. Unticking the box resets the time to 0 (user, 2026-10-01), so ticking it again starts at 00:00; `load()` likewise keeps no time for an unticked box. `syncIdle()` (in `refreshAll()` and startup) draws it; nothing else depends on it, so its handlers don't call `update()`.
    - **Space:** the count and slider keep their space while unticked (`.idle-out[hidden]` is `visibility: hidden`), so ticking moves nothing. The block is 89 px tall at 1,920 px and hangs below the Totals row; `fit()` now also requires it to end inside the Technologies column (`idleFits()`), shrinking the table text or refusing a row otherwise. Cost: 5–6 fewer table rows at the most (Generic, Include Regional & Unique, Naval: 1,920×1,080 36 → 31, 1,600×900 27 → 22, 1,366×768 20 → 14, 1,280×720 18 → 12), and windows about 600 px tall get smaller table text sooner (1,366×600 with Franks: 15 → 14.5 px). Every other element is where HEAD has it at 15 sizes (1,920×1,080 to 360×640, short windows included); narrow layouts and phones are unchanged. Collapsing it while unticked would avoid the cost but let ticking it shrink a full table's text; not offered to the user yet as of 2026-10-01.
- **Unit dropdown:**
  - **Groups.** Units: Town Center, Dock, Barracks, Archery Range, Stable, Siege Workshop, Castle, Donjon, Monastery, Market. Buildings: Economic, Military, each alphabetical. Group order is simply the `UNITS` array order. Within Castle, the unique units come first (alphabetical), then Trebuchet and Petard (user, 2026-09-29).
  - **Contents:** only what the civ can make by the selected age. Generic lists every common entry, and adds the regional and unique ones only with Include Regional & Unique ticked.
  - **Search:** any member's name finds its merged entry, with a grey hint showing the matched name. Picking an entry that's already in the table adds 1 to its count.
  - **Stays open** while you toggle Naval, Buildings, Full Tech Tree, Include Regional & Unique, the Age radios or Post-Imp.
- **Arrow buttons** (all three search pickers: unit, allies, your civ): the arrow at the right of the box is a real button (`.combo-toggle`, created in `combo()`).
  - Clicking it opens a closed list and closes an open one, the same as clicking outside: typed unit text is kept, and the civ box restores the civ name.
  - It points down while closed and up while open, and flips instantly (no animation). CSS keys off the input's `aria-expanded`.
  - The button takes no focus (`mousedown` is prevented, `tabindex=-1`, `aria-hidden`), so the text box keeps focus and typing still works. Keyboard users keep ArrowDown / Escape.
  - Tested with real mouse clicks via the Chrome DevTools Protocol, including switching between pickers.
- **Naval** (was Include Dock; default off) hides the Dock group plus the Dock, Fish Trap and Harbor buildings from the dropdown. Rows already in the table stay. Since 2026-09-30 it also shows (and puts into effect) the naval techs, like Buildings does Treadmill Crane: Circumnavigation (Portuguese unique tech, `naval: true` in `UT`, `navalOk()`) and Shipwright. Their ticks are kept while hidden.
- **Shipwright** (user, 2026-09-30; `state.shipwright`, `#shipwright`): Imperial Age, ships (`DOCK` = flag `D`) −20% wood and trained 50% faster (aoe2techtree: "Ships cost -20% wood and build +50% faster"; now researched at the University). `SHIPWRIGHT_MISSING` lists the 29 civs without it, from aoe2techtree `data.json` (unchanged since 09-28): Armenians, Bohemians, Bulgarians, Burgundians, Burmese, Cumans, Franks, Georgians, Hindustanis, Huns, Jurchens, Khitans, Khmer, Lithuanians, Malians, Mapuche, Muisca, Persians, Poles, Portuguese, Saracens, Slavs, Tatars, Teutons, Tupi, Vietnamese, Vikings, Wei, Wu. Handled exactly like Treadmill Crane: greyed with "X can't research Shipwright" for those civs, greyed before Imperial, ticked by Post-Imp, cleared by Clear Techs and Clear All, hidden in Transition Eco. Example: Britons War Galley / Galleon 90w / 27 s → 72w / 18 s.
- **Include Regional & Unique** (default off; Generic only, added 2026-09-28):
  - It shares Full Tech Tree's spot. Generic shows this box, and any other civ shows Full Tech Tree instead.
  - **Off:** Generic's dropdown hides every regional or unique unit and building, leaving 58 of the 191 entries.
  - **On:** Generic's dropdown lists all 191 entries.
  - **Scope:** it filters Generic's dropdown and (since 2026-09-30) Generic's alternative incomes in the resource dropdowns. Rows already in the table, the numbers, the Upgrade button and other civs are untouched.
  - **As a view option:** like Naval, Clear All leaves it alone, ticking it doesn't make Clear All appear, and its state is saved (`state.regional`).
  - **Classification:** unique = the entry's `U` flag. Regional = any member listed in `REGIONAL` (see Data).
  - **Mixed lines:** Knight-line (with Savar), Militia-line (with Legionary), Scout Cavalry-line (with Winged Hussar), Elite Skirmisher (with Imperial Skirmisher), and Bombard Cannon (with Houfnice) count as common. (Heavy) Camel Rider (was Camel Rider-line) counts as regional.
- **Buildings** (default off) adds buildings to the dropdown, switches the two labels and shows Treadmill Crane.
  - # Builders shows while Buildings is on *or* the table has a building row.
  - The Stone column shows under the same condition. Otherwise it's blanked in place, because only buildings cost stone.
- **Full Tech Tree** (default off):
  - For Generic, Include Regional & Unique takes its place. It starts unticked when you move from Generic to a civ, and stays ticked when you switch between civs.
  - **Adds** every non-unique unit, building and tech for the civ. The civ keeps its own unique units. aoe2techtree's "regional" units and buildings count as non-unique.
  - **Switches off** the civ's bonuses, its own and its allies' team bonuses, and unique techs (greyed, with ticks kept).
  - **Also removes** early-age exceptions (Burgundian eco upgrades, Cuman Feudal rams…), farm tables (Khitans get Farms back) and ally unlocks.
- **Post-Imp** (default off):
  - Rows move up their line as far as the civ can reach (user, 2026-09-30): `postImpTechs()` presses Upgrade (`upgradeRow()`, shared with the button) on every row until it can't, e.g. Archer → Crossbowman / Arbalester, Skirmisher → Elite Skirmisher / Imperial Skirmisher, Eagle Scout → Elite Eagle Warrior, Stone Wall-line → Fortified Wall; a row reaching one already in the table merges into it. It runs whenever Post-Imp is applied: ticking it, changing civ or Full Tech Tree while it's on, and at startup with a saved Post-Imp. `upgradeRow()` also caps builders at the new building's limit. Custom rows stay.
  - Sets Imperial Age and ticks every tech the civ can research, including Treadmill Crane. Unique techs are skipped under Full Tech Tree.
  - The dropdown and table show only the latest version of each line that the civ can reach, named for that civ. For example, Militia-line becomes Champion, but Two-Handed Swordsman for Malay and Legionary for Romans.
  - Unticking any tech, or leaving Imperial, unticks Post-Imp. Changing the civ or Full Tech Tree keeps it on and re-ticks.
- **Alternative incomes** (no checkbox since 2026-09-28; the Niche Incomes box was removed):
  - Every `NICHE` source joins its resource's dropdown whenever it's available.
    - The rest follow their civ (e.g. Portuguese get Feitorias, Gurjaras garrisoned livestock) and their age. Since 2026-09-30 no source is worked by villagers: Polish stone gold, Paper Money and Burgundian Vineyards became extra-income lines.
    - Team-bonus sources would also come with an ally (`team: true`); none is left since Burgundian Relic food became the relic count's `relicfood` team bonus.
    - Generic lists a civ's own incomes (all of them now: Feitoria, Garrisoned Sheep / Cows) only with Include Regional & Unique ticked (user, 2026-09-30; was every source always). `nicheAvail()` checks `state.regional` for Generic, and the checkbox now runs `syncSettings()`, so unticking it puts a chosen one back to the standard source (Trees / Farm / Gold Mine / Stone Mine), headings following.
  - A non-villager source renames its column ("# Feitorias", "# Livestock"; also the standard Fishing Ship source, "# Fishing Ships") and doesn't count toward Villagers required. `kindOf()` returns `fship` for it.
  - Losing access to a chosen source (civ, age, ally or Full Tech Tree change) snaps it back to the standard source. `toggleAlly()` runs `syncSettings()` so that works for allies too.
- **Allies:**
  - Each ally's team bonus applies, tagged "(from X Ally)". The same team bonus never counts twice.
  - A Berber ally unlocks Genitours; an Italian ally unlocks the Condottiero.
  - Generic is exempt from Full Tech Tree, so allies' team bonuses always apply with Generic.
- **Tech checkboxes** are greyed out (never hidden) until their age, and switched off and greyed if the civ lacks the tech. Every one has a tooltip in the form "reason. effect".
- **One entry per unit line** (per building):
  - Adding another version of a line replaces the existing row in place and keeps its count.
  - **Upgrade** moves the row to the next version the civ can reach now. If there's none, the button is greyed with the reason. If that version already has a row, the two merge.
  - The Castle and Donjon Serjeant can coexist, as can the Castle and Barracks Huskarl.
  - Serjeant (Donjon) has no Upgrade button (user, 2026-09-30): its `EDGES` link to Elite Serjeant (the Castle's entry) is gone, so it's a line of its own. Upgrade and Post-Imp used to turn a Donjon row into the Castle's (Elite) Serjeant (12 s instead of 16 s); Post-Imp now leaves it as "Serjeant (Donjon)".
- **Farm Reseeding row:**
  - Added automatically while Farm (or Pasture) is the food source, at least one farmer is needed and the adder's Farm Reseeding box is ticked (`state.useReseed`, default on). It isn't in the dropdown and has no remove button or counts.
  - Has Horse Collar, Heavy Plow and Crop Rotation checkboxes. Heavy Plow is the same tick as in the Food lane.
  - Reads "[farms × farm wood]w / [cycle]s", and its lumberjacks go into the wood column.
  - **Khitans** (pastures, i.e. without Full Tech Tree) get **Pasture Reseeding** instead (user, 2026-09-29):
    - Checkboxes Domestication, Pastoralism and Transhumance. They're the `hcol`, `hp` and `crop` ticks under other names (`PASTURE_TECH`, `techName()`, `techTitle()`), with the same ages and prerequisite chain. So a Horse Collar / Heavy Plow tick carries over as Domestication / Pastoralism when switching to Khitans.
    - Khitans are no longer in `NO_TECH` for those three.
    - Under Full Tech Tree Khitans have Farms, and the row is the normal Farm Reseeding.
- **Fish Trap Rebuilding row** (user, 2026-09-30; `rebuildTraps()`, `trapRowHtml()`, `#trap-row`, class `reseed` so it's italic and counts in `fillRows()`):
  - Farm Reseeding's counterpart: added automatically (between Farm Reseeding and House Building) while Fishing Ship on Trap is the food source (`trapsOn()`), at least one ship is needed, and the adder's **Farm Reseeding** box is ticked. The user chose that one box covers both rather than a third footer box; only one of the two rows can show at a time, so nothing moves.
  - No checkboxes (Fishing Lines / Gillnets are in the Food box). Reads "[traps × trap wood]w / [cycle]s"; its lumberjacks go into the wood column. One unit row tall at every size (checked at 13 sizes, no overflow).
- **Fish Trap build time** (user, 2026-09-30): 47 s, 42 s with Fishing Lines, 40 s with Gillnets (`TRAP_TIME`; the `UNITS` entry now says 47, was aoe2techtree's 40). `effective()` sets it for the Fish Trap, and only while Fishing Lines / Gillnets show (Fishing Ship food), as "Gillnets: built in 40s" in the tooltip. The lanes' change handler refreshes a Fish Trap row's cost text (`refreshRowMeta()`), since it doesn't re-render rows.
  - Fishing Ships build Fish Traps, so Treadmill Crane (user) and the villager builder bonuses (Spanish, Romans; Claude's reasoning, not confirmed) skip it: they match `VIL_BLD` = buildings except the Fish Trap.
  - The Japanese "Fishing Ships work +5/10/15/20% faster" bonus does speed it up (user): `effective()` turns a `rate` effect on `fship` into a time effect for the Fish Trap, always (not only with Fishing Ship food). Japanese Imperial with Gillnets: 40 ÷ 1.2 = 33.3 s; rebuilding with a Knight-line: 4 traps, 400w / 1446.4 s, 0.71 lumberjacks.
  - No builders, like the Farm (user, 2026-09-30): see Builder limits below.
  - **One per Fishing Ship box** (user, 2026-09-30): like the Farm's "25s" box, a box after the Fish Trap row's cost labelled with the Fishing Ship's training time ("40s"; Persians' Docks and Shipwright shorten it: 33.3 s Imperial, 26.7 s with Shipwright). Ticked, the row's time is that training time instead of the build time: 100w every 40 s. Shared code: `WORKER` (Farm → Villager, Fish Trap → Fishing Ship), `perWorker()`, `workerTime()`, `perWorkerHtml()` (were `farmPerVil`, `villagerTime`, `farmOptHtml`; `FARM` is gone). Saved in the same `row.perVil` field. The tooltip drops the time effects (Gillnets, Japanese) and adds "One per Fishing Ship: a Fishing Ship every 40s". Greys out with the row at 0 buildings. Width: "Fish Trap ☐ 40s" is as long as the longer unit names, so it wraps where they do (phones, 600 px, where the Unit column is 29 px and every row wraps); at 768 px and up it's one line.
- **House Building row** (user, 2026-09-29; `houseBuilding()`, `houseRowHtml()`, `#house-row`, class `houses`):
  - Added automatically after Farm Reseeding while the adder's House Building box is ticked (`state.useHouses`, default on) and units (not buildings) are being trained, i.e. some unit row has a count above 0, unless the civ needs no houses (Huns, not under Full Tech Tree). Not in the dropdown; no remove button, counts or checkboxes.
  - Reads "[house wood]w / [seconds between houses]s", e.g. "25w / 150s" for one Stable of Knights, "25w / 75s" for two. Its wood cell counts lumberjacks, going into the Wood total: the house wood, plus the build time as lumberjack time lost (the user: "Lumberjack builds a house, so for every house there is 25 seconds of lost villager income on wood"). The cell's tooltip splits the two.
  - Costs 1 unit row on phones: with both automatic rows showing, 412×780 fits 10, 390×740 7 and 360×640 1 (11 / 8 / 2 without it). No page scroll at any tested size.
- **Buttons appear only when they'd do something.** Clear All, Clear Civs and Clear Techs keep their space while hidden.
  - **Clear All** resets the table, civ, allies, age, food and sources, every tech tick and Post-Imp.
    - It leaves Naval, Buildings, Include Regional & Unique, Farm Reseeding and House Building alone, and ticking those doesn't make it appear.
    - Full Tech Tree simply hides, because Clear All returns to Generic.
  - **Clear Civs** sets Generic with no allies.
  - **Clear Techs** unticks everything, including remembered age-locked ticks, and ends Post-Imp.
  - **Clear table** empties the table.
- **Narrow layout** (`html.narrow`, ≤ 1150 px, or ≤ 1320 px with # Builders):
  - The unit adder takes its own full-width row. The four resource boxes stay **side by side** under it, in the table's order: Wood, Food, Gold, Stone.
  - The user explicitly rejected a layout where Food gets its own full-width row. The table's Unit / # Builders / # Buildings columns sitting under the resource boxes is fine.
  - On phones (≤ 599 px) the boxes are compacted: each heading's rate goes under the name, padding and Food's indent are tighter, and labels wrap. "Wheel&shy;barrow" carries a soft hyphen because headless Chrome on Windows doesn't apply `hyphens: auto`.
  - Headless-Chrome checks from 360 to 1,150 px found no label, heading or dropdown overflowing its box. At 360 px the Wood and Stone dropdowns truncate with an ellipsis ("Lumberja…").
- **Tabs** (added 2026-09-28): **Villagers Required** (the default view; the page always opens on it and the tab isn't restored on reload) and **Transition Eco**. Arrow keys switch tabs.
- **Transition Eco tab.** The same page, with these changes:
  - **Hidden:** the whole adder (label, Naval / Buildings / Full Tech Tree / Include Regional & Unique, unit search), the Unit / # Builders / # Buildings columns, the table rows, the Age radios, Post-Imp, Conscription, Treadmill Crane, Shipwright and Villagers required.
  - In the wide and mid layouts they keep their space (`visibility: hidden`), so the resource boxes don't move when switching tabs. In the stacked layout the adder's and Age radios' own lines collapse; otherwise a phone couldn't show the Transition Eco row.
  - **Stone** always shows.
  - **Left of the table** (where the Unit column is): radios "Researching Feudal Age (130s)", "Researching Castle Age (160s)", "Researching Imperial Age (190s)" (on phones "Feudal Age (130s)" …), and "After [] seconds" (user, 2026-10-01; value 4 = `TE_AFTER`): [] is a number field (`#te-after`, `.te-after-in`, styled like a Custom Rate field, as tall as the text), whole seconds 0 to 98h14m53s = 353,693 (`TE_AFTER_MAX`; user, 2026-10-01, "specifically"; was 3,600; the field was 4.3em wide for 6 digits; since 2026-10-01 late, 2.15em (user: "half as short", read as half as wide), room for 3 digits, widening for a longer time via a hidden copy (`.te-after-wrap` / `.te-after-size` after the field, `sizeTeAfter()`): 3600 → 2.33em, 353693 → 3.28em, never clipped, line height unchanged), 300 to start with (`TE_AFTER_SECS`), saved as `state.te.after`. Clicking the field doesn't pick the radio; typing a time does (`setTeAfter()`). While typing, an empty or invalid field keeps the last time; leaving it puts the time back. `renderTE()` leaves the field alone while it has focus. The `#te-row` change / input handlers skip it for the villager steppers. The label's tooltip says "What the villagers gather in that time, in Imperial Age".
    - **Its slider** (user, 2026-10-01): 0–900 s (`#te-after-slide`; was 0–300 until the user widened it the same day), right after the label in `.te-after-line` (flex, wrapping). Shown only while After is picked, hidden but keeping its space otherwise (`[hidden]` = `visibility: hidden`), so switching radios moves nothing. Field and slider stay in step (`syncTeAfter()`, from `renderTE()` and `setTeAfter()`); a time over 900 puts the slider at its end. Same line at 1,160 px and wider (at 1,160×700 since the narrower field, 94 px wide), on tablets and at 915×412 (no extra height); its own line (+16 px) at 600 px and portrait phones (where, with the narrower field, "seconds" no longer wraps at 412 / 360 px). On short windows (`max-height: 520px`) its flex-basis is 3em so it stays beside the label (65 px at 640×360), since a line more there would cut the row off.
    - The times include civ age-up bonuses, underlined, with the base and the bonus on hover. Malay: advancing +66% faster. Portuguese: technologies +25% faster. Persians: Town Centers work +5/10/15% faster in the age being left.
  - **Under each resource column:** a −/+ stepper (0–200) for how many villagers work it, or Feitorias / livestock / Fishing Ships for such a source (the column heading says which).
  - **The Researching radio sets the current age.** Researching Feudal means Dark Age, Castle means Feudal, Imperial means Castle. That drives which eco upgrades, alternative incomes and per-age civ bonuses apply. "After [] seconds" is always Imperial Age (user, 2026-10-01): `curAge()` = research − 1 = 3, `teAfter()` tells it apart.
  - **Right panel:** only the unique techs that change gathering or income and can count there (`teTech()`: not Imperial, except with "After [] seconds", which lists Paper Money, the only Imperial one; Grand Trunk Road, Burgundian Vineyards since 2026-09-30), Clear Techs, and "On reaching <Age>" with the Wood / Food / Gold / Stone gathered during the research. With "After [] seconds" the heading reads "After 45s", "After 5m" (whole minutes) or "After 5m30s" (`minSec()`), and from an hour "After 1h", "After 1h2m5s", "After 98h14m53s" (Claude's extension: parts that are 0 are left out, so 3,601 s is "After 1h1s"; 0 s is "After 0s"), and the amounts are what's gathered in that time.
  - **Tab-specific bonus line** (`relevant(e)`): each tab lists only the bonuses that matter to it, for your civ, allies' team bonuses and the allies picker's preview column.
    - Transition Eco: gathering and age-ups (`rate`, `farm`, `ageup`, `agebonus` = `TE_KINDS`; `relicgold` left 2026-09-29, since relics don't count there).
    - Villagers Required: everything except `ageup` / `agebonus`.
    - With nothing left, the usual "No applicable civ bonuses." / "No applicable team bonuses from allies." shows.
    - E.g. Transition Eco hides unit costs, training and build times, unlocks, and "farms hold more food" (Farm Reseeding only). Byzantines show "No applicable civ bonuses." there.
  - **Typed amounts** (user, 2026-10-01): each "On reaching" / "After" amount is a text field (`.te-amt`, `#te-wood` …). Typing a whole number (spaces and commas ignored) sets that resource's count to the fewest that gather at least that much, up to 200 (`TE_MAX`): n = ⌈(amount − age-up bonus − extra) ÷ per villager⌉, from `teCalc` (per resource: `per` = rate × time, `extra` = what other gatherers bring in, e.g. Portuguese foragers' wood, `bonus` = Dravidian / Ethiopian age-up resources), which `updateTE()` fills. The other resources' counts stay; their amounts may change through the extras. While typing, anything but a whole number changes nothing and the field keeps its text (`updateTE()` skips a focused field); Enter or leaving it shows what that many gather (`dataset.shown`). With nothing gathered per villager (After 0s) typing changes nothing. Checked: Researching Feudal, 1000 wood → 20 lumberjacks, 1009; 999999 → 200, 10096; Dravidians 150 → 0 (the +200 covers it), 300 → 2; Portuguese with 10 foragers, wood 50 → 0, 500 → 11 (530).
    - **Width:** each field sits over a hidden copy of its text (`.te-amt-wrap` grid, `.te-amt-size`, `sizeAmt()`; the input is `width: 0; min-width: 100%` so its default 20-character width doesn't size the cell), so it's exactly as wide as the number. Boxed (like the other fields) in the panel; in the strip above the table (≤ 820 px, short windows) underlined only, so the strip is as wide as with plain numbers. Checked against the plain numbers at 15 sizes × 0 / 10 / 200 villagers each: identical layout, no clipped digits. (A first try sized the fields in `ch`; the digits are narrower than "0"'s `ch` here, so the strip grew 2–13 px and could wrap.)
  - **Age-up resources:** the totals include what the civ receives on reaching the age (Dravidians +200 wood, Ethiopians +100 food and +100 gold), even with nobody working. The hover breaks it down ("247.0 gold gathered + 100 on reaching Feudal Age"). Not with "After [] seconds" (no age is reached); for the same reason the bonus line leaves out `ageup` / `agebonus` bonuses there (`relevant()`), so Malay and Dravidians show "No applicable civ bonuses.".
  - **Clear All** also resets the Researching radio to Feudal, the After time to 300 s and the steppers to 0 (`isCleared()` checks the time).
  - **Layout with the fourth radio** (checked 2026-10-01 at 15 sizes against the page before it): the Transition Eco row is one radio line taller (+23–24 px; +39 on 412 / 390 px phones, where "seconds" wraps under the field; +57 at 360 px) and stays fully visible, with no page scroll and the resource boxes unmoved. At 640×360 (phone held sideways) the table text shrinks from 11 to 9.5 px to fit it. On phones the strip above the table is one or two lines depending on how wide the amounts are, for every radio (not new). With the 98h maximum the amounts can reach 8 digits (27,470,156 wood with 200 lumberjacks); at 640×360 the strip then wraps and 7 px of the row is cut off at the 9 px minimum text (50 villagers per resource still fit). Not fixed.
- **Short windows** (`@media (max-height: 520px)` together with the stacked layout, i.e. phones held sideways):
  - The header and resource boxes take the phones' compact sizes, and the civ bonus line clamps to one line.
  - The Technologies panel becomes the strip above the table, as at ≤ 820 px wide (that media query now reads `(max-width: 820px), (max-height: 520px)`).
  - In Transition Eco the Researching radios share one line, with the short labels.
- **Transition Eco verified** in headless Chrome (2026-09-28) at 21 sizes × 2 setups, from 1,920×1,080 through short desktop windows, tablets in both orientations, and phones in portrait and landscape down to 640×360:
  - no header overlaps, no page scroll
  - the Transition Eco row and the totals always fully visible
  - nothing sticking out of its box
- **Layout:** works down to about 360×640. At ≤ 820 px the Technologies panel becomes a strip above the table. `fit()` shrinks the table text (15 px, less on narrow screens, down to 9 px), and `addUnit()` silently refuses a row that still doesn't fit.
- **Theme and storage:** light and dark come from CSS tokens. Dark applies via `prefers-color-scheme` or `data-theme="dark"`, the claude.ai artifact-viewer convention; nothing on the page sets it. State persists in `localStorage`.

## How a number is computed
- **Per row and resource** (`update()`, 1959):
  - `v = count × cost[res] ÷ t ÷ rate[res]`, everything per second. The cell shows `v` to 1 decimal (`fmt`); hovering shows 2 decimals. Only totals use `vils()` = `ceil(v − 1e-9)`.
  - For building rows, `t` is the build time after civ and tech effects. Then `withBuilders(t, n) = 3t / (n + 2)` applies for n > 1. That matches the user's example: a 200 s Castle with 4 builders takes 100 s.
- **Base time by age** (user, 2026-09-30; `AGE_TIME`, `baseTime(u)`): the Eagle Scout trains in 40 s in Feudal Age and 30 s in Castle and Imperial Age, without upgrading (the `UNITS` entry now says 40; was aoe2techtree's 46). `baseTime()` is the starting time in `effective()` and the base in `costMeta()`'s underline test and tooltip, so 30 s in Castle isn't marked as changed and bonuses start from it: Aztecs 34.8 s Feudal, 26.1 s Castle ("Base 20f 50g / 30s"); Imperial with Conscription 22.6 s. Only the Eagle Scout has one.
- **`effective(u)`** (1714) applies `activeFx()` (1666) in this order:
  1. your civ's `FX`
  2. each ally's team `FX` (skipped when the ally is your own civ)
  3. ticked unique techs whose age is reached
  4. Conscription
  5. Treadmill Crane (only with Buildings on)
  - Under Full Tech Tree (not Generic), steps 1–3 are skipped.
  - How each effect applies:
    - `cost`: × (1 − p/100) on `e.res`, or on all four resources.
    - `flat`: subtracts `v`, floored at 0.
    - `swap`: moves `frac` of one resource's cost into another; with `v`, the other resource gets a fixed `v` instead of the same amount. Kamandaran (Persians) uses it: the Archer-line's gold goes and it costs +25 wood, 25w 45g → 50w (user, 2026-09-30; was 70w; desc and label "Archer-line costs +25 wood instead of gold"). Hussite Reforms, Corvinian Army and Forced Levy stay 1:1.
    - `time`: t ÷ (1 + p/100).
  - Costs are `Math.round`ed only at the very end. Times aren't rounded; they're shown to 1 decimal.
  - Order matters: `flat` vs `cost`, and a `swap` sees the already-discounted cost.
  - Time effects multiply. Example: Goths with a Huns ally and Post-Imp train Champions in 21 s ÷ 1.2 ÷ 2 ÷ 1.33 = 6.6 s.
- **`rates()`** (1745):
  - Base rates:
    - Farm: `FARM_TABLES[active table][farmKey()] / 60`.
    - Fishing Ship: `FISH_SHIP[state.fish][gn | fl | ""] / 60`.
    - Other food: `FOOD[src]`.
    - Wood: `LUMBER[best of tms | bs | dba | ""] / 60`.
    - Gold and stone: `BASE`, times `MULT` for each ticked upgrade.
  - **Mule Cart technologies** (Armenians, user, 2026-09-30; `ecoboost` FX, label "Mule Cart technology effects +40%", aoe2techtree): each Lumber / Mining Camp tech's gain is 40% larger. Gold / stone: each `MULT` 1.15 becomes 1.21. Wood: each step of the measured `LUMBER` table (Double-Bit Axe 27.9 / 23.3 = +19.7%, Bow Saw 33.2 / 27.9 = +19.0%, Two-Man Saw +10.2%) grows by 40%, so wood = 23.3 × Π(1 + step gain × 1.4); without the bonus that's the table itself. Armenians: wood 29.7 with Double-Bit Axe, 37.6 with Bow Saw (Britons 27.9 / 33.2); gold 27.6 / 33.4; stone 26.1 (they lack Two-Man Saw and Stone Shaft Mining). Claude's reading of "+40%" for the measured wood table; the user hasn't given measured Armenian rates. In `TE_KINDS`, so it's listed and counts in Transition Eco too.
  - Then `rate` effects:
    - `all`: every resource, except food while Fishing Ship is the source (it's the Romans' villager bonus).
    - `food`: food, Fishing Ships included (Danes: "Fishing Ships and Villagers drop off +5% food").
    - A resource name: that resource.
    - A food-source name (`farm`, `berries`, `hunt`, `sheep`, `fship`): food, but only while that source is selected. Japanese: `fship` +5/10/15/20% by age.
  - Last, a chosen niche source **replaces** its resource's rate:
    - Rate: `perMin / 60`. The Varangian sources use 10% of the bonus-adjusted food rate instead.
    - Niche gold × 1.1 with Grand Trunk Road.
- **Foragers** (`forageEach()`, 2026-09-30): Portuguese `forage` FX, only while Berries is the food source. In `update()`, after House Building: foragers = the food total rounded up; their wood (× 5 per minute) divided by the wood rate comes off the wood total, never below 0 (in Feitorias if Feitoria is the wood source). The wood total's tooltip says "after N foragers cover X". Example: Portuguese Imperial, Knight-line + Villager on Berries: 13 foragers, 65 w/min, cover all 1.31 lumberjacks → wood 0. Transition Eco adds the food villagers' forager wood to "On reaching" wood (tooltip "(x by foragers)"): Portuguese, 10 berry villagers, Researching Feudal (104 s) = 86.7 wood.
- **Relics** (`relicIncome(n)`, Villagers Required only, none before Castle Age): 30 gold per minute each, × 1.33 with an Aztec civ or ally (`relicgold`), × 1.1 with Grand Trunk Road; plus 20 food each with a Burgundian civ or ally (`relicfood` team bonus, so off under Full Tech Tree and never for Generic on its own). In `update()` this income, divided by the resource's per-gatherer rate, comes off that resource's total (never below 0) before Farm Reseeding counts farmers. Per-row cells are unchanged; the total's tooltip says how much the relics cover. Example: Britons, Knight-line + Villager, 3 relics: gold 6.58 − 3.95 = 2.63 → 3.
- **House Building** (`houseBuilding(r, tot, popRate)`, after `reseed()`):
  - `popRate` = Σ over unit rows of count × `unitPop(u)` ÷ train time (with its bonuses). `unitPop` = 1, 0.5 for Karambit Warriors (`HALF_POP`), × (1 − p/100) for each active `pop` effect (Mahayana: Villagers and Monks −10%; Aznauri Cavalry: mounted −20%; both Imperial unique techs).
  - Population per house `housePop()` = 5 + `housepop` effects (Inca +5, civ bonus). Seconds between houses = housePop ÷ popRate.
  - House cost and build time come from `effective(House)`, so cost bonuses and build-speed bonuses apply: Wu team bonus (+100%, 12.5 s), Spanish (+30%), Romans (+5%), Treadmill Crane (+20%, only while Buildings is ticked, as ever).
  - Lumberjacks = houses per second × (house wood ÷ wood rate + build time). Example: Britons, 1 Stable of Knights: 25 ÷ 150 ÷ 0.388 = 0.43 + 25 ÷ 150 = 0.17 → 0.60.
  - Left out (one-off, not a rate): Chinese Town Centers +15, Goths +10 in Imperial, Slav military buildings +5 (team), Dravidian Docks +5 (team), Tupi towers and Castles +10 (team), Mongol Nomads. The house-builder time goes into the Wood column even when a non-villager wood source (Feitoria) is picked.
- **Totals:**
  - Each resource total is `ceil(exact sum)`, not the sum of the rounded cells, because villagers on one resource serve every row.
  - Farm Reseeding lumberjacks are added to the wood sum before rounding.
  - **Villagers required** = the rounded totals of every resource whose source is villagers, plus every builder (count × builders on each building row). The default single builder counts too. Farms and Fish Traps count none (`NO_BUILDERS`).
- **Farm Reseeding** (`reseed()`, 1936):
  - `farmers` = the food total rounded up (Farm source only).
  - `cycle` = farm build time (1 builder, with build-speed effects) + `farmFood()` ÷ farm rate.
  - `lumberjacks` = farmers × farm wood ÷ cycle ÷ wood rate.
  - `farmFood()` = (175 + Horse Collar 75 + Heavy Plow 125 + Crop Rotation 175), with the upgrade part × 2.25 for Sicilians, then × 1.15 for Maya and × 1.1 for a Chinese civ or ally.
  - **Pastures** (Khitans): 2 villagers per pasture, so pastures = ⌈farmers ÷ 2⌉ (9 farmers → 5 pastures). A pasture costs 110 w and takes 22 s to build (the `Pasture` entry, through `effective()`). `pastureFood()` = 345 + Domestication 115 + Pastoralism 230 + Transhumance 345 (so 345 / 460 / 690 / 1035, the user's figures). `cycle` = build time + pastureFood ÷ (2 × food rate); lumberjacks = pastures × 110 ÷ cycle ÷ wood rate. No `farmfood` bonus applies to pastures (e.g. a Chinese ally's +10% farm food), an unconfirmed choice. The pasture farm table already gives Pastoralism (`hp`) no effect on the gather rate.
  - Example: Khitans, 1 Town Center of villagers, no techs: 6 farmers → 3 pastures, 330w / 475.9 s, 1.79 lumberjacks (computed after the 2026-09-29 wood rates).
  - Farm wood follows cost effects: Teutons pay 36 w, Malians 51 w.
- **Fish Trap Rebuilding** (`rebuildTraps()`, after `reseed()`):
  - `traps` = the food total rounded up (Fishing Ships, one trap each).
  - `cycle` = trap build time (`effective(Fish Trap).t`, see above) + `trapFood()` ÷ trap rate per ship (Japanese bonus included).
  - `lumberjacks` = traps × trap wood ÷ cycle ÷ wood rate. Trap wood follows cost effects (Malay 67 w, Malians 85 w).
  - `trapFood()` = 715 (`TRAP_FOOD`, user) × each `trapfood` effect (Malay × 3). No farm-food bonus applies (Maya, Chinese, Sicilians).
  - Examples: Britons, Knight-line, no upgrades: 6 traps, 600w / 2099.6 s, 0.74 lumberjacks; Gillnets: 5 traps, 500w / 1735.7 s, 0.74; Malay with Gillnets: 335w / 5127 s, 0.17.
- **`pct(e)`** reads `e.p` as either a number or a `[Dark, Feudal, Castle, Imperial]` array indexed by `curAge()`: `state.age`, or in Transition Eco the age being left. `pctAt(e, age)` takes an explicit age.
- **Transition Eco amounts** (`updateTE()`):
  - "After [] seconds": that time instead of a research time, Imperial Age, no age-up resources. Checked: Generic, 10 lumberjacks, 300 s = 1165 (10 × 23.3/60 × 300); Dravidians with all Lumber Camp techs 1830 (no +200); Vietnamese with Paper Money +41 gold.
  - Research time: `ageUp(target)` = 130 / 160 / 190 s ÷ (1 + p/100) for each active `ageup` effect, with p taken in the age being left.
  - Amount per resource = count × `rates()[res]` (per second) × research time, rounded down, plus any `agebonus` for that resource. The hover shows the gathered part to one decimal.
  - Age-related bonuses checked against aoe2techtree on 2026-09-28 and **not** applied:
    - Bengalis get 2 villagers on age-up (villagers, not resources).
    - Byzantine, Italian and Muisca age-up discounts (paid before the research, so they don't change what's gathered).
    - Khmer need no buildings to advance.
    - Wei get a villager per eco upgrade.
    - Spanish +20 gold per technology researched: the user decided age-ups don't count (2026-09-28).
  - Example: Generic, 10 wood villagers, Researching Feudal = 10 × 23.3/60 × 130 = 504 wood. Malay = 304 (78.3 s). Persians = 480 (123.8 s). (Checked in headless Chrome, 2026-09-29.)
  - Farm Reseeding plays no part here.
- **Sanity check:** defaults (Generic, Imperial, no techs, Farm) with Knight-line ×1 and Archer ×1. Verified by hand from the code.

  | Wood | Food | Gold | Villagers required |
  |---|---|---|---|
  | 5 (1.84 + 1.74 reseeding + 1.11 houses) | 6 (5.91) | 10 (9.96) | 21 (20.56) |

  The chat's output 4 quotes 18 (17.71) for the same case, because it predates Farm Reseeding.

## Data baked in
- **`CIV_NAMES`** (862): 56 civs. `CIVS` = `["Generic", …CIV_NAMES]`.
- **`UNITS`** (864): 191 entries.
  - Shape: `[name, group, wood, food, gold, stone, seconds, flags, ageString, genericAge, members?]`.
  - `ageString` has one character per civ, in `CIV_NAMES` order: `0` = not in that civ's tech tree, `1`–`4` = first available age.
  - Entries per group:

    | Group | Entries |
    |---|---|
    | Castle | 72 |
    | Dock | 20 |
    | Barracks | 19 |
    | Military | 18 |
    | Economic | 17 |
    | Archery Range | 17 |
    | Siege Workshop | 11 |
    | Stable | 11 |
    | Monastery | 3 |
    | Town Center | 1 |
    | Donjon | 1 |
    | Market | 1 |

  - Line members that cost the same and train equally fast share one entry. Merging took the list from 284 entries to 190 (191 with the Xolotl Warrior, added later).
- **Flags:** the comment at 1423 documents I, A, M, G, S, W, X, D, N, L, B and U. Four more are used but undocumented: `R` Archery Range, `K` Barracks, `C` camel, `E` elephant.
- **`REGIONAL`** (1432) holds the member names aoe2techtree labels `RegionalUnit` or `RegionalBuilding` in its per-civ tree files (`data/trees/*.json`, fetched 2026-09-28), plus the Xolotl Warrior, which the user specified as regional (2026-09-29).
  - Units: the Eagle, Champi, Camel Rider, Battle Elephant, Armored/Siege Elephant, Elephant Archer, Steppe Lancer, Fire Lancer, Hei Guang, Rocket Cart, Mounted Crossbowman, Longship and Varangian Guard lines, plus Slinger, Traction Trebuchet, Catapult Galleon, Dromon, Lou Chuan and Xolotl Warrior.
  - Buildings: Caravanserai, Fortified Church, Mule Cart, Pasture, Settlement.
  - aoe2techtree's `UniqueUnit` / `UniqueBuilding` labels match the `U` flag in `UNITS` exactly.
  - `REGIONAL_OR_UNIQUE(u)` combines both checks. Only Generic's dropdown filter uses it.
- **`MEMBERS`** (1057) maps every single unit and building to `[unique 0/1, 56-char availability]`. **`EDGES`** (1335) lists the upgrade links.
  - Through `LINE_OF`, `KIDS` and `ENTRY_OF` they drive Post-Imp naming, the Upgrade button and the one-entry-per-line rule.
- **`ALIAS`** (1568) maps a member name to its entry name. Only `load()` uses it, to migrate old saves. Search doesn't need it: the search strings (`searchOf()`) already contain member names, the group and "units" or "buildings".
- **Gather rates:**
  - `LUMBER`, **per minute** (user, 2026-09-29): 23.3 unupgraded, 27.9 Double-Bit Axe, 33.2 Bow Saw, 36.6 Two-Man Saw. It replaced wood .39/s × 1.2 × 1.2 × 1.1.
  - `BASE`, per second: gold .38, stone .36.
  - `FOOD`, per second: berries .31, hunt .41, sheep .33, fish .43 (Fisherman).
  - `FISH_SHIP`, **per minute**, by spot then upgrade (user): shore 14.4 / 15.84 with Fishing Lines / 17.42 with Gillnets (2026-09-29; taken to be the shore fish rates when deep fish was added); deep 25.3 / 27.8 / 30.5 (2026-09-30); trap 20.9 / 23 / 25.3 (2026-09-30). Example: Japanese Imperial, Deep, Gillnets = 36.6; Trap, Gillnets = 30.4.
  - `FARM_TABLES`, **per minute**: tables for generic, aztecs, berbers, khmer and pasture, keyed `"" | hp | wb | wb+hp | hc | hc+hp`. The generic `hc` value without Heavy Plow (23.87) is an estimate. The pasture table is the user's figures of 2026-09-29: 22.8 unupgraded, 24.4 with Wheelbarrow, 25.4 with Hand Cart (was 22.89 / 23.96 / 25.26); Pastoralism doesn't change it.
  - `MULT`: each mining tech × 1.15.
  - Heavy Plow, Wheelbarrow and Hand Cart only change the farm rate.
- **Eco techs:**
  - `UPS`: `dba bs tms hp wb hc fl gn gm gsm sm ssm hcol crop` (`fl` Fishing Lines, `gn` Gillnets).
  - `REQUIRES`: `bs→dba`, `tms→bs`, `gsm→gm`, `ssm→sm`, `hc→wb`, `hp→hcol`, `crop→hp`, `gn→fl`. `setUp()` cascades in both directions: ticking a tech ticks its prerequisite, and unticking one unticks what depends on it.
  - `TECH_AGE` is 1-based. `TECH_AGE_CIV` gives Burgundians every eco upgrade one age early. Fishing Lines is Feudal and Gillnets Castle (Burgundians Dark / Feudal).
  - **aoe2techtree check (2026-09-29):** all 56 civs have the Dock, the Fishing Ship (Dark Age), Fishing Lines and Gillnets, so neither tech is in `NO_TECH`. Source: `data/data.json` (`civs[civ].Tech` / `.Unit`) and the per-civ files `data/trees/<CIVNAME in capitals>.json`, which give each node's `age_id` and `node_status`. aoe2techtree still names them Incas and Mayans. The same lists match `NO_TECH` for Two-Man Saw, Crop Rotation, Heavy Plow and Horse Collar exactly.
  - No unique tech or team bonus changes what Fishing Ships gather. Carry capacity (Dravidians +15, the techs' +5) isn't modelled. Varangian food gold covers Fishing Ships (5%, every spot, Fish Traps included) as well as villagers (10%); the user restated both shares on 2026-09-30, unchanged.
  - `NO_TECH`, the civs missing each tech:

    | Tech | Civs missing it |
    |---|---|
    | Two-Man Saw | 20 |
    | Gold Shaft Mining | 13 |
    | Stone Shaft Mining | 19 |
    | Crop Rotation | 19 |

- **`NICHE`** (1359), rates per minute:

  | Resource | Source | Rate | Notes |
  |---|---|---|---|
  | Wood | Feitoria | 42 | |
  | Food | Feitoria | 97 | |
  | Food | Garrisoned Sheep (Gurjaras) | 6 | Garrisoned Cows: 8 |
  | Gold | Feitoria | 60 | |
  | Stone | Feitoria | 18 | |

  The Feitoria is Portuguese, available from Imperial, and survives Full Tech Tree as a unique building.
- **`FX`** (1462, plus 1509–1514 for five civs whose only entries are team bonuses): the civ and team bonuses that move the numbers.
  - Kinds: `cost`, `flat`, `time`, `rate`, `farm` (farm table), `swap`, `unlock` (ally-unlocked units), `relicgold` (Aztec team bonus), `farmfood` (Maya, Sicilians, Chinese team bonus), `ageup` (age research speed; only Transition Eco uses it: Malay 66, Portuguese 25, Persians [5,10,15,20]), `agebonus` (resources received on every age-up, added to Transition Eco's totals: Dravidians +200 wood, Ethiopians +100 food +100 gold), `forage` (Portuguese: 5 wood per minute per forager, 2026-09-30; shown in both tabs' bonus lines). `team()` marks team bonuses. Persians' `ageup` has its own label ("Town Centers research Ages {p}% faster"). Their Town Center `time` effect is a Villagers Required bonus.
  - 4 civs have no entry and show "No applicable civ bonuses.": Bengalis, Jurchens, Tatars and Tupi (Varangians got `foodgold`, Shu `woodfood`, Poles `stonegold`, Burmese and Vietnamese `freeups`, Georgians `church`, Saxons the TCs/Castles discount on 2026-09-30). Vietnamese' other eco bonus (upgrades cost no wood, research faster) isn't modelled: tech costs and research times aren't.
- **Tuntian** (Wei, Castle Age; user, 2026-09-30): unique-tech checkbox, desc "Soldiers passively produce food" (aoe2techtree). While it's ticked and in effect, the Food box shows a "Soldiers" count under the upgrades (`#soldiers`, a −/+ like Relics, 0–999 = `SOLDIERS_MAX`, default 0, `state.soldiers`, saved, Clear All resets it, `isCleared()` checks it) and the food per minute ("18f/min"). 1.8 food per soldier per minute (`soldierfood`), taken off the food total right after the relics, before Farm Reseeding ("after 10 soldiers cover 0.89"). The row exists only for Wei (not Full Tech Tree; `.gone` otherwise) and keeps its space while Tuntian is off or before Castle Age, so ticking moves nothing; hidden in Transition Eco like Relics (and Tuntian isn't listed there). The food figure has a fixed minimum width (4.1em; "1798f/min" = 4.05em) and the row may wrap in every layout, so its height depends on the box, not the number: it wraps under the count at 1,920 (the box has 169 px, the line needs 178), 1,250, 600 and phones, one line at 768–1,000 and 915×412. Wei's settings row: +37 px at 1,920, +18 at 768. Relics and Soldiers share `counter(box, key, max)` (was the Relics-only `setRelics` and handlers); the CSS lists `#relics, #soldiers` side by side (same specificity as before). Checked at 13 sizes: same height at 0 / 10 / 999 soldiers and with Tuntian off, no overflow; Britons unchanged.
- **`UT`** (1518): 24 unique techs across 23 civs (Franks have two) that change cost, train time, gather rate, income (Paper Money, Burgundian Vineyards, Tuntian, 2026-09-30) or (Mahayana, Aznauri Cavalry, added 2026-09-29) population space. Grand Trunk Road is the gather-rate one (gold +10%).
- **Burgundian Vineyards** (Burgundians, Castle Age; user, 2026-09-30): replaces the "Burgundian Farmer" gold source (old saves fall back to Gold Mine), built like Paper Money. Desc "Farmers slowly generate gold in addition to food" (aoe2techtree). Effect `farmgold` (`farmGoldEach()`), gold per farmer per minute by the best of Wheelbarrow / Hand Cart: 0.8, 0.9 with Wheelbarrow, 1 with Hand Cart (user), only while Farm is the food source. The Gold box's "+x g/min from Farmers" line (`#farm-gold`, the other gold lines' place; exists while the civ has the tech, `hasTechFx("farmgold")`), x = farmers (the food total rounded up, after relic food) × gold each, taken off the gold total ("after 12 farmers cover 0.42"). Listed in Transition Eco (`teTech()`), where it counts when Researching Imperial: 10 farmers × 0.8 × 190 s = 25 gold. Checked: Knight-line + Villager, 12 farmers +9.6 g/min; Wheelbarrow 11 farmers +9.9; Hand Cart 11 farmers +11. The settings row doesn't change with the tick, food source or relics; with both the relic food line (Food box) and this line (Gold box), Burgundians' row is 10–18 px taller than before at 600 px, 412 px, 360 px and 640×360, where the Gold box is now the taller.
- **Paper Money** (Vietnamese, Imperial; user, 2026-09-30): replaces the "Vietnamese Lumberjack" gold source (old saves fall back to Gold Mine). A unique-tech checkbox in the Technologies panel's Imperial slot, desc "Lumberjacks slowly generate gold in addition to wood" (aoe2techtree). Its effect `woodgold` (`woodGoldShare()`): 3.55% of the lumberjack rate (user, 2026-09-30; upgrades and wood bonuses included, like Shu food; was a flat 0.8 per minute), so 0.83 g/min unupgraded and 1.30 with Two-Man Saw, only while Trees is the wood source. Works like Polish stone gold: the Gold box's "+x g/min from Lumberjacks" line (`#wood-gold`, same place; it exists while the civ has the tech, `hasWoodGold()`, so ticking moves nothing; hidden until ticked and in Imperial), x = lumberjacks (the wood total rounded up) × gold each, taken off the gold total ("after 27 lumberjacks cover 0.98"). Not listed in Transition Eco (not a `rate` tech, and Imperial is never the age being left). Checked: 5 Knight-line + 5 Crossbowman / Arbalester, 27 lumberjacks, +22.3 g/min, gold 54.82 → 53.85; with Two-Man Saw 18 lumberjacks, +23.4 g/min, 53.80; Gold box height unchanged by the tick at 13 sizes. Ticking any unique tech on phones (412 px) moves the table 19 px (Clear Techs appears in the strip); found while checking, true of Goths' Perfusion too, not changed. `ut()` takes an optional fifth argument of extra properties; `naval: true` (Circumnavigation) lists it and puts it into effect only while Naval is ticked.
- **Conscription** (Imperial): the code applies it to Barracks, Archery Range, Stable, Castle and Donjon. Its tooltip ("Military buildings except Siege Workshops") is broader, since Docks aren't included, so check the game before "fixing" either one.
- **Treadmill Crane** (Castle): builders work 20% faster on every building except the Fish Trap (`VIL_BLD`; user, 2026-09-30). `CRANE_MISSING` lists the 20 civs without it.
- **Malay Fish Traps** (2026-09-30, aoe2techtree "Fish Traps cost -33% and provide +200% food"): two labels, "Fish Traps cost −33%" (`cost`) and "Fish Traps provide +200% food" (`trapfood`, with a Fish Trap matcher `m` so `buildingsOnly()` treats it as a building bonus). Both are listed while Buildings is ticked, a Fish Trap row is in the table, or Fish Trap Rebuilding is in effect (`inReach()`).

## Code map
Line numbers are for the 2,927-line version.
- **File layout:** CSS 10–677 · markup 679–860 · global data tables 862–1335 · app IIFE 1336–2924.
- **Usual refresh chain:**
  1. `syncSettings()` (source dropdowns, eco boxes)
  2. `renderTechs()`
  3. `renderCivFx()`
  4. `renderRows()` (table HTML, then `update()` for the numbers)
  5. `fit()` (`applyLayout()` classes, then the font shrink)
  6. `save()` (`syncButtons()`, then localStorage)

  `refreshAll()` (2827) runs the whole chain. A count or source change only needs `update()` + `save()`.
- **Key functions:**

  | Function | Line |
  |---|---|
  | `isCleared` / `anyTechOn` | 1587 / 1593 |
  | `load` | 1608 |
  | `activeFx` | 1666 |
  | `ageOf` | 1701 |
  | `effective` | 1714 |
  | `rates` | 1745 |
  | `nicheAvail` | 1785 |
  | `postImpView` | 1825 |
  | `upgradeOf` | 1864 |
  | `costMeta` | 1900 |
  | `reseed` | 1936 |
  | `update` | 1959 |
  | `renderRows` | 2082 |
  | `applyLayout` | 2134 |
  | `fit` | 2150 |
  | `addUnit` | 2168 |
  | `techOpen` / `on` | 2295 |
  | `syncSettings` | 2322 |
  | `renderCivFx` | 2386 |
  | `renderTechs` | 2410 |
  | `setCiv` | 2455 |
  | `syncFtt` (the Full Tech Tree / Include Regional & Unique swap) | 2447 |
  | `combo` | 2469 |
  | `renderSources` (the income dropdowns) | 2300 |
  | `groupName` (the Fortified Church heading) | 2614 |
  | `fitHeader` (header arrangement) | 2112 |
  | `fitFish` (fishing radios or split dropdown), `laneShown` (upgrade shown for the source), `forageEach`, `navalOk`, `fillRows` / `placeGrand` (filler rows, Villagers required level with Totals) | 2026-09-30, grep |
  | `ageUp` / `updateTE` (Transition Eco) | 2012 |
  | `setTab` | 2779 |
  | Clear All handler | 2844 |
  | `postImpTechs` / `setPostImp` | 2878 / 2885 |

## Conventions & gotchas
- **Two age indexings.**
  - `state.age` is 0-based: 0 = Dark … 3 = Imperial, and 3 is the default.
  - Every table age is 1-based: `UNITS[8]`/`[9]`, `TECH_AGE`, `UT[].age`, `NICHE[].age`, `CRANE_AGE`, `CONSCRIPTION_AGE`.
  - `ageOpen(a) => a <= state.age + 1` bridges the two. Messages use `AGES[a - 1]`, and per-age `p` arrays are indexed by `state.age`.
- **`fttOn()` vs `state.ftt`.**
  - `fttOn()` means ticked *and* not Generic.
  - `hasTech`, `hasCrane`, `ageOf`, `nicheAvail`, `renderTechs` and `anyTechOn` read `state.ftt` directly.
  - That's only safe because `setCiv()` and startup force `state.ftt = false` for Generic. Keep that invariant.
- **Free techs** (user, 2026-09-30; `freeups` FX, aoe2techtree wording): Bohemians "Mining Camp technologies free" (Gold Mining, Gold Shaft Mining, Stone Mining, Stone Shaft Mining), Burmese "Lumber Camp technologies free" (Double-Bit Axe, Bow Saw, Two-Man Saw; Burmese wood 27.9 Feudal, 33.2 Castle, 36.6 Imperial), Vikings "Wheelbarrow, Hand Cart free" (farm 23.0 Feudal, 23.9 Castle+, 24.0 with Heavy Plow; aoe2techtree's other Vikings bonuses: Warships and team Docks were already in, Infantry +20% HP is combat), Franks "Farm upgrades free" (user, 2026-10-03: "Mill upgrades mandatory for Franks"; aoe2techtree "Farm upgrades free (require Mill)", the Mill isn't modelled: Horse Collar from Feudal, Heavy Plow from Castle, Crop Rotation from Imperial; Heavy Plow's box in the Food box and all three in the Farm Reseeding row, whose `reseedRowHtml()` now sets `fixed` / `disabled` for a free tech itself; checked: Franks Imperial farm 21.2/min, reseeding 720w / 1571.6 s, the same as the three ticked by hand; under Full Tech Tree they're ordinary boxes again), Vietnamese "Conscription free" (user: "not an optional tech for Vietnamese unless Full Tech Tree"; `ups: ["consc"]`, handled in `activeFx()`, `renderTechs()` and `anyTechOn()` since Conscription isn't in `UPS`; left out of Transition Eco's bonus line, `relevant()`; Knight-line 30 → 22.6 s in Imperial). While that civ (not Full Tech Tree), they count as ticked from their age (`freeUp(k)`; `on(k)` = (stored tick or free) and open). Their boxes are ticked and disabled (`.opt.fixed`, normal label, default cursor; tooltip "Free for Bohemians. …"); before their age they're greyed as usual. The stored tick isn't touched, so another civ gets the user's own ticks back; `anyTechOn()` ignores free ticks (Clear Techs stays hidden), `isCleared()` reads stored ticks. In `TE_KINDS`. Checked: Feudal gold 26.2 / stone 24.8, Castle+ 30.2 / 28.6, Transition Eco Researching Castle 10 gold miners = 699.
- **Remembered ticks.**
  - A tick locked by age is kept, just filtered: `on(k)` = stored tick && `techOpen(k)`. It comes back in a later age.
  - A tick for a tech the civ *lacks* is erased: `syncSettings()` does it for eco techs, `renderTechs()` for Treadmill Crane. Switching back leaves the box unticked.
  - Unique-tech ticks (`state.uts`, stored by name) survive civ switches and Full Tech Tree.
  - Clear Techs erases everything, age-locked ticks included.
- **Post-Imp cancelling.** `setPostImp(false)` runs on any tech untick and when leaving Imperial.
- **The Post-Imp view is cached.** `postImpView()` is memoised in `piView`. `renderRows()` and `setPostImp()` call `invalidateView()`, and anything else that changes availability must too.
- **Row identity** is the entry name `UNITS[i][0]`, used in `data-name`, `state.rows[].name` and `byName`.
  - `displayName(i)` is for display only, and it differs under Post-Imp.
  - The dropdown also drops a "(Group)" suffix under that group's own heading, so "Serjeant (Donjon)" shows as "Serjeant".
- **`update()` reads rows from the DOM** (`tr[data-name]`), so render before updating.
- **Farm Reseeding isn't in `state.rows`.** `renderRows()` appends it (and House Building) and `reseed()` shows or hides it. Its checkboxes use `data-up`, so they share state with the Food lane.
  - `renderRows()` runs after `syncSettings()` in every refresh chain and rebuilds the row, so `reseedRowHtml()` must set each box's `disabled`, `na` (greyed) and tooltip itself. Fixed 2026-09-29: Crop Rotation for civs without it (e.g. Dravidians) was disabled but not greyed after Post-Imp, a civ or age change, or on load.
- **`addUnit()` is speculative.** It pushes the row, renders, and takes that row back out (by reference, not `pop()`, since `renderRows()` may have moved it before the buildings; `addCustom()` likewise) if `fit()` fails. Incrementing a row, or replacing it with another version of its line, doesn't re-check the fit.
- **Layout is driven by JS.** `applyLayout()` toggles four classes on `<html>`:

  | Class | When |
  |---|---|
  | `html.mid` | ≤ 1320 px, or ≤ 1600 px while # Builders shows |
  | `html.narrow` | ≤ 1150 px, or ≤ 1320 px while # Builders shows |
  | `html.show-bld` | the # Builders column is visible |
  | `html.no-stone` | the Stone column is blanked |

  - The "Mid" and "Narrow" CSS blocks (≈507–537) are top-level rules keyed on those classes, even though they're indented like media-query bodies. The `@media (max-width: 820px)` and `(max-width: 599px)` blocks refine `html.narrow`.
- **One shared column grid.**
  - `.lanes`, `.lane-bg` and the table's `<col>`s all use `--cols` and the `--w-*` widths.
  - **The four resource columns are always the same width** (user's rule). `--w-wood/food/gold/stone` all read `--w-res`: 12 rem wide, 11 rem mid. The measured minimum is 10.9 / 10.2 rem, set by the longest income options (then "Gurjara Livestock (large)", "Vietnamese Lumberjack"). The user asked for a little extra so the resource block sits nearer the page centre. The narrow and phone layouts set all four to one value directly. Change `--w-res` rather than one resource.
  - The Technologies panel is absolutely positioned over the last column (`--w-tech`), and the table is `calc(100% - var(--w-tech))` wide.
  - Resize columns through the variables.
- **Hiding without losing space.** `.pagebtn[hidden]` and `.dockopt[hidden]` use `visibility: hidden`, and `html.no-stone` blanks the Stone column. The # Builders column collapses to 0 width instead, so toggling Buildings is the one action that still shifts columns. The user confirmed that's intended.
- **Shared checkbox slot.** Full Tech Tree and Include Regional & Unique share `.opt-slot`, an inline grid with both labels stacked in one cell. The slot is always as wide as the longer label (≈151 px), so switching Generic ↔ civ moves nothing.
  - **Wrap bands:** with Buildings off, the checkbox line doesn't fit between ~1,151 and ~1,245 px wide (mid layout) or between ~1,321 and ~1,329 px (wide layout). There it wraps onto two lines, and the settings row is 22 px taller. Widening the resource columns (`--w-res`) takes room from the adder and widens these bands.
  - The wrap is the same for every civ at a given width. Headless-Chrome checks at 20 widths (360–1,920 px) found no element moving on the swap.
- **Transition Eco state and CSS.**
  - `state.tab` is "vr" or "te". It's saved, but `load()` ignores it, so every visit opens on Villagers Required. `state.te = { research: 1–4, after: 0–353693, vils: { wood, food, gold, stone } }` (4 = "After [] seconds").
  - `inTE()` and `curAge()` bridge the two tabs: `ageOpen`, `pct`, `available` and `nicheAvail` all use `curAge()`.
  - `applyLayout()` toggles `html.tab-te`. All show/hide is CSS keyed on it (end of the stylesheet), and the Transition Eco row is `#te-row` in the tfoot.
  - Switching tabs or changing the Researching radio calls `refreshAll()`, because the age changes.
- **Header arrangement is measured.** `fitHeader()` (from `applyLayout()`) tries none → `hdr-b` → `hdr-b hdr-c` → `hdr-b hdr-c hdr-d`. It measures under `hdr-measure` (items at natural width, wrapping) and keeps the first arrangement whose line holds its items. The old `html.mid .civpick` and phone picker rules became `hdr-*`.
  - Results: one line at ≥ ~1,450 px, `hdr-b` down to ~1,151, `hdr-c` on tablets, `hdr-d` at ~600 px and on phones.
  - This also fixed the old 600–700 px overlap of Clear Civs and the allies picker.
- **Duplicated controls.** Post-Imp and Clear Techs each exist twice (`.postimp-box`, `.clear-techs`): once in the settings row and once in the Technologies panel for narrow layouts. The handlers keep both in sync.
- **Search** runs text through `norm()`, which lowercases it and collapses hyphens and whitespace. Every token must match, in any order. Under Post-Imp an entry is found by every name in its line, so "eagle scout" finds Elite Eagle Warrior.
- **Persistence.**
  - The key is `villagers-required.v3`, falling back to v2 and then v1. The age is restored only from a v3 save.
  - Old row names ("Militia line", or member names like "Champion") are mapped through `ALIAS`.
  - Duplicate rows from one line are merged at startup.

## Pending
- **Villagers Required on phones held sideways** (found 2026-09-28): at 640×360 to 915×412 no table row fits. The adder plus the resource boxes (240 px) fill the height. This predates the day's changes. It needs a design decision (e.g. a different short-window arrangement); the user hasn't been asked yet.

## Settled after the chat (user, 2026-09-28)
- **Builders** (output 36): every builder counts toward Villagers required, including the default 1 per building. That's the current behaviour; keep it. Exceptions: Farms (2026-09-29) and Fish Traps (2026-09-30) count no builders at all.
- **# Builders space** (output 38): the column stays as it is. It collapses when hidden, and toggling Buildings may shift columns. Don't change anything about it.
- **Regional units** (output 12): showing them for every civ under Full Tech Tree is intended.
- **Warrior Priest** (output 22): no separate heading. Instead the Monastery heading reads "Fortified Church" while Armenians or Georgians is selected, with or without Full Tech Tree (`groupName()`).
- **Grouping** (output 6): the Monk and Trade Cart groups and the Economic/Military building split are fine as they are.
- **Technologies heading** (output 15): leave it out. Tech checkboxes appear all over the page, so a heading on just this panel is redundant and misleading.
- **Decimals** (output 3): gather rates and bonus-shortened times keep 1 decimal.

## Known limitations & modelling choices
- **Raw gather rates.** Walking and drop-off distance aren't modelled, so carry-capacity bonuses (Goth hunters, Dravidian fishermen) change nothing. Wheelbarrow and Hand Cart only affect farms, as in the original tool.
- **Deliberately left out:**
  - combat, HP, line-of-sight and population effects
  - research speed and repair
  - situational bonuses (none left: Georgian Fortified Churches and the Saxon Foot Soldier discount per Town Center / Castle are in since 2026-09-30, see UI & behaviour)
  - lump sums (resources on age-up or when a building is placed, and one-off population space)
  - trade income (the Bengali and Spanish team bonuses)
  - the Poles' Folwark
  - (the Wei's Tuntian: in since 2026-09-30, 1.8 food per soldier per minute)
  - what Kasbah and Butalmapu do for allies
- **Feitoria counting.** One Feitoria yields all four resources at once, so if it's picked for several resources, the real need is the largest of those columns, not their sum.
- **Double-counted villagers** (no longer applies, 2026-09-30): villager-based alternative sources (Burgundian Farmer, Vietnamese Lumberjack, Polish Stone Miner gold) were counted apart from the same villagers' main job. They're now extra-income lines that reduce the other resource's gatherers instead.
- **Units trained at two buildings.** Huskarl (Barracks) and Tarkan (Stable) are separate entries so that the building's team bonus applies. The Donjon Serjeant is the only such unit whose train time differs: 16 s, against 12 s at the Castle.
- **Farm Reseeding** assumes one farm per farmer, rebuilt by that farmer (1 builder). Pastures: two villagers each, rebuilt at the 1-builder time; an odd farmer counts as a whole pasture at the 2-villager cycle.
- **Fish Trap Rebuilding** assumes one trap per Fishing Ship, rebuilt by that ship.
- **Snapshot data.** Everything is a snapshot (aoe2techtree as of 2026-09-22, plus the hand-measured rates), so a balance patch silently makes the tables wrong.

## Repo state (2026-09-28)
- **Remote:** `origin` is github.com/cnordenb/aoe2vils, branch `main`.
- **Commits:** just one, `e03199e` "Initial commit" (LICENSE and a one-line README).
- **Untracked:** `index.html` and `.gitignore`, so the app isn't committed yet. `.gitignore` lists `context_a.md` and `context_b.md`.
- **App history:** git has none, so `context_a.md` serves as the change history, prompt by prompt.

## Reading context_a.md
- **Numbering.** It holds 42 prompt/response pairs labelled 1–44. The labels skip twice ("prompt 11" → "output 12", "prompt 34" → "output 35"), but each output answers the prompt just above it, so nothing is missing.
- **No code.** The outputs are summaries and tool calls are collapsed ("Ran N commands"), so no diffs are recorded.
- **When exchanges contradict each other, the later one wins.** Notable reversals:

  | Topic | Change |
  |---|---|
  | Food source | radio buttons → dropdown (43) |
  | Horse Collar / Crop Rotation | left out → added with Farm Reseeding (42) |
  | Full Tech Tree for Generic | shown disabled (21) → hidden (24) |
  | Full Tech Tree ages | civ keeps its early ages (12) → standard ages, no bonuses (23) |
  | Clear | with Undo (17) → no Undo (18) → no "Page cleared" (19) |
  | Clear All vs the view toggles | reset them (17) → leaves them alone (41) |
  | Post-Imp | button (17) → checkbox (27) |
  | Conscription | skipped the Donjon (22) → includes it (40) |
  | Treadmill Crane | shown when a building row exists (26) → when Buildings is ticked (40) |
  | Adder and column labels | switched with Buildings (36) → fixed (37) → switch again (40) |
  | Stone column | collapsed (37) → blanked in place (38) |
  | Unique techs | 18 → 19 (35) |

- **Stale numbers.** Examples from before output 42 leave out the Farm Reseeding lumberjacks.
- **Where the log ends.** Its last exchange (44: total wood in Farm Reseeding, plus the Niche Incomes checkbox, since removed) is the state `index.html` arrived in. Later changes come from Claude Code sessions and are listed below.

## Changes since the chat (Claude Code)
- **2026-10-03** (user): Franks' farm upgrades are free (locked on, like Bohemians' Mining Camp techs; see Free techs under Conventions & gotchas). The table lists units before buildings (see Row order under UI & behaviour). Checked in headless Chrome: Villager, Farm, Knight, Castle added through the dropdown show as Villager, Knight-line, Farm, Castle, in the save too; custom rows follow their kind; removing a row focuses the next one; Franks Feudal / Castle / Imperial / Full Tech Tree, Transition Eco; no errors.
- **2026-10-01, late** (user): After [] seconds field half as wide (2.15em, widening for 4+ digits), its slider 0–900 s. Checked at 15 sizes against the previous version: nothing worse; the slider is wider at several sizes, fits beside the label at 1,160×700, and the row is 17–18 px shorter on 412 / 360 px phones.
- **2026-10-01, Gill Sans** (user asked about Firefox's console warning "Request for font "Gill Sans" blocked at visibility level 3 (requires 4)"): Firefox's fingerprinting protection only lets pages use some local fonts; Gill Sans (installed here with Microsoft Office, next to Gill Sans MT) isn't one at its default level, so every lookup was refused and logged. The body stack only reaches past Alegreya Sans for glyphs it lacks (✓ in the allies list, → on custom rows; U+2192 isn't in Google's Latin subset), and Gill Sans was a look-alike for when Google Fonts failed, so it was dropped: `"Alegreya Sans", "Segoe UI", system-ui, sans-serif`. Checked: headless Firefox drew a "Gill Sans" test text in Segoe UI (blocked, as reported); Chrome layout identical at 15 sizes × 3 setups × 2 tabs, ✓ and → unchanged; the page renders in the embedded fonts in headless Firefox. Firefox's warning goes to the web console only (not `MOZ_LOG=fontlist`), so it couldn't be read headlessly.
- **2026-10-01, late night** (user): "6 HTTP requests, want 2": the live site (GitHub Pages, `CNAME` aoe2vils.net, served through Cloudflare, no injected scripts) showed the page, `favicon.ico` (the user added it) and the four `data:` fonts in DevTools. The fonts moved from `@font-face` to `FontFace` objects; checked: served over HTTP locally, exactly 2 requests; fonts load; layout identical at 15 sizes × 3 setups × both tabs.
- **2026-10-01, night** (user): fonts embedded, no more Google Fonts requests (user chose embedding over system fonts). Checked: no resource requests at all, the four faces load, and layouts identical to the hosted fonts at 15 sizes × 3 setups × both tabs (6 of 90 cases differ by under 0.5 px in Villagers required's position, `placeGrand()` rounding). To redo: fetch `https://fonts.googleapis.com/css2?family=Alegreya:wght@700&family=Alegreya+Sans:wght@400;500;700` with a Chrome user agent (else Google serves TTF), take the `/* latin */` blocks' woff2 URLs.
- **2026-10-01, evening** (user): Transition Eco amounts can be typed in, setting the villager counts (see the Transition Eco tab). Also the user's own commit "name change": the page heading (`<h1>`) and `<title>` read "aoe2vils.net" (were "Villagers required" / "Villagers required · AoE2 DE"); the grand total's label "Villagers required" is unchanged.
- **2026-10-01, last** (user): After [] seconds goes up to 98h14m53s (heading with hours), with a 0–300 s slider while it's picked; Idle TC Time's time can be typed, up to 59:59, the slider back to 00:00–10:00 (">10:00" / "24+" gone).
- **2026-10-01, later** (user): Transition Eco's fourth radio "After [] seconds" (0–3,600, default 300), always in Imperial Age (user's answer), heading "After 5m30s" etc. See the Transition Eco tab under UI & behaviour.
- **2026-10-01** (user): Idle TC Time checkbox, Villagers missing and the idle-time slider under Villagers required, wide layouts only (see Technologies panel under UI & behaviour). The user's answers: 25 s per villager whatever the civ; not added to Villagers required; Clear All resets both the box and the slider.
- **2026-09-30, last** (user): Fish Trap Rebuilding (see UI & behaviour and How a number is computed); Fish Trap build time by upgrade and the Japanese Fishing Ship bonus, without Treadmill Crane or villager builder bonuses; Fish Traps count no builders, like Farms; Malay Fish Trap bonus added; an idle Farm row's "25s" box greys out too; custom buildings allow 0 builders; the Fish Trap row got a "One per Fishing Ship" box like the Farm's; the Trap spot is locked in Dark Age; Paper Money unique tech replaces the Vietnamese Lumberjack source (3.55% of the lumberjack rate); Burgundian relic food shown in the Food box; naval bonuses hidden while Naval is off; Burgundian Vineyards unique tech replaces the Burgundian Farmer source; Camel Scout named and listed in Feudal Age; Eagle Scout training time by age (40 s Feudal, 30 s after); no Upgrade button on Serjeant (Donjon); Armenian Mule Cart technology effects +40%; Bohemian Mining Camp, Burmese Lumber Camp and Vikings' Wheelbarrow / Hand Cart technologies and Vietnamese Conscription free (locked on); Georgian "Near Fortified Church" boxes (+10% per resource); Vikings' free Wheelbarrow / Hand Cart; Wei Tuntian with a Soldiers count in the Food box; Saxons' TCs/Castles count and Foot Soldier discount; Generic's Feitoria / garrisoned livestock only with Include Regional & Unique; Kamandaran Archer-line 50w (was 70w).
- **2026-09-30, later** (user): Polish stone gold replaces the Polish Stone Miner source (see the rule above); Fish Trap rates 20.9 / 23 / 25.3 (the Trap radio's tooltip lost "(shore fish rate for now)"). Headless-Chrome helper: `cdp-lib.mjs` in the session scratchpad, run with `node --experimental-websocket` (Node 21 has no global WebSocket otherwise); Chrome is at `C:/Program Files (x86)/Google/Chrome/Application/chrome.exe`. **Disk:** each run makes a Chrome profile `%TEMP%\cdp-*` (~45 MB); the old helper never removed them, and 135 of them (6 GB, 28–30 Sep) filled the C: drive on 2026-09-30 (edits failed with ENOSPC; `index.html` was unharmed). They were deleted, and the helper now removes earlier runs' profiles (older than 10 min) when it starts; a detached cleanup after exit doesn't survive the tool's process teardown. Copy that version of the helper into a new session's scratchpad.
  - To compare with the last commit, write it with a byte-safe redirect: `cmd /c "git show HEAD:index.html > <file>"` (PowerShell's pipe re-encodes; `git show --output` doesn't apply to a blob), then run the helper with `PAGE=<file>`.
- **2026-09-30** (user). Checked in headless Chrome against HEAD at 23 sizes × 3 food sources × both tabs: no box or table moved except at 360×640, where the removed Food indent makes the boxes 6 px shorter.
  - Farm Reseeding / House Building tooltips reworded (see the wording list).
  - Fishing Ship: Shore / Deep / Trap radios, deep-sea rates, split dropdown where they don't fit; Fishing Lines / Gillnets moved up to Heavy Plow's / Wheelbarrow's lines.
  - Food box: the indented line left of the upgrades removed.
  - Upgrades a source doesn't use are hidden, not greyed. Clear Techs counts only ticks whose box shows (`anyTechOn()` uses `laneShown()`).
  - Portuguese Forager: no longer a wood source (old saves fall back to Lumberjack); a Portuguese `forage` bonus and the Wood box's Forager line instead.
  - "Include Dock" renamed "Naval"; Circumnavigation only while it's ticked; Shipwright checkbox added.
  - Wide layout: fixed technology places, at least four table rows (filler rows), Villagers required level with the Totals row. Narrow layouts, phones and Transition Eco unchanged (same regression check).
  - Renames: "Custom Rate", "Stone Mine", "Garrisoned Sheep" / "Garrisoned Cows". Shu lumberjack food replaces the Shu Lumberjack source; `fitIncome()` gained a three-line mode. Post-Imp upgrades table rows. Custom unit maximum 999, building 9999.
  - Custom units (see UI & behaviour). Single-select lists (`combo()`) now honour a disabled option too.
  - Sources renamed Trees / Gold Mine; column headings by source with `fitHeads()`; Varangian food gold replaces their three gold sources; extra-income lines sized by `fitIncome()`. Regression against HEAD: unchanged at every size except 360×640 (Food indent, 6 px more table) and 900×700 (Britons' bonus line one line shorter with Buildings off, 20 px more table).
  - Civ and team bonus labels rewritten to aoe2techtree's wording (see the rule above). Portuguese "Technologies research +25% faster" is now a team bonus, as aoe2techtree lists it (user: a bonus that moves the numbers as a civ bonus moves them as a team bonus too); every other modelled team bonus already matched. Checked: Britons + Portuguese ally research Feudal in 104 s, Malay + Portuguese ally 62.7 s, no double count with a Portuguese ally. Unique tech descriptions left as they were.
- **2026-09-29: Civ names** "Mayans" → "Maya", "Incas" → "Inca" (user). `CIV_NAMES` keeps its order, so the per-civ age strings are unchanged. `load()` maps the old names in saved civ and allies.
- **2026-09-29: Castle order.** Unique units now come before Trebuchet and Petard in the dropdown (moved in `UNITS`).
- **2026-09-29: Idle rows greyed** (user): a row's name, cost and time grey out while its # Buildings is 0.
- **2026-09-29: Farm row** (user): no # Builders stepper and an empty # Builders cell. A Farm counts **0 builders** (the builder is always a villager moved over or a new one who will farm it), so it adds nothing to the builders total or Villagers required; its time is still the one-builder build time (old saves are clamped to 1). `FARM = named("Farm")`. A checkbox labelled "25s" (the Villager training time; user renamed it from "One per Villager (25s)") on the name's line after the cost ("Farm 60w / 15s  ☐ 25s"; user; right after the name in narrow layouts, which hide the cost), so the row is no taller than the others (`.row-opts`, inline; its input needs `font: inherit` or its em size uses the control's own 13.3px; checked 1,920 to 360 px wide, including Persians' "22.7s") (`row.perVil`, saved with the row, default off) sets the row's time to the Villager's training time instead of the Farm's build time: 60w every 25 s rather than 15 s. The time follows `villagerTime()` = `effective(Villager).t`, so Persian Town Center bonuses shorten it (20.8 s in Imperial) and the label shows the actual figure; build-speed bonuses (Spanish, Treadmill Crane) drop out of the tooltip, cost bonuses (Teutons) stay. `effective()` now also returns `timeFx`, the applied time labels. Farm Reseeding is unaffected (it still uses the build time).
- **2026-09-29: Automatic rows look like unit rows** (user): Farm Reseeding and House Building lost their wood-tinted background; only their names stay italic.
- **2026-09-29: Row cells show one decimal** (e.g. 0.6 instead of 1); only the totals round up to whole villagers (user). Checked from 1,920×1,080 to 360×640 with values up to "1989.8": no cell overflows.
- **2026-09-29: Farm Reseeding / House Building opt-outs** (user): two checkboxes in the adder's footer, both on by default and saved; unticking one hides its row and its lumberjacks. Preferences like Include Dock (now Naval), so Clear All leaves them. Adder geometry unchanged at 10 sizes; the footer no longer grows 1 px when Clear table appears.
- **2026-09-29: House Building row** (user). See UI & behaviour and How a number is computed. New FX kinds `nohouses` (Huns), `housepop` (Inca) and `pop` (the two new unique techs). Checked by hand-calculation in headless Chrome for Britons, Aztecs, Inca, Huns (± Full Tech Tree, as ally), Wu (own and ally), Spanish, Treadmill Crane, Malay Karambits, Mahayana (incl. locked in Castle) and Aznauri Cavalry.
- **2026-09-29: Relic count** replaces the Relic gold source and the Burgundian Relic food source (user). See UI & behaviour and How a number is computed. Old saves with either source fall back to the standard one. Burgundians now have an FX entry (`relicfood`, team bonus), so their bonus line shows it.
- **2026-09-29: Builder limits per building** (user). The limit follows the footprint: 1×1 → 14, 2×2 → 22, 3×3 → 33, 4×4 → 44, 5×5 → 53 (`BUILDERS_BY_SIZE`; the user corrected 1×1 from 4 then 8, 2×2 from 8 then 16, 3×3 from 24, 4×4 from 32 and 5×5 from 40). `SIZE` holds each building's footprint, from the game data (HSZemi/aoe2dat `full.json.xz`, DE update 147949, June 2025; footprint = 2 × `CollisionSize`). The user's choices for the odd ones:
  - Exceptions in `MAX_BUILDERS`: Gate and Palisade Gate (4×1) 38 (was 20); Farm 1, used only for its build time (see the Farm row entry: no builders counted, empty cell).
  - Pasture: goes by its 2×2 foundation (the finished pasture is 4×4).
  - Settlement (newer than that data): 3×3.
  - Result: 14 = Fish Trap, Mule Cart, Bombard Tower, Outpost, Palisade Wall, Stone/Fortified Wall, Watch Tower-line; 22 = House, Lumber Camp, Mill, Mining Camp, Pasture, Donjon; 33 = Dock, Farm, Folwark, Harbor, Settlement, Archery Range, Barracks, Blacksmith, Fortified Church, Krepost, Monastery, Stable; 38 = Gate, Palisade Gate; 44 = Caravanserai, Market, Town Center, Castle, Siege Workshop, University; 53 = Feitoria, Wonder (a Wonder with all 53: 3,500 s × 3 ÷ 55 = 190.9 s).
  - Fish Traps: since 2026-09-30 like Farms (user): no # Builders stepper, an empty cell, no builders counted anywhere (`NO_BUILDERS`, `MAX_BUILDERS` 1, old saves clamped to 1). Was: counted in the # Builders total but not in Villagers required. The Mule Cart is built by villagers (the data says so).
  - Steppers read their limits from their input's `min` / `max` (`limits()`; the old `FIELDS` table is gone). Saved builder counts are clamped on load.
- **2026-09-29: Lumberjack rates** (user): a per-minute table `LUMBER` instead of .39/s with multipliers. Shu Lumberjack food = 6.859% of the lumberjack rate (was a flat 1 per minute).
- **2026-09-29: Fishing Ship food source** (user's rates), with Fishing Lines and Gillnets swapped in for Wheelbarrow and Hand Cart; "Fishing" renamed "Fisherman". Japanese Fishing Ship bonus added; the Danes' drop-off bonus now says it covers Fishing Ships. Checked in headless Chrome at 8 sizes from 1,920×1,080 to 360×640: the Food box doesn't move and the table stays put (see UI & behaviour).
- **2026-09-29: Pasture gather rates** updated (user): 22.8 / 24.4 / 25.4 food per minute unupgraded / Wheelbarrow / Hand Cart.
- **2026-09-29: Pasture Reseeding** for Khitans (see the Farm Reseeding row and How a number is computed). Heavy Plow is invisible in the Food box for Khitans. Clear Techs counts `hp` like `hcol` / `crop` for Khitans (only while the reseed row shows).
- **2026-09-29: Xolotl Warrior** (user's spec). A regional Stable unit: 60 food, 75 gold, 30 s, flags `MSL`, from Castle Age for Aztecs, Inca, Maya, Mapuche, Muisca and Tupi. It's a standalone entry, not part of the Knight-line (no `EDGES`), placed after Hei Guang in the Stable group.
  - **No Stable, still listed.** Those six civs are exactly the ones that can't build a Stable (Stable's age string is 0 for them). The user said they can still train the Xolotl if they do have one, so it's in their dropdown from Castle Age with or without Full Tech Tree. Nothing in the app ties a unit to its building being available.
  - **Full Tech Tree:** as a regional unit it opens for every civ, like the other regional units (settled rule). The user wasn't asked whether that matches the game.
  - **Bonuses that reach it:** Aztec military train speed, Inca military food discount, the Huns' Stable team bonus, Conscription, Chivalry. Headless-Chrome check: Aztecs Imperial = 60f 75g / 26.1 s; Aztecs + Huns ally = 21.7 s; Inca Castle = 45f 75g; Generic lists it only with Include Regional & Unique.
- **2026-09-28: Include Regional & Unique** checkbox for Generic, in Full Tech Tree's spot (see UI & behaviour and `REGIONAL`).
- **2026-09-28: Monastery heading** reads "Fortified Church" for Armenians and Georgians.
- **2026-09-28: Short-window mode** for phones held sideways, so Transition Eco fits at every tested size and orientation (see UI & behaviour). The user decided Spanish age-ups earn no +20 gold.
- **2026-09-28: Bonus line filtered per tab.** Transition Eco lists only gathering and age-up bonuses (`TE_KINDS`), including allies' team bonuses and the allies picker's preview.
- **2026-09-28: Age-up resources in Transition Eco.** Dravidians +200 wood and Ethiopians +100 food +100 gold are added to the "On reaching" totals (`agebonus` FX).
- **2026-09-28: Transition Eco tab** (see UI & behaviour). The user's choices:
  - The Researching radio sets the age.
  - Civ age-up bonuses apply.
  - Villager counts use −/+ steppers.
  - The right panel shows only gather-rate unique techs.
  - Also: the header arrangement is now measured (fixing the 600–700 px overlap), and new `ageup` FX were added (Malay, Portuguese, Persians).
  - The tabs cost phones one table row: 360×640 = 2, 390×740 = 8, 412×780 = 11, 768×1024 = 18.
- **2026-09-28: Ally count.** The Allied Civilization(s) label shows "n/7".
- **2026-09-28: Status line removed.** `#status`, `say()` and all 8 messages are gone:
  - Unit added.
  - Unit: now N buildings.
  - New replaced Old.
  - Old upgraded to New.
  - Unit removed.
  - Table cleared.
  - The sheet is full. Remove a row to add another.
  - Up to 7 allies. Untick one to add another.
  - A full sheet now refuses a new row silently. The ally list already greys out civs at 7.
- **2026-09-28: No animations.** The user made it a strict standing rule. Removed the arrow's rotation transition and the 1 s highlight on added or upgraded table rows (`@keyframes flash`, `flash()`).
- **2026-09-28: Dropdown arrows work.** The search pickers' arrow opens and closes the list and points up while it's open. It replaces the old `.combo::after` decoration, which ignored clicks.
- **2026-09-28: Equal resource columns.** Wood, Food, Gold and Stone share `--w-res` (was 10 / 16 / 11 / 11.5 rem wide and 8.6 / 14 / 9.4 / 9.8 rem mid).
  - It was first set to the measured minimum (11 / 10.25 rem), then widened at the user's request to 12 / 11 rem so the block sits nearer the page centre.
  - At 1,920 px the block's centre is now 151 px right of the board's centre (183 px at 11 rem).
- **2026-09-28: Narrow layout keeps the resources side by side.** The four resource boxes now form one row under the unit adder, instead of Food taking a full-width row above Wood / Gold / Stone. Phones get the compaction described under UI & behaviour.
  - The settings area got shorter: 245–277 px on phones (was 295) and 253 px on tablets (was 342–380).
  - Rows that fit: 360×640 = 3, 390×740 = 9, 412×780 = 12, 768×1024 = 18.
  - The old mangled `.lanes` selector for phones is gone.
- **2026-09-28: Upgrades follow the source.** Wood, gold and stone eco upgrades are greyed out unless the standard gatherer is selected, like the farm upgrades for Food.
- **2026-09-28: Niche Incomes checkbox removed.** Alternative incomes appear in the source dropdowns whenever available. Relic gold is offered in every age. The food dropdown moved under the Food heading, and hidden dropdowns keep their space. Dropdowns are compact on phones.
  - The desktop settings row grew 4–12 px.


human input..:

TODO next:
- oysters still missing
- more testing to see if anything is missing (last detected was Frank mill bonus missing, 2026-10-03)
- trade cart and trade cog (gold income type)
- test if Khmer farms need separate rates