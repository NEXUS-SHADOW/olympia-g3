# Proposal: A Conserved-Gold Economy for Olympia G3

*Status: Design discussion. No game rules or engine behavior have been changed.*

*Related: [Issue #135 — Investigate MIMO and add a gold-supply audit](https://github.com/playbymail/olympia-g3/issues/135).*

## Summary

Generate a finite quantity of gold when the world is created. Thereafter, gold is neither created nor destroyed: every economic operation moves existing gold among players, populations, merchants, monsters, hoards, and mineral deposits. Gold can nevertheless enter or leave a **regional or player economy**. The quantity *available for transactions* may rise or fall even as the world total remains fixed.

This proposal explores a soft constraint on non-player economies: a population short of currency can still farm, barter, and survive, but urban specialization, wages, commercial services, and taxation suffer. Craftspeople emigrate, peasants may return to subsistence agriculture, and displacement and insecurity can foster banditry. A city's prosperity should ideally emerge from these conditions, not from an arbitrary prosperity score.

This is a proposed direction for investigation, **not an implementation specification**. All amounts and percentages below are illustrative.

## Problem

In a game with unlimited gold faucets, a persistent world's money supply may grow faster than its sinks can remove it. Conversely, simply charging gold for every activity and deleting it can produce a long-run shortage. Neither mechanism captures the consequences of wealth moving between provinces, the population, and rulers.

Olympia also needs a model in which maintaining power has costs: troops require pay, castles require maintenance, and animals must be trained and replaced. A faction can minimize spending by withdrawing from the world, but sustaining military and economic power requires continued participation in it.

The existing G3 mechanics do **not** currently implement this closed-gold proposal. Issue #135 should first establish the actual sources, sinks, and transfers.

## Goals

- Preserve global gold conservation after world generation, subject to explicitly modeled exceptions if any are later approved.
- Let provinces and cities gain or lose circulating money through understandable economic activity.
- Make upkeep, taxation, trade, investment, migration, and security interact meaningfully.
- Permit urban contraction and recovery without automatically starving an entire population.
- Allow players to influence outcomes while avoiding the need to simulate every individual NPC transaction.
- Make economic flows auditable and deterministic enough for reproducible turn tests.

## Non-goals

- Simulating every peasant, artisan, merchant, or purchase.
- Making gold the sole measure of prosperity or productive capacity.
- Implementing a modern banking, credit, or fiat-money system.
- Specifying final prices, migration rates, or tax rates before simulation and playtesting.
- Rebalancing current G3 as part of the initial MIMO audit.

## 1. Accounting boundaries

Let the total quantity of gold in the entire world be `G`.

At world creation, allocate it among deposits, monster and lair inventories, non-player populations and merchants, and any starting player holdings. After creation:

`G(t+1) = G(t)`

Thus, at the **global** boundary, newly minted gold and destroyed gold both equal zero.

At a **regional** or **player-economy** boundary, however:

`G_region(t+1) = G_region(t) + inflows - outflows`

Flows are defined relative to a boundary. A payment from a ruler to local workers moves gold from the player sector to the local population sector, but does not leave the province. Payment to a visiting merchant may leave the province when the merchant travels away. A player-to-player transaction changes neither global quantity nor necessarily the regional quantity.

### Suggested gold states

| State | Description | Usually taxable? |
| --- | --- | --- |
| Circulating | Gold held by households, traders, governments, and players and available for transactions | Only through applicable taxes or other lawful transfers |
| Hoarded | Savings, buried treasure, monster holdings, or otherwise inactive money | Not automatically |
| Unmined | Gold in mineral deposits | No; extraction moves it into circulation or a hoard |

The states are accounting categories, **not extra copies of the gold**. Every unit must have one owner/location and belong to exactly one state. Geographic region and controlling sector are additional independent attributes. A monster hoard can be physically inside a province without being part of its accessible monetary economy.

Distinguish at least: (a) world stock, (b) region-local accessible stock, (c) player-controlled stock, and (d) active transaction volume. Money stock and money turnover are not identical.

## 2. Non-player population as a reservoir

When players pay wages, purchase local goods, or commission work, gold passes to the population and local merchants rather than disappearing. The population can return gold through taxes, purchases from players, fees, or trade. A province's ability to pay taxes is therefore limited by its actual monetary resources, not an endlessly replenishing abstract tax base.

Some paid gold leaks from the region through imported goods, traveling merchants, migrant workers, remittances, military expeditions, and trade. These are **regional outflows**, not global sinks. The destination must be accounted for, even if represented by an aggregate treasury rather than an individual NPC.

Illustration: a ruler spends 100 gold on castle maintenance; 60 reaches local workers and merchants, 25 is carried to another region, and 15 is saved or hoarded. All 100 remains somewhere in the world. The split is illustrative and must not be encoded as a fixed rule without testing.

### Hard and soft monetary constraints

Under a hard constraint, a population with no gold cannot purchase goods or pay taxes. This can make an entire region inert.

Under the proposed **soft constraint**, residents may still produce food, barter, and undertake subsistence activity. But currency-dependent services, urban employment, specialist trade, equipment acquisition, and monetary taxes contract. The population can remain alive while the city becomes commercially unviable.

Currency shortages should affect *commercial exchange and fiscal capacity* more directly than the physical ability to grow food. Gold is not interchangeable with food, land, tools, or skill.

## 3. Urban decline, migration, and recovery

Cities depend on specialization and dense exchange. Craftspeople often need paying customers and reliable suppliers; they cannot all meet their needs through subsistence production. A prolonged shortfall of circulating currency and demand could create the following feedback loop:

1. Gold leaves the region, or is hoarded locally; commercial spending declines.
2. Merchants sell less; workshops struggle to buy supplies and pay workers.
3. Skilled tradespeople seek more prosperous towns; some workshops close.
4. Displaced peasants return to rural subsistence where possible; others migrate or seek paid service.
5. A fraction of displaced people may become bandits or pillagers, especially where security and alternative employment are poor.
6. The shrinking commercial base reduces tax revenues; insecurity further discourages trade and settlement.

These are **possible behavioral responses**, not automatic transformations. Poverty alone should not turn every peasant into a bandit. Land access, food supply, governance, security, alternative employment, and migration routes should matter.

Recovery should likewise be possible. A ruler's investments in security, roads, construction, market access, local purchases, and reliable wages can attract people and restore commerce. Spending money *outside* the region would yield less local stimulus than spending it locally.

### City conditions

| Illustrative condition | Observable effects |
| --- | --- |
| Prosperous | Merchants and skilled labor arrive; broader goods and services are available |
| Stable | Existing commerce and employment remain viable |
| Stagnating | Weak demand, fewer orders, declining market activity |
| Impoverished | Skilled emigration and workshop closures; rural return migration |
| Collapsing | Abandonment, reduced security, pillaging risk, and diminished taxes |

These labels should be **derived descriptions**, if used at all, rather than an independent dial decremented by the engine. The underlying state may include accessible money, transaction activity, population, employment/specialization, local production, food, trade links, and security. Do not assume money balance alone determines prosperity: money *velocity* (how often currency changes hands) matters as well.

## 4. Monsters, lairs, and hoards

Monsters and lairs are natural forms of temporary monetary sequestration. Gold carried by bandits, held by a dragon, or buried with the dead is unavailable for local commerce until retrieved. Recovering it changes ownership and accessibility, not the world total.

This ties returning money to exploration and risk, but can concentrate wealth among already-powerful factions. Starting hoards should be budgeted at world generation. Subsequent hoards should ideally arise from modeled acquisition (theft, tribute, looting, or other transfers), rather than minting fresh gold whenever an encounter appears.

## 5. Should inaccessible gold become new deposits?

Two designs deserve comparison:

**A. Literal conservation and location.** Gold remains wherever it was last held. Deposits are assigned at generation, mined over time, and not replenished. This is conceptually clean, but hoarded or abandoned money may become inaccessible indefinitely. Exploration opportunities may diminish over a long-lived game.

**B. Abstract recycling reserve.** The engine moves some gold that is deliberately removed from *accessible circulation* into a tracked world reserve. New mineral veins, hoards, or discoveries can be allocated from that reserve, never exceeding its balance. This supports continued discoveries but is a deliberate game abstraction: wages paid to workers do not literally transform into ore underground.

Avoid silently converting **all** money paid to NPC populations into new deposits. Doing so could perversely reward impoverishing a region with richer mines. Recycling, if adopted, should have explicit eligibility, delays, limits, geographic rules, and a transparent conservation ledger.

The initial implementation could omit recycling entirely until simulations demonstrate a need for it.

## 6. Economic consequences and risks

### Potential benefits

- **Meaningful rule:** Tax extraction without local reinvestment can impoverish a region, while maintaining armies and buildings returns some money to workers and suppliers.
- **Regional differentiation:** Trade routes, commerce, security, and migration affect where currency circulates.
- **Emergent urban change:** Cities expand or contract due to player decisions and regional conditions, without scripted prosperity decrements.
- **Exploration value:** Gold in deposits and hoards can re-enter circulation when found.
- **Auditable supply:** A finite world budget supports strong conservation assertions.

### Risks and mitigations to investigate

| Risk | Possible response |
| --- | --- |
| A wealthy faction drains a province's currency | Taxes and commerce weaken; local investment and imports can restore circulation |
| Existing factions monopolize returning treasure | Vary the location, difficulty, and accessibility of opportunities; measure wealth concentration |
| Gold disappears into inaccessible accounts or abandoned holdings | Account for these states explicitly; consider a narrowly specified recovery policy |
| Per-city or per-NPC simulation becomes too expensive | Use aggregate population and merchant balances with explicit interregional flows |
| Fixed prices become disconnected from local conditions | Treat prices and trade availability as separate later decisions; observe actual play first |
| Shortages trigger an irreversible death spiral | Permit rural subsistence, migration, trade, and player-supported recovery |
| Arbitrary recycling breaks immersion | Prefer traceable transfers; use a reserve only if justified and label it an abstraction |

## 7. Proposed implementation and evaluation sequence

This is a design exploration and should proceed separately from the initial audit in #135.

1. **Audit current behavior.** Complete #135's inventory of gold creation, destruction, and transfers. Establish the baseline turn-level accounting and tests.
2. **Model conservation in isolation.** Prototype explicit world, region, and sector balances without immediately changing live market or tax rules. Validate that every transfer conserves gold globally.
3. **Prototype one province and one city.** Use an aggregate non-player population treasury, a commercial activity measure, local upkeep and taxation, and a simple neighboring-region flow. Test both money shortages and player-funded recovery.
4. **Introduce migration and security responses.** Test artisan departure, subsistence migration, recruitment, bandit formation, and possible recovery. Avoid deterministic mass conversion of civilians to pillagers.
5. **Compare world models.** Run representative multi-turn cases with/without recycling, multiple regional trade arrangements, active and inactive factions, concentrated and distributed wealth, and different rates of local spending.
6. **Present balancing decisions.** Only then propose changes to markets, taxes, upkeep, deposits, hoards, and population behavior as separate issues.

### Suggested metrics

- Total gold by owner/sector, state, and region; global conservation discrepancy.
- Circulating money per capita and turnover/transaction volume by city or region.
- Inflows/outflows from tax, wages, upkeep, commerce, loot, migration, and trade.
- Employment/specialization, artisan migration, rural population, food security, and bandit incidents.
- Tax capacity, trade availability, city recovery time, and wealth concentration among factions.

## 8. Open decisions

1. **Population abstraction:** One treasury per province, separate rural/urban treasuries, or multiple socioeconomic groups?
2. **Regional accounting:** When is a merchant's money considered to have left: on payment, on travel, or on settlement elsewhere?
3. **Taxes:** What fraction of local population balances is accessible to a ruler, and how does coercive extraction affect security and migration?
4. **Velocity and prices:** Should commercial activity be derived from observed transactions, approximated through turnover, or reflected in market capacities and prices?
5. **Subsistence:** What goods and labor can a population supply with little or no gold?
6. **Migration:** How do artisans, peasants, soldiers, and bandits choose destinations and occupations?
7. **Hoard recovery:** Can abandoned gold remain inaccessible permanently, or is some form of recycling needed in a very long-running game?
8. **World growth:** If new map regions or populations are introduced midgame, must their gold allocation come from existing world reserves?
9. **Compatibility:** Which changes would alter existing Olympia gameplay enough to require an explicitly different rule set or scenario option?

## Recommendation

Use **global gold conservation** as the accounting invariant, **regional monetary flows** as the economic mechanism, and a **soft monetary constraint** for non-player populations. Let urban prosperity emerge from money circulation, production, specialization, trade, and security. Keep a possible recycling reserve as an option, not an assumed source of fresh mineral deposits.

Before implementing this model, complete the MIMO audit described in [#135](https://github.com/playbymail/olympia-g3/issues/135). It should give us the baseline against which to test whether a finite-gold design improves Olympia's persistent economy.
