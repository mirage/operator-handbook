# MirageOS operator handbook

Live at [https://mirage.github.io/operator-handbook](https://mirage.github.io/operator-handbook).

This book use `mdbook` and `mdbook-graphviz`. You can install them via `rustup`
and `cargo`:

```sh
$ rustup install stable
$ cargo install mdbook
$ cargo install mdbook-graphviz
$ export PATH=$PATH:$HOME/.cargo/bin
$ git clone https://github.com/mirage/operator-handbook
$ cd operator-handbook
$ mdbook serve
```

You can see the book on `http://localhost:3000`
