# 20221229_rust_wasm_webapp_test1

## about

rustでwasm生成してwebアプリ作成する個人的な学習リポジトリです。



## メモ: 初回環境構築手順メモ

- 20221229
	- まず最初はwasmerでのhello worldを試した。
		- https://wasmer.io/
			- wasmerインストール
				- curl https://get.wasmer.io -sSfL | sh
			- wasmerのパッケージマネージャーwapmで動作確認
				- wapm install cowsay
				- wapm run cowsay Hello World
					- 動作done
			- とりあえずドキュメント精査は後回しにしてセットアップ状況を確認
				- ~/.wasmer/wapm.log
				- これをtailすればどこから実行してるかなどの詳細が分かる様子。
				- `/wapm_packages/syrusakbary/cowsay@0.2.0/target/wasm32-wasi/release/cowsay.wasm
					- ここにwasmパッケージが入ることを確認。

	- 普段使いのvscodeにwebassembly拡張を入れる。
		- WebAssembly Foundationのもの。1.4.0。
		- cawsayのwasmを右クリックでwatに変換できることを確認。

	- バイトコードアライアンスのgithubのチュートリアルを眺める。
		- https://github.com/bytecodealliance/wasmtime/blob/main/docs/WASI-tutorial.md
			- 初手でclangでcを変換するのは失敗したので一旦深追いせず、今回はrustを採用。

	- rust導入
		- https://www.rust-lang.org/ja/learn/get-started
			- 手元環境には過去に試したときのrustが入ってた(which rustup)ので、今回は「rustup update」を実行して最新にするところから。
		- cargo new wasi_demo
		- cd wasi_demo
		- rustup target add wasm32-wasi
		- cargo build --target wasm32-wasi
		- file target/wasm32-wasi/debug/wasi_demo.wasm
			> target/wasm32-wasi/debug/wasi_demo.wasm: WebAssembly (wasm) binary module version 0x1 (MVP)
			- ※BigSurだと「ERROR: Bad magic format `version %#x (MVP)' (bad format char: #)」になる。Venturaではwasmを認識する様子。
		- wasmer run target/wasm32-wasi/debug/wasi_demo.wasm
			- helloworld完了。

	- ここでwasi_demo.wasmが2.1MBと巨大なのが気に食わないので情報調査。
		- wasmを十分に小さくする手順を先に把握しておかないと、意欲が湧かない。
			- ※これはまあ、flexで作ったswf時代の手軽さを懐古しているだけではあるのだけど、それにしてもhelloworld的なものが2.1MBなのはwebアプリの古兵としては納得いかず。。
		- https://doc.rust-lang.org/cargo/reference/profiles.html
			- こちらを参考に、`Cargo.toml`を編集。
				> [profile.release]
					debug = false
					opt-level = "z"
					strip = true
			- cargo build --release --target wasm32-wasi
				- `68KB`になった。
				- wasmer run target/wasm32-wasi/release/wasi_demo.wasm
				- 動作確認も正常。

	- ここまでで、以下の状況をセットアップ・確認できた。
		- rustをセットアップして、「wasm32-wasi」ターゲットのwasmを作成できた
		- wasmerをセットアップして、「wasm32-wasi」ターゲットのwasm実行を正常に確認できた
		- cargo.tomlに設定を追加し、release設定でサイズの最小化を適用する方法を把握した


`EOF`