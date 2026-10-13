# clone-lin-gmx-v2-pnl

Experimental i64-scale LIN clone of GMX V2 `PositionUtils.getPositionPnlUsd` / `Price.pickPriceForPnl` / `Calc.roundUpDivision`.

**Proofs and evidence live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open)** (`src/lin_gmx_v2_pnl.lin`, `test/prove_gmx_v2_pnl_external.py`, `examples/gmx_v2_pnl/`). This repository is the separate LIN copy only.

## Kernel (EXPERIMENTAL, not mainnet uint256, not pool-capped)

```
price = pickPriceForPnl(isLong, false)   # long -> min, short -> max
value = tokens * price
pnl   = isLong ? value - sizeUsd : sizeUsd - value
dt    = full close ? tokens
      : isLong ? ceil(tokens * sizeDeltaUsd / sizeUsd)
               : floor(tokens * sizeDeltaUsd / sizeUsd)
pnlUsd = pnl * dt / tokens
```

Named contrasts (recomputed in lin-open against C11 `__int128`, limb muldiv, LIN Compiler 0 VM, Python bigint, and libtcc `roundtrip-jit`):

| Path | Result |
|---|---|
| long 2100 vs entry 2000 | `+1e14` |
| long 1900 | `-1e14` |
| short minimize uses pmax=1950 | `+5e13` (not `+1e14` at pmin=1900) |
| long min oracle 2000/2100 | `0` |
| long wrong max oracle | fakes `+1e14` |
| V2 dust tokens=3 sizeUsd=3001 p=1100 | `299` vs V1-style `300` |
| long half-close 3/3000/1500 | ceil dt=`2` PnL=`200` vs floor `1` |
| naive i64 `7e9*2e9/1e6` | wraps; 128-bit `14000000000000` |

This is **not** GMX V1 `adjustForDecimals`, `validateLiquidation`, `getNextFundingRate`, `getFundingFee`, or `getFeeBasisPoints`. Pool `MAX_PNL_FACTOR_FOR_TRADERS` cap is skipped.

## Upstream

- https://github.com/gmx-io/gmx-synthetics `a85ea3491c19c93bb4b5a002d9b358fb769b7849`
- `contracts/position/PositionUtils.sol` blob `53667b52746f5f74003b24290fa8ef0f37b83a84`
- `contracts/price/Price.sol` blob `679ea9fb62a7077e49cdfbc6e0b9a2298ebd4340`
- `contracts/utils/Calc.sol` blob `9e48fc4c851ef97c8c57b3755257968b3ea2d397`
- License: GPL-2.0-or-later after BUSL Change Date 2026-08-31

## Files

- `src/lin_gmx_v2_pnl.lin` — LIN kernel (`g2p_*`)
- `gmx_v2_pnl_c11.c` — independent C11 oracle (selftest)
