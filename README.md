# ERC-20 Token Wallet & Value Transfer

A full-stack Web3 project: a from-scratch **ERC-20 smart contract** (backend) and a **wallet dApp** (frontend) that reads balances and sends tokens with strict validation. Built for DecodeLabs Project 3.

## Architecture (Input → Process → Output)

| Phase | Where | What happens |
|---|---|---|
| **Input** | `frontend/index.html` | Address regex + EIP-55 checksum, zero-address block, amount format check (no negatives/exponents), decimal-place limit, overdraft check against the user's balance |
| **Process** | `contracts/DecodeToken.sol` | `transfer` runs checks → effects → event: rejects the zero address, reverts with `InsufficientBalance(available, required)` |
| **Output** | Contract + UI | Mandatory `Transfer(from, to, value)` event; the UI re-reads `balanceOf` and rebuilds history from event logs |

## Security & correctness highlights

- **No overdrafts, no underflow:** explicit balance check plus Solidity ≥0.8 built-in overflow/underflow reverts (no SafeMath needed).
- **Zero-value transfers** are valid and emit an event, as EIP-20 requires.
- **Double-precision trap avoided:** the frontend never uses JS `Number` for balances. Amounts are parsed with `ethers.parseUnits` and compared as `BigInt`, so 256-bit values keep full precision.
- **Custom errors** for cheaper gas and descriptive failures; **no external calls** in transfer logic (no reentrancy surface).
- The frontend renders on-chain data with `textContent` only (XSS-safe).

## ERC-20 interface implemented

`name`, `symbol`, `decimals`, `totalSupply`, `balanceOf`, `transfer`, `approve`, `allowance`, `transferFrom`, plus `Transfer` and `Approval` events.

## Getting started

```bash
npm install
npx hardhat test                 # run the test suite

# Terminal 1
npm run node                     # local chain on http://127.0.0.1:8545 (chainId 31337)
# Terminal 2
npm run deploy:local             # deploys DecodeToken, writes frontend/config.js
npm run frontend                 # serves the dApp on http://localhost:3000
```

In MetaMask, add the local network (RPC `http://127.0.0.1:8545`, chain ID `31337`) and import one of the private keys printed by `hardhat node`. The first account holds the full 1,000,000 DLT supply.

### Deploy to Sepolia

Copy `.env.example` to `.env`, fill in `SEPOLIA_RPC_URL` and `DEPLOYER_PRIVATE_KEY` (a throwaway test key), then run `npm run deploy:sepolia`. Set `chainId` to `11155111` in `frontend/config.js` if the script does not.

## Project structure

```
contracts/DecodeToken.sol     ERC-20 token
test/DecodeToken.test.js      Hardhat tests (transfers, overdraft, zero-value, allowances)
scripts/deploy.js             Deploys and writes frontend/config.js
frontend/index.html           Wallet dApp (ethers v6, no build step)
.github/workflows/ci.yml      Compile + test on every push
```

## Test coverage

Metadata and minting, transfer + event, full-balance transfer, zero-value transfer, overdraft (1 wei over), empty wallet, zero-address recipient, negative / out-of-range inputs, approve/transferFrom, allowance exceeded, infinite allowance.

## Ideas to extend

Gas-fee estimator (`estimateGas`), paginated transaction history, EIP-2612 permit, pausable/mintable roles, Etherscan verification on a public testnet.

## License

MIT
