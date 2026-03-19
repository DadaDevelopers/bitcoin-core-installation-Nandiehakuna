# Bitcoin Core Installation Assignment

**Student:** Nandiehakuna  
**Cohort:** DadaDevelopers Cohort 4  
**Repository:** bitcoin-core-installation-Nandiehakuna  
**Bitcoin Core Version:** v28.0  
**OS:** Ubuntu/Debian Linux  

---

## Assignment Tasks

1. Compile and run the integration tests and share the results
2. Run Bitcoin Core and use at least 5 RPC commands and share the output

---

## Part 1: Compilation

### Dependencies Installation

Installed all required build dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential libtool autotools-dev automake pkg-config \
  bsdmainutils python3 libevent-dev libboost-dev libssl-dev \
  libsqlite3-dev libzmq3-dev git curl wget
```

### Cloning the Repository

```bash
git clone --branch v28.0 https://github.com/bitcoin/bitcoin.git
cd bitcoin
```

### Build Configuration

```bash
./autogen.sh
./configure --without-gui --with-sqlite=yes
```

**Key build options enabled:**
- Wallet support with SQLite
- ZMQ support
- Test suite
- No GUI (faster build)

### Compilation

```bash
make -j$(nproc)
```

**Binaries successfully built:**
- `bitcoind` — Bitcoin daemon
- `bitcoin-cli` — RPC command line interface
- `bitcoin-tx` — Transaction utility
- `bitcoin-wallet` — Wallet utility
- `bitcoin-util` — General utility
- `test/test_bitcoin` — Unit test binary
- `bench/bench_bitcoin` — Benchmark binary

---

## Part 2: Tests

### Unit Tests

```bash
./src/test/test_bitcoin
```

**Result:**
```
Running 620 test cases...
*** No errors detected
```

✅ All 620 unit tests passed with zero errors.

### Integration Tests

```bash
python3 test/functional/test_runner.py --extended 2>&1 | tee integration_test_results.txt
```

**Result:**
```
ALL                                                      | ✓ Passed  | 4538 s (accumulated)
Runtime: 1733 s
```

✅ All integration tests passed. Skipped tests are expected — they require legacy wallet (BDB) support and USDT tracing which were not included in the build configuration.

---

## Part 3: RPC Commands

Bitcoin Core was run in **regtest mode** — a local private blockchain for testing with no real Bitcoin involved.

### Starting the Node

```bash
./src/bitcoind -regtest -fallbackfee=0.0001 -daemon
```

---

### RPC Command 1 — `getblockchaininfo`

**Purpose:** Returns general information about the current state of the blockchain.

```bash
./src/bitcoin-cli -regtest getblockchaininfo
```

**Output:**
```json
{
  "chain": "regtest",
  "blocks": 0,
  "headers": 0,
  "bestblockhash": "0f9188f13cb7b2c71f2a335e3a4fc328bf5beb436012afca590b1a11466e2206",
  "difficulty": 4.656542373906925e-10,
  "time": 1296688602,
  "mediantime": 1296688602,
  "verificationprogress": 1,
  "initialblockdownload": true,
  "chainwork": "0000000000000000000000000000000000000000000000000000000000000002",
  "size_on_disk": 293,
  "pruned": false,
  "warnings": []
}
```

---

### RPC Command 2 — `createwallet`

**Purpose:** Creates a new wallet to hold keys and Bitcoin.

```bash
./src/bitcoin-cli -regtest createwallet "testwallet"
```

**Output:**
```json
{
  "name": "testwallet"
}
```

---

### RPC Command 3 — `getnewaddress`

**Purpose:** Generates a fresh Bitcoin address from the wallet to receive funds.

```bash
./src/bitcoin-cli -regtest getnewaddress
```

**Output:**
```
bcrt1q3fzgvux0wsjg47kps5zz9uy83k0f7eyjg7r6rp
```

---

### RPC Command 4 — `generatetoaddress`

**Purpose:** Mines 101 blocks instantly in regtest mode, sending the block rewards to our address. 101 blocks are needed so the first block reward (50 BTC) matures past the 100-confirmation coinbase maturity rule and becomes spendable.

```bash
./src/bitcoin-cli -regtest generatetoaddress 101 bcrt1q3fzgvux0wsjg47kps5zz9uy83k0f7eyjg7r6rp
```

**Output:** 101 block hashes returned (truncated for brevity):
```json
[
  "27cf12c2f5a35c662d4364808c921ecb2a2093d251dcbafdba80d8421aa9c33b",
  "0ad154a46f351d878fe0c62b4770500814739d1a3c0215d45c5aced5d153f3a4",
  ...
  "733beda4c7978eed2fab2db34eeb7e6896a6b3241d909631b0fca2cdbdf5cf3a"
]
```

---

### RPC Command 5 — `getbalance`

**Purpose:** Shows the confirmed spendable balance in the wallet.

```bash
./src/bitcoin-cli -regtest getbalance
```

**Output:**
```
50.00000000
```

Only 50 BTC is spendable — the first block's reward. The remaining 100 block rewards are still maturing.

---

### RPC Command 6 — `sendtoaddress`

**Purpose:** Sends 10 BTC to a new address, creating a transaction on the blockchain.

```bash
./src/bitcoin-cli -regtest sendtoaddress bcrt1q78527m4zt8me90uemce6rp2e9xntr6axa34c8r 10
```

**Output (Transaction ID):**
```
985b031785650a7e079f0bd360c97fef5c2060b15c208f541f95797360a6c184
```

---

### RPC Command 7 — `gettransaction`

**Purpose:** Retrieves detailed information about a specific transaction using its transaction ID.

```bash
./src/bitcoin-cli -regtest gettransaction 985b031785650a7e079f0bd360c97fef5c2060b15c208f541f95797360a6c184
```

**Output:**
```json
{
  "amount": 0.00000000,
  "fee": -0.00001410,
  "confirmations": 3,
  "blockhash": "668ef9cb6b614d1d937329866947124eb073746dfaa0f81678258a175eee4eb4",
  "blockheight": 102,
  "txid": "985b031785650a7e079f0bd360c97fef5c2060b15c208f541f95797360a6c184",
  "details": [
    {
      "address": "bcrt1q78527m4zt8me90uemce6rp2e9xntr6axa34c8r",
      "category": "send",
      "amount": -10.00000000,
      "fee": -0.00001410
    },
    {
      "address": "bcrt1q78527m4zt8me90uemce6rp2e9xntr6axa34c8r",
      "category": "receive",
      "amount": 10.00000000
    }
  ]
}
```

Transaction confirmed in block 102 with 3 confirmations. ✅

---

## Summary

| Task | Status |
|------|--------|
| Bitcoin Core v28.0 compiled from source | ✅ |
| Unit tests (620 cases) | ✅ All passed |
| Integration tests | ✅ All passed (Runtime: 1733s) |
| RPC Command 1: `getblockchaininfo` | ✅ |
| RPC Command 2: `createwallet` | ✅ |
| RPC Command 3: `getnewaddress` | ✅ |
| RPC Command 4: `generatetoaddress` | ✅ |
| RPC Command 5: `getbalance` | ✅ |
| RPC Command 6: `sendtoaddress` | ✅ |
| RPC Command 7: `gettransaction` | ✅ |
