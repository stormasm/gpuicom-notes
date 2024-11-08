
delete the old ui branch and make a new one

```rust
gbd ui
gco -b ui
```

Make the following modifications to *Cargo.toml* to enable just the ui library.

```rust
[workspace]
members = ["crates/app", "crates/ui", "crates/story"]
members = ["crates/ui"]

default-members = ["crates/app"]
default-members = ["crates/ui"]
resolver = "2"
```
