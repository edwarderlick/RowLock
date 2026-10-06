# RowLock

**Intelligent CSV handoff escrow on GenLayer.** A buyer funds a task for a worker to categorize CSV rows. RowLock checks the submitted file against the funded input snapshot, then uses GenLayer's intelligent consensus to judge whether each category fits its description. A passing handoff pays the worker; a failing handoff refunds the buyer.

## Contract and network

| Item | Value |
| --- | --- |
| Contract source | [`contracts/rowlock.py`](contracts/rowlock.py) |
| Raw source SHA-256 | `77c5d1dfe10086f7d51baaa580e744faedfec3e26a5ed5c1a02ddb90d3449236` |
| GenVM runner | `py-genlayer:5jycge4q8k23462jtb0b9fyey1s9qz928sz2nbrd9mg4sxqg2qng` |
| Network | [GenLayer Studio Dev](https://docs.genlayer.com/developers/networks), chain ID `61997` |
| RPC | `https://studio-dev.genlayer.com/api` |
| Live evidence | [`evidence/studio-dev.md`](evidence/studio-dev.md) |

The source file linked above is the same byte sequence identified by the SHA-256 in the [live test record](evidence/studio-dev.md). Studio Dev is a preview network and can be reset; the transaction IDs and observed balances are preserved in that record.

## How the escrow works

1. The buyer deploys RowLock with a worker address, HTTPS input CSV URL, input SHA-256, classification rubric, and duration.
2. `fund_escrow` accepts GEN, fetches and hashes the input CSV, checks its `id,description,amount` structure, and stores a snapshot. The buyer and worker cannot change that snapshot after funding.
3. Before the deadline, the worker calls `submit_output` with an HTTPS output CSV URL and SHA-256. The output adds a `category` column containing `BUG`, `FEATURE`, or `DOCS`.
4. `resolve` fetches the output, verifies its hash and unchanged input columns, and asks GenLayer validators to evaluate whether the categories match the descriptions. `PASS` transfers the escrow to the worker; a failed output check refunds the buyer.
5. The buyer may call `cancel` before an output is submitted. After the deadline, `expire` refunds the buyer. Settlement state is available through `get_settlement`.

The category judgment is the intelligent part: interpreting a description and deciding whether its label is appropriate requires semantic evaluation. That agreed result directly controls the payout.

## Studio Dev deployments and transactions

The following two deployments used the source hash above. Full addresses, transaction hashes, settlement getters, and before/after balances are in the [evidence record](evidence/studio-dev.md). Each transaction link opens the Studio Dev explorer.

| Scenario | Deployed contract | Transactions | Recorded settlement |
| --- | --- | --- | --- |
| Valid categories | `0x31039cDC6fEa667B453f116eF19F9CC799866Ee1` | [Deploy](https://explorer-studio-dev.genlayer.com/tx/0x1ad283dfdcd9ddae992147a5ec064d60cbfee8d79a1c805310810618cf9f9142) · [Fund](https://explorer-studio-dev.genlayer.com/tx/0x49c05a0b9580047b2c6bb76f9f572ba8ab1898ed1eaf11af11ee47a277cf47e7) · [Submit](https://explorer-studio-dev.genlayer.com/tx/0x505a093d4669ca06ab9535a8fb56b778cca1083f5695056a63bc7275cceef439) · [Resolve](https://explorer-studio-dev.genlayer.com/tx/0x463da11e12590dd5a3f666e191ef91c0d3561d594b04a15ef385b8374c0980c6) | `WORKER_PAID`; worker gained `1,000,000,000,000,000` wei; contract balance became `0`. |
| Invalid category | `0xFC9fa33C2cac5AB17D1D4aF56FB58FbfF3a7F53b` | [Deploy](https://explorer-studio-dev.genlayer.com/tx/0x613e952f8675cc2ed86921b93dfe18f597cdf4270dcf11f8ed9dcf25248f58e6) · [Fund](https://explorer-studio-dev.genlayer.com/tx/0x517f8e3ef83fc69fbeea20951ed57026c990f96b88fd06be1a068f4ff50a95df) · [Submit](https://explorer-studio-dev.genlayer.com/tx/0x345f09d79ca2c1a4d5c2e746c68609b54e12c35eb148974d40ccfb90125b1448) · [Resolve](https://explorer-studio-dev.genlayer.com/tx/0xaef54aa18a3822136d18d0fc758274365afd06b08c62e142ebed8122c31d38d8) | `BUYER_REFUNDED: FAIL_CATEGORY`; contract balance became `0`. |

The worker payment and buyer refund above are separate deployments so each `resolve` path can be shown independently. The buyer's net balance change on the refund path includes transaction fees; the settlement getter records a `1,000,000,000,000,000` wei refund.

## Tests and validation

Use Python with the versions in [`requirements.txt`](requirements.txt):

```bash
pip install -r requirements.txt
pytest tests/direct/test_rowlock.py -v
genvm-lint contracts/rowlock.py
```

The recorded direct run passed **18/18 tests**. It covers payment, refund, cancellation, expiry, malformed input and output, changed source content, failed fetching, and prompt injection. The linter passed with four `time.time()` nondeterminism warnings; see the source for the exact time handling. The direct tests and linter are separate from the two live Studio Dev settlement runs documented above.
