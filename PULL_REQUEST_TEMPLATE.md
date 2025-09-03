## Summary

- **Title**: Vault + Strategy MVP (ERC-4626 + Compound v3 on Base)
- **Scope**: Introduces modular strategy interface, ERC-4626 vault, first production strategy (Comet/Compound v3 USDC on Base), deploy script, env scaffolding.

## Contracts Added/Changed

- `contracts/interfaces/IStrategy.sol`
- `contracts/Vault.sol`
- `contracts/strategies/CompoundV3Strategy.sol`
- `contracts/strategies/AaveV3Strategy.sol` (stub)

## Deployment

- Script: `script/deploy/base.ts`
- Env: copy `.env.example` to `.env` and set:
  - `PRIVATE_KEY`
  - `BASE_RPC_URL` or `BASE_SEPOLIA_RPC_URL`
  - `BASESCAN_API_KEY`
  - `USDC_BASE=0x833589fCD6EDb6E08f4c7c32D4f71B54bDa02913`
  - `COMET_PROXY_BASE=<cUSDCv3 Comet proxy address on Base>`

## Audit Notes

- ERC-4626 integration uses overrides of `deposit`, `mint`, `withdraw`, `redeem` to route funds to/from the active strategy.
- Strategy approvals use `SafeERC20.forceApprove` for OZ v5.
- `Vault` roles: `DEFAULT_ADMIN_ROLE` and `STRATEGIST_ROLE` (admin is granted both at deploy). Strategy updates require pause.
- `totalAssets()` includes idle + `strategy.totalAssets()`.
- Reentrancy protected on state-changing entry points that move funds.
- Pausable on all user flows via overrides.

## Risk/Invariant Checklist

- [ ] Initial deposit frontrun considerations acknowledged (OZ ERC-4626 docs). Consider seeding vault before enabling deposits.
- [ ] Strategy `asset()` verified to match `Vault.asset()` during `setStrategy`.
- [ ] No external calls before effects when minting/burning shares (use of `super.*` then strategy routing).
- [ ] Allowance increases use `safeIncreaseAllowance` to avoid sticky approvals.
- [ ] `harvest()` is a no-op for Comet base asset (no incentives by default). Future rewards logic to be added behind role-gated call.
- [ ] Upgrade path: swapping strategy requires pausing vault.

## Testing Plan (follow-up PR)

- Unit tests for share-accounting invariants, deposit/withdraw flows, and strategy interactions (mock Comet).
- Fork tests on Base Sepolia/Mainnet for end-to-end sanity.

## Screenshots/Artifacts

- Hardhat compile successful on Solidity 0.8.24, OZ 5.4.

## Breaking Changes

- None (new module).