# noun

Content-addressable 32-byte identifiers for immutable Merkle proofs.

A `Noun` is the identity of a byte string: BLAKE3 of the bytes, stored as a
fixed `[u8; 32]`. Equal contents produce the same noun. The type is `no_std`,
`#[repr(C)]`, and ordered, so it can be a map key, a wire id, or a Merkle leaf
without allocating.

This crate does not build trees or verify proofs. It only mints and compares
the identifiers those proofs name.

## Install

```toml
[dependencies]
noun = "0.1.0"
```

```sh
cargo add noun
```

`noun` depends on `blake3` with its default features disabled.

## Example

```rust
use noun::Noun;

let noun = Noun::of(b"example data");
assert!(!noun.is_null());
assert_eq!(noun.as_bytes().len(), 32);

// Content-addressed: the same bytes always mint the same id.
assert_eq!(noun, Noun::of(b"example data"));

// Round-trip a raw 32-byte id. Any other length is rejected.
let restored = Noun::from_bytes(noun.as_bytes()).expect("32 bytes");
assert_eq!(restored, noun);
assert!(Noun::from_bytes(&[0u8; 31]).is_none());

// The all-zero id is the null noun. It is not the hash of empty input.
assert!(Noun::null().is_null());
assert!(!Noun::of(b"").is_null());
```

A keyed noun uses BLAKE3's keyed mode. The key is 32 bytes and is not mixed
into the payload:

```rust
use noun::Noun;

let key = [7u8; 32];
let sealed = Noun::keyed(&key, b"example data");
assert_ne!(sealed, Noun::of(b"example data"));
```

## API

| Method | Role |
| --- | --- |
| `Noun::of(bytes)` | BLAKE3 of `bytes`. |
| `Noun::keyed(key, bytes)` | BLAKE3 keyed hash. `key` is `[u8; 32]`. |
| `Noun::null()` | The all-zero identifier. |
| `Noun::is_null()` | `true` only for the all-zero id. |
| `Noun::from_bytes(bytes)` | Wrap an existing 32-byte id. `None` otherwise. Does not hash. |
| `Noun::as_bytes()` | Borrow the raw `[u8; 32]`. |

`Noun` derives `Clone`, `Debug`, `PartialEq`, `Eq`, `PartialOrd`, `Ord`, and
`Hash`. The hash field is private: construct with `of`, `keyed`, `null`, or
`from_bytes`.

## Documentation

API docs are on [docs.rs/noun](https://docs.rs/noun).

## License

[AGPL-3.0-or-later](LICENSE).
