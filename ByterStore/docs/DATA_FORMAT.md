# On-Disk Data Format

ByterStore persists each model as one or more **segments**. Each segment consists of three files sharing the same numeric id.

## Segment files

| Extension | File | Contents |
|-----------|------|----------|
| `.kv` | `segment_{id}.kv` | Append-only data records |
| `.idx` | `segment_{id}.idx` | Binary index: `(uint64_t hash, uint64_t offset)*` |
| `.bf` | `segment_{id}.bf` | Serialized Bloom filter |

Example for model `users`:

```
data/users/segment_1.kv
data/users/segment_1.idx
data/users/segment_1.bf
```

## Record layout (`.kv`)

Records are written sequentially. All multi-byte integers are **little-endian** (native on x86/ARM).

```
┌─────────────────────────────────────────────────────────────┐
│ record_len   uint32  Total length of fields after this field │
│              (includes key, val, flags, reserved, CRC)       │
├─────────────────────────────────────────────────────────────┤
│ key_len      uint32  Byte length of key                      │
│ val_len      uint32  Byte length of value                    │
│ flags        uint8   1 = alive, 0 = tombstone (deleted)      │
│ reserved     uint8   Reserved (always 0 today)               │
├─────────────────────────────────────────────────────────────┤
│ key          key_len bytes   UTF-8 key string                │
│ value        val_len bytes   UTF-8 value string              │
├─────────────────────────────────────────────────────────────┤
│ crc32        uint32  IEEE CRC-32 over payload below          │
└─────────────────────────────────────────────────────────────┘
```

### CRC scope

CRC is computed over bytes starting **immediately after** `record_len` through end of `value`:

```
crc_input = key_len || val_len || flags || reserved || key || value
```

Algorithm: **CRC-32 (IEEE 802.3)** via lookup table in `utils.hpp`.

On read, mismatch → record treated as corrupt → `get` returns not found.

### Tombstones

- **Delete** sets `flags = 0` in place (same offset).
- **Empty value on write** also sets `flags = 0`.
- **Updates** append a **new** record; index hash → new offset (last write wins for that segment’s index).

### Size formula

```
record_len = 4 + 4 + 1 + 1 + key_len + val_len + 4
           = key_len + val_len + 14   (when counting header fields after record_len)
```

(Implementation: `record_len` includes `key_len`, `val_len`, `flags`, `reserved`, key, value, and CRC field sizes.)

## Index file (`.idx`)

Binary stream of pairs, no header:

```
repeat until EOF:
  hash   uint64   FNV-1a(key) at insert time
  offset uint64   Byte offset in matching .kv file
```

Loaded into `RobinHoodMap<uint64_t, size_t>` on segment open; saved on segment destructor.

## Bloom filter file (`.bf`)

```
bitsize  size_t   Number of bits (m)
bits     bitsize bytes   One byte per bit slot (0/1)
```

Used before index lookup to skip definite misses. Parameters: `m` bits, `k` hash functions (double-hashing from 64-bit hash).

## Lookup algorithm

```
hash = fnv1a(key)
for segment in [current, closed...]:
  if not bloom.maybeContains(hash): continue
  if offset = index.get(hash):
    read record at offset
    if flags == 0: continue  // tombstone in old copy
    if key matches and crc ok: return value
return not found
```

## Segment rotation

When the active segment’s size exceeds `segment_size_mb` from config, `SegmentMgr` seals the segment (push to `closed`) and opens `segment_{next_id}`.

Closed segments remain readable for lookup; new writes go to the new active segment.

## Example (conceptual)

Key `"alice"`, value `"{\"age\":30}"`:

| Field | Value |
|-------|-------|
| key_len | 5 |
| val_len | 10 |
| flags | 1 |
| key | `alice` |
| value | `{"age":30}` |
| crc32 | computed over header fields + key + value |

Stored at end of `segment_1.kv`; index maps `fnv1a("alice")` → offset.
