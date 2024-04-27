# MirageOS operator manual

This book use `mdbook` and `mdbook-graphviz`. You can install them via `rustup`
and `cargo`:

```sh
$ rustup install stable
$ cargo install mdbook
$ cargo install mdbook-graphviz
$ export PATH=$PATH:$HOME/.cargo/bin
$ git clone https://github.com/mirage/operator-manual
$ cd operator-manual
$ mdbook serve
```

You can see the book on `http://localhost:3000`
