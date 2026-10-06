# RowLock

RowLock is an intelligent decentralized data verification contract designed for GenLayer. It uses an LLM to evaluate CSV row data against a provided rubric, ensuring inputs meet qualitative constraints before releasing escrowed funds to a worker. If the output fails the criteria, funds are returned to the buyer.

## Execution Requirements
RowLock relies on GenLayer's nondeterministic execution (GenVM) to access off-chain data (HTTPS URLs) and evaluate tasks using intelligent consensus. 

## Tests
To run the local unit tests, install dependencies:
```bash
pip install -r requirements.txt
```
Then run the tests:
```bash
pytest tests/direct/test_rowlock.py -v
```
All 18 local unit tests pass.

```bash
genvm-lint contracts/rowlock.py
# ? Lint passed (2 checks)
# Warnings:
#   line 110: Non-deterministic call 'time.time()'
#   line 136: Non-deterministic call 'time.time()'
#   line 157: Non-deterministic call 'time.time()'
#   line 238: Non-deterministic call 'time.time()'
```
*(The time functions are wrapped in strict_eq in the contract, safely handling the linter warnings.)*

## Evidence
This frozen version of RowLock (SHA-256: `77c5d1dfe10086f7d51baaa580e744faedfec3e26a5ed5c1a02ddb90d3449236`) has been fully validated on GenLayer Studio Devnet (Chain ID 61997). See the successful transaction evidence and balances in [evidence/studio-dev.md](evidence/studio-dev.md).

