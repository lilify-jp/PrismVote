# 🚀 Prism Vote - クイックスタートガイド

## ⚠️ 重要: 正しい起動方法

**エラーが出る場合は、このファイルを必ず確認してください！**

---

## ✅ 正しい起動コマンド

```bash
# これが正しい起動方法です
npm run tauri dev
```

または

```bash
npx tauri dev
```

---

## ❌ よくある間違い

### 間違い1: Viteだけを起動している
```bash
# ❌ これではTauriが起動しません
npm run dev
```

**エラー内容:**
```
TypeError: window.__TAURI_IPC__ is not a function
```

**原因:** ブラウザでViteの開発サーバー (http://localhost:1420) を直接開いているため、TauriのAPIが利用できません。

**解決策:** `npm run tauri dev` を使用してください。

---

### 間違い2: ブラウザで直接開いている

**症状:**
- 投票作成時にエラーが出る
- 設定でIDが生成されない
- Rust backendと通信できない

**原因:** TauriアプリではなくWebブラウザで開いている

**解決策:**
1. ブラウザを閉じる
2. ターミナルで `npm run tauri dev` を実行
3. Tauriのネイティブウィンドウが自動的に開きます

---

## 📋 初回起動の完全な手順

### 1. 依存関係のインストール（初回のみ）

```bash
# フロントエンド依存関係
npm install

# Rustツールチェーン確認
rustc --version
cargo --version
```

Rustがインストールされていない場合:
- Windows: https://rustup.rs/ から rustup-init.exe をダウンロード
- インストール後、ターミナルを再起動

### 2. アプリの起動

```bash
# Tauriアプリを起動（これだけでOK！）
npm run tauri dev
```

初回起動時は以下が自動的に実行されます:
- ✅ Rustバックエンドのコンパイル（数分かかります）
- ✅ Viteフロントエンドサーバーの起動
- ✅ Tauriウィンドウの起動

### 3. アプリが起動したら

- 自動的にTauriのネイティブウィンドウが開きます
- ブラウザで開かないでください
- ウィンドウのタイトルは「Prism Vote」です

---

## 🔧 トラブルシューティング

### エラー: `window.__TAURI_IPC__ is not a function`

**チェックリスト:**
1. [ ] `npm run tauri dev` を使っていますか？
2. [ ] ブラウザではなくTauriウィンドウで開いていますか？
3. [ ] http://localhost:1420 を直接開いていませんか？

**解決方法:**
```bash
# 間違ったプロセスを終了
Ctrl + C でViteサーバーを停止

# 正しいコマンドで再起動
npm run tauri dev
```

---

### エラー: `icons/icon.ico not found`

**解決済み:** tauri.conf.json で bundle を無効化済み。このエラーは開発時には無視できます。

---

### エラー: 投票が作成できない

**症状:**
```
Failed to create poll: TypeError: window.__TAURI_IPC__ is not a function
```

**確認事項:**
1. Tauriウィンドウで開いていますか？（ブラウザではないですか？）
2. `npm run tauri dev` で起動しましたか？
3. ターミナルにRustバックエンドのログが表示されていますか？

**解決方法:**
```bash
# すべてのプロセスを終了
Ctrl + C

# クリーンビルド
npm run build
cargo clean --manifest-path=src-tauri/Cargo.toml

# 再起動
npm run tauri dev
```

---

### エラー: IDが生成されない

**原因:** Tauriバックエンドが起動していない

**解決方法:**
1. `npm run tauri dev` で起動
2. 設定画面で「新しいIDを生成」ボタンをクリック
3. 初回起動時は自動生成されます

---

## 💡 開発のヒント

### ホットリロード
- Reactコード (src/**/*.tsx) を変更すると自動リロード
- Rustコード (src-tauri/**/*.rs) を変更すると自動再コンパイル

### デバッグ
```bash
# Rustバックエンドのログを表示
RUST_LOG=debug npm run tauri dev

# フロントエンドのコンソールを開く
Tauriウィンドウで右クリック → Inspect Element → Console
```

### ビルド（リリース版作成）
```bash
# 本番ビルド
npm run build
npm run tauri build

# 実行ファイルの場所:
# Windows: src-tauri/target/release/prism-vote-app.exe
```

---

## 📱 使い方

詳しい使い方は [USAGE.md](USAGE.md) を参照してください。

---

## ❓ それでも問題が解決しない場合

1. **依存関係の再インストール:**
```bash
rm -rf node_modules
npm install
cargo clean --manifest-path=src-tauri/Cargo.toml
```

2. **Node.jsのバージョン確認:**
```bash
node --version  # v18 以上を推奨
```

3. **Rustのバージョン確認:**
```bash
rustc --version  # 1.70 以上を推奨
```

4. **Githubで問題を報告:**
- エラーメッセージ全文
- 使用したコマンド
- OSとバージョン

---

**楽しい投票体験を！** 🎉
