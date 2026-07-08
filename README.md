# kotoba-lang/org-ietf-opus

Zero-dep portable `.cljc` Opus (IETF RFC 6716) packet **framing** — TOC
(table-of-contents) byte parsing and frame-packing (splitting a packet into
its constituent Opus frames per all 4 frame-count codes). Named
`org-ietf-opus`, same `org-ietf-<spec>` pattern as `org-ietf-deflate`/
`org-ietf-turn`/`org-ietf-cbor`.

**New implementation, not an extraction** — `kotoba-lang/utsushi` had no
Opus support at all, not even a TODO stub (discovered while decomposing
`utsushi` into per-format-spec repos; see `com-junkawasaki/root` ADR
precedent 2607072500). **Actual SILK/CELT audio decode is out of scope** —
frames stay opaque byte ranges, matching the framing-only boundary this
repo family uses (`org-iso-h264`, `org-iso-aac`).

## Namespaces

| ns | role |
|---|---|
| `opus.toc` | TOC byte → config (mode/bandwidth/frame-size, RFC 6716 §3.1 Table 2), stereo flag, frame-count code |
| `opus.packet` | Frame-count-code-driven packet splitting (RFC 6716 §3.2): code 0 (1 frame), code 1 (2 equal frames), code 2 (2 explicit-length frames), code 3 (arbitrary count, CBR or VBR with padding) |

## Usage

```clojure
(require '[opus.toc :as toc] '[opus.packet :as pkt])

(toc/parse toc-byte)     ; => {:config :mode :bandwidth :frame-size-ms :stereo? :frame-count-code}
(pkt/frames packet-bytes) ; => {:toc {...} :frames [{:start :end} ...]}  (opaque byte ranges)
```

## Test

The RFC 6716 §3.1 config table and §3.2 frame-packing rules are precisely
and unambiguously specified (fixed lookup tables / byte-level length
codings) — tests check hand-computed byte layouts against them directly,
the same validation style this repo family uses for cases where a real
encoder isn't the deciding source of truth (contrast `org-iso-h264`'s SPS
parser and `org-iso-aac`'s ADTS parser, both validated against real
`ffmpeg`-encoded fixtures where the spec has header-field freedom a real
encoder pins down).

```sh
clojure -M:test
```
