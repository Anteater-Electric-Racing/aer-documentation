In order to develop, first install cargo if you haven't already:
```bash
curl https://sh.rustup.rs -sSf | sh
```
Then install mdbook (make sure to restart your terminal if you just installed rust before running the following)
```bash
cargo install mdbook mdbook-mermaid
```
Then you can clone the repo and run `mdbook serve`
```bash
git clone https://github.com/Anteater-Electric-Racing/aer-documentation
cd aer-documentation
mdbook serve
```
