# DataSignals Lab - signal-set proof chain

Every day this repository receives one new entry in `chain.jsonl`: a canonical
hash of the signal set published that day on https://datasignalslab.com
(reports, alert streams and the append-only performance archive). Each day
hash includes the previous one, so no historical signal can be altered,
added or removed afterwards without breaking every later hash.

Two independent anchors, both free and verifiable by anyone:

1. **This public git mirror.** Each day is committed and tagged
   (`proof-YYYY-MM-DD`). GitHub records the push time server-side.
2. **OpenTimestamps.** Each day entry (`days/YYYY-MM-DD.json`) is stamped
   against the public OpenTimestamps calendars, which anchor it in the
   Bitcoin blockchain (`days/YYYY-MM-DD.json.ots`).

## Verify it yourself

Recompute the chain (no trust in us required):

```python
import hashlib, json
prev = "0" * 64
for line in open("chain.jsonl"):
    e = json.loads(line)
    body = {k: v for k, v in e.items() if k != "hash"}
    want = hashlib.sha256((prev + json.dumps(body, sort_keys=True,
        separators=(",", ":"), ensure_ascii=False)).encode()).hexdigest()
    assert e["hash"] == want and e["prev"] == prev, f"chain broken at {e['date']}"
    prev = e["hash"]
print("chain intact:", prev)
```

Verify a Bitcoin timestamp (needs `pip install opentimestamps-client`):

```
ots verify days/2026-07-08.json.ots -f days/2026-07-08.json
```

Compare tag dates with entry dates: `git log --tags --format='%ai %d'`.

Plain-language explanation: https://datasignalslab.com/proof.html
