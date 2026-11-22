# FFXI Addon — voidwatch

> [!NOTE]
> **Quick Reference**
> - **Path:** `C:\Windower\addons\voidwatch\`
> - **Author / Version:** Mojo / `1.20150930`
> - **Commands:** `//vw`
> - **Load Status:** On-Demand
> - **Origin:** Third-Party / Local modified variant of Mojo's Voidwatch addon

Automates repetitive Voidwatch loops: trading cells/displacers to a Planar Rift, popping the NM (if leader), opening the Riftworn Pyxis, and collecting or converting loot while skipping rare duplicates.

## What the Code Does
- **Cell Trading (`trade_cells`):** Finds `Planar Rift` within `< 6` yalms (`get_mob_by_name`), scans inventory (`bag 0`) for `1 Cobalt Cell` (`3434`), `1 Rubicund Cell` (`3435`), and `5 Phase Displacer` (`3853`), and injects an outgoing `0x036` trade packet. If `leader()` is true and all 5 Phase Displacers were traded, schedules `poke_rift` (`0x01A`) every `4` seconds.
- **Popping & Menu Automation:** When `0x034` arrives for `Planar Rift`, `escape()` presses Escape via `setkey escape down/up` to trigger outgoing `0x05B`, which `start_fight` rewrites to `Option Index = 0x51` (`_unknown1 = 0`).
- **Pyxis Looting (`observe_box_spawn`, `examine_box`, `obtain_item`):** Watches `0x0E` (`_unknown2 == 768`/`770`) and `0x38` (`deru`/`kesu`) for `Riftworn Pyxis` spawn/despawn. On spawn, waits a staggered party-sorted offset (`(k - 1) * 0.4`s) and pokes the Pyxis (`0x01A`). In `0x34`, inspects 8 item slots in `Menu Parameters`:
  - Because `pulse_items` is **commented out** in this copy, `choice.option` is never set to an item slot; it defaults to `choice.option = 9` (if `count == 1` and the item is Rare and already owned across all 12 `bags`) or `choice.option = 10` (convert/collect all non-rare or multi-slot).
  - `obtain_item` rewrites outgoing `0x05B` with `choice.option` (`_unknown1 = 0`).
- **Cycle Reset:** Incoming `0x036` (`Message ID == 10876` or `10875`) or Pyxis despawn resets `conditions['trade'] = true`; when `0x0E` sees `Planar Rift` respawn, it waits `1`s and restarts `trade_cells()`.

## Commands & Usage
- `//vw start` — Enable automation (`running = true`) and immediately trade cells to a nearby Planar Rift.
- `//vw stop` — Disable automation (`running = false`).

## Events, Packets & Technical Hooks
- **Windower Events:** `addon command`, `incoming chunk`, `outgoing chunk`.
- **Packets:**
  - Outgoing: `0x01A` (injected poke), `0x036` (injected cell trade), `0x05B` (modified to pop NM or claim/convert Pyxis).
  - Incoming: `0x0E` (spawn/despawn), `0x38` (`deru`/`kesu` spawn/despawn), `0x34` (menu parameters), `0x36` (ineligibility messages `10875`/`10876`).
- **Dependencies / Libraries:** `logger`, `coroutine`, `packets`, `resources`.

## Local Settings & Customizations
No `data/settings.xml`. Compared to `voidwatchpulsecell`, this copy has `pulse_items` commented out (lines 39–45), while already containing the `cells` table (`Cobalt Cell`, `Rubicund Cell`, `Phase Displacer`) and `math.sqrt(mob.distance) < 6` distance check in `get_mob_by_name`.

## Related Notes
- `FFXI Windower Overview`
- `FFXI Windower Addons Catalog`
- `FFXI Addon — voidwatchpulsecell`
