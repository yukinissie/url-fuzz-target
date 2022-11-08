# afl.rsを動かしてみる

AFL++をRustでwrappしたafl.rsを見つけてしまった．
これを使うとRustで作られたソフトウェアをファジングできるらしい．
https://github.com/rust-fuzz/afl.rs

使ってみる．

## セットアップ方法[1]

必要なツールは以下3つ
- Cコンパイラー（例：gcc，clang）
- make
- Rust実行環境
    - https://www.rust-lang.org/tools/install

環境
- x86-64のLinux
- x86-64のmacOS

以下のコマンドで入れられます．

```
cargo install afl
```

## チュートリアル
詳細はRust Fuzz Bookを見てください．
https://rust-fuzz.github.io/book/afl/tutorial.html

### ファズターゲットを作成する

```bash
cargo new --bin url-fuzz-target
cd url-fuzz-target
```

Cargo.tomlを更新する

```toml
[dependencies]
afl = "*"
url = "*"
```

src/main.rsを更新する

```rust
#[macro_use]
extern crate afl;
extern crate url;

fn main() {
    fuzz!(|data: &[u8]| {
        if let Ok(s) = std::str::from_utf8(data) {
            let _ = url::Url::parse(&s);
        }
    });
}
```

### ファズターゲットをビルドする

```bash
cargo afl build
```

### 開始時の入力を提供する

```bash
mkdir in
echo "tcp://example.com/" > in/url
echo "ssh://192.168.1.1" > in/url2
echo "http://www.example.com:80/foo?hi=bar" > in/url3
```

### ファジングを開始する

```bash
cargo afl fuzz -i in -o out target/debug/url-fuzz-target
```

### 補足：うまくファジングが実行できないとき
ファジング実行時に`libpython3.9.so.1.0` が見つからないとか言われたときは/lib64にシンボリックリンクを貼ります．

```bash
ls -s /home/todalab/anaconda3/lib/libpython3.9.so.1.0 /lib64
```

その上で以下のコマンドを打って実行します．

```bash
LD_LIBRARY_PATH=/lib64 cargo afl fuzz -i in -o out target/debug/url-fuzz-target
```
