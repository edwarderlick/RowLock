# RowLock Phase 2 Test Results

I executed both the worker-payment path and the buyer-refund path on the Studio Devnet (Chain ID 61997) using the exact, frozen `rowlock.py` source (SHA-256 `77c5d1dfe10086f7d51baaa580e744faedfec3e26a5ed5c1a02ddb90d3449236`). The deployments were successful using the GenLayer Python SDK. The plaintext byte source was hashed explicitly prior to execution.

**Hosted Schema Validation**: Validation against the Studio Dev portal succeeded with the correct mappings.

## Live Run for SHA-256: `77c5d1dfe10086f7d51baaa580e744faedfec3e26a5ed5c1a02ddb90d3449236`
*Fresh disposable accounts were created locally, funded via faucet, and executed successfully. Keys were kept out of tracked files.*

### 1. Worker-Payment Path (Valid Output -> Worker Paid)
- **Buyer**: `0xa55f093e184c2D5BD30634890993eaD48E94709d`
- **Worker**: `0xf078f570181990d34cA0dBfba3Caf1f24e323D8D`
- **Contract Address**: `0x31039cDC6fEa667B453f116eF19F9CC799866Ee1`
- **Deploy TX**: `0x1ad283dfdcd9ddae992147a5ec064d60cbfee8d79a1c805310810618cf9f9142` (Finalized)
- **Fund TX**: `0x49c05a0b9580047b2c6bb76f9f572ba8ab1898ed1eaf11af11ee47a277cf47e7` (Finalized)
- **Submit TX**: `0x505a093d4669ca06ab9535a8fb56b778cca1083f5695056a63bc7275cceef439` (Finalized)
- **Resolve TX**: `0x463da11e12590dd5a3f666e191ef91c0d3561d594b04a15ef385b8374c0980c6` (Finalized)
- **Settlement Result**: `{'amount': 1000000000000000, 'outcome': 'WORKER_PAID', 'recipient': '0xf078f570181990d34cA0dBfba3Caf1f24e323D8D', 'settled': True}`
- **Balances**:
  - **Before Resolve**: Worker = 99,921,318,250,009,529 wei | Contract = 1,000,000,000,000,000 wei
  - **After Resolve**: Worker = 100,921,318,250,009,529 wei | Contract = 0 wei
  - **Verification**: Worker balance increased by exactly 1,000,000,000,000,000 wei (100% of escrow). Contract balance fell to 0.

### 2. Buyer-Refund Path (Invalid Category -> Resolve Refunds Buyer)
- **Buyer**: `0x5582bD97A1e8e02848B9490aEc504b6bBA6bC7a2`
- **Worker**: `0xe49c0463CE5E329eEE86C7af2baeC672c78cF797`
- **Contract Address**: `0xFC9fa33C2cac5AB17D1D4aF56FB58FbfF3a7F53b`
- **Deploy TX**: `0x613e952f8675cc2ed86921b93dfe18f597cdf4270dcf11f8ed9dcf25248f58e6` (Finalized)
- **Fund TX**: `0x517f8e3ef83fc69fbeea20951ed57026c990f96b88fd06be1a068f4ff50a95df` (Finalized)
- **Submit TX (Invalid Category)**: `0x345f09d79ca2c1a4d5c2e746c68609b54e12c35eb148974d40ccfb90125b1448` (Finalized)
- **Resolve TX**: `0xaef54aa18a3822136d18d0fc758274365afd06b08c62e142ebed8122c31d38d8` (Finalized)
- **Settlement Result**: `{'amount': 1000000000000000, 'outcome': 'BUYER_REFUNDED: FAIL_CATEGORY', 'recipient': '0x5582bD97A1e8e02848B9490aEc504b6bBA6bC7a2', 'settled': True}`
- **Balances**:
  - **Before Resolve**: Buyer = 99,921,094,250,009,529 wei | Contract = 1,000,000,000,000,000 wei
  - **After Resolve**: Buyer = 100,873,631,000,009,529 wei | Contract = 0 wei
  - **Verification**: Contract balance fell to 0. Buyer balance significantly increased, successfully returning the escrow minus gas fees.
