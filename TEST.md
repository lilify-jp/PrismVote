# Prism Vote - 完全テストガイド

このドキュメントでは、Prism Voteのすべての機能を体系的にテストする方法を説明します。

## 目次

1. [環境準備](#環境準備)
2. [ブートストラップノードの確認](#ブートストラップノードの確認)
3. [クライアントアプリのテスト](#クライアントアプリのテスト)
4. [P2Pネットワークのテスト](#p2pネットワークのテスト)
5. [ブラインド署名のテスト](#ブラインド署名のテスト)
6. [投票機能のテスト](#投票機能のテスト)
7. [トラブルシューティング](#トラブルシューティング)

---

## 環境準備

### 必要なソフトウェア
- Node.js 18以上
- Rust 1.70以上
- SSH クライアント（VPS監視用）

### インストール確認
```bash
# Node.jsバージョン確認
node --version  # v18.x.x以上

# Rustバージョン確認
cargo --version  # 1.70.0以上

# 依存関係インストール
npm install
```

---

## ブートストラップノードの確認

### 1. VPSへのSSH接続
```bash
ssh -i C:\Users\Mttm2\Downloads\abc.pem root@162.43.75.151
```

### 2. サービス状態確認
```bash
# ステータス確認
systemctl status prism-bootstrap

# 期待される出力:
# ● prism-bootstrap.service - Prism Vote Bootstrap Node
#      Active: active (running)
```

### 3. リアルタイムログ監視
```bash
# ログをリアルタイムで表示
journalctl -u prism-bootstrap -f

# 期待される出力:
# [INFO  prism_bootstrap] 🚀 Starting Prism Bootstrap Node
# [INFO  prism_bootstrap] 📋 Peer ID: 12D3KooWEDyAUWfAegnWFbbFMW2Am8EypBUKJAiNnJf6R3fgPnPa
# [INFO  prism_bootstrap] ✅ Bootstrap node is ready!
```

### 4. ファイアウォール確認
```bash
# ポート4001が開いているか確認
ufw status | grep 4001

# 期待される出力:
# 4001/tcp                   ALLOW       Anywhere
```

### 5. 外部からの接続テスト
```bash
# 別のターミナルから（Windows側）
telnet 162.43.75.151 4001

# または
Test-NetConnection -ComputerName 162.43.75.151 -Port 4001
```

**✅ チェックポイント:**
- [ ] ブートストラップノードが起動している
- [ ] Peer IDが表示されている
- [ ] ポート4001が開放されている
- [ ] 外部から接続可能

---

## クライアントアプリのテスト

### 1. アプリ起動
```bash
# 開発モードで起動
npm run tauri dev
```

### 2. 起動確認
**期待される動作:**
- Viteサーバーが起動（http://127.0.0.1:1420）
- Tauriウィンドウが開く
- ホーム画面が表示される

**ログ確認:**
```bash
# 別のターミナルでログを確認
tail -f test.log

# または直接ターミナル出力を見る
```

**期待されるログ:**
```
P2P node started on /ip4/0.0.0.0/tcp/0
Peer ID: 12D3KooW...
🔗 Connecting to bootstrap node: /ip4/162.43.75.151/tcp/4001/p2p/12D3KooWEDyAUWfAegnWFbbFMW2Am8EypBUKJAiNnJf6R3fgPnPa
✅ Dialed bootstrap node
📍 Added bootstrap node to Kademlia DHT
🚀 Kademlia bootstrap initiated
```

**✅ チェックポイント:**
- [ ] アプリウィンドウが開く
- [ ] P2Pノードが起動する
- [ ] ブートストラップノードにダイヤルする
- [ ] エラーが出ていない

### 3. UI動作確認

#### ホーム画面
- [ ] 「新規投票作成」ボタンが表示される
- [ ] 空の状態メッセージが表示される

#### 設定画面
- [ ] 左サイドバーの「設定」をクリック
- [ ] 匿名IDが表示される（または生成ボタンが表示される）
- [ ] P2Pネットワークセクションが表示される

---

## P2Pネットワークのテスト

### 1. P2Pノード起動テスト

#### 設定画面でP2P起動
1. 設定画面に移動
2. 「P2Pノードを起動」ボタンをクリック
3. ボタンがローディング状態になる
4. アラート「P2Pノードを起動しました」が表示される

**期待される状態:**
```
✅ P2P接続中
📊 接続ピア数: 0 (または1以上)
🆔 Peer ID: 12D3KooW... (表示される)
```

#### VPS側でログ確認
```bash
# VPSのターミナルで
journalctl -u prism-bootstrap -f -n 20
```

**期待されるログ（VPS側）:**
```
[INFO] ⬇️  Incoming connection from ...
[INFO] ✅ Connection established with 12D3KooW... (Total: 1)
[INFO] 🤝 Identified peer: 12D3KooW... (Agent: rust-libp2p/0.53.2)
[INFO] 🗺️  Routing updated for peer: 12D3KooW...
[INFO] 👥 Peer 12D3KooW... subscribed to "prism-votes"
```

**✅ チェックポイント:**
- [ ] P2P起動ボタンが機能する
- [ ] VPS側に接続イベントが表示される
- [ ] Identifyプロトコルでピア情報が交換される
- [ ] Gossipsubトピックに購読される
- [ ] Kademliaルーティングが更新される

### 2. 複数ノードテスト

#### 方法1: 別のマシン/デバイスで起動

**推奨方法** - 最も簡単で現実的なテスト方法

```bash
# マシン1（メインPC）
npm run tauri dev

# マシン2（別のPC/ノートパソコン/タブレット）
git clone <repository>
npm install
npm run tauri dev
```

**期待される動作:**
1. 両方のノードがブートストラップノードに接続
2. Kademlia DHT経由で相互に発見
3. mDNS（同じWi-Fi）またはグローバルDHTで接続

#### 方法2: Windows リリースビルド + 開発モード（同一マシン）

```bash
# ターミナル1: リリースビルドを作成
npm run tauri build
cd src-tauri/target/release
./prism-vote-app.exe

# ターミナル2: 開発モードで起動
npm run tauri dev
```

**注意:**
- リリースビルドと開発ビルドは異なるポートを使用
- P2Pノードは動的にポートを割り当てる（port: 0）
- Vite開発サーバー（1420）はリリースビルドでは使わない

#### 方法3: Docker（上級者向け）

```bash
# Dockerfile作成（別途必要）
docker build -t prism-vote .
docker run -p 4002:4001 prism-vote
docker run -p 4003:4001 prism-vote
```

**VPS側のログ（2ノード接続時）:**
```
[INFO] ✅ Connection established with 12D3KooW[Node1]... (Total: 1)
[INFO] ✅ Connection established with 12D3KooW[Node2]... (Total: 2)
```

**クライアント側のログ:**
```
Discovered peer: 12D3KooW... at /ip4/192.168.x.x/tcp/xxxxx
Connection established with: 12D3KooW...
🤝 Identified peer: 12D3KooW...
```

**✅ チェックポイント:**
- [ ] 2つのノードが起動する
- [ ] 両方がブートストラップノードに接続
- [ ] DHTで相互発見が動作
- [ ] 設定画面で「接続ピア数: 1」と表示される

**⚠️ 重要:**
- 同一マシンで `npm run tauri dev` を2回実行すると **Viteポート（1420）が競合** します
- 複数ノードテストは **別のデバイス** または **リリースビルド** を使用してください

### 3. ネットワーク分離テスト

#### 異なるネットワークからの接続
1. スマートフォンのテザリングでPCを接続
2. または別のWi-Fiネットワークに接続
3. アプリを起動

**期待される動作:**
- mDNSでは発見されない（異なるネットワーク）
- ブートストラップノード経由でDHTを通じて発見される
- グローバルP2Pネットワークとして機能

**✅ チェックポイント:**
- [ ] 異なるネットワークでもブートストラップノードに接続できる
- [ ] Kademlia DHT経由でピアを発見できる
- [ ] ローカルネットワークに依存しない

---

## ブラインド署名のテスト

### 1. Rustユニットテスト

```bash
# すべてのテストを実行
cargo test --lib

# ブラインド署名のテストのみ実行
cargo test --lib blind_signature

# 詳細出力付き
cargo test --lib blind_signature -- --nocapture
```

**期待される出力:**
```
running 5 tests
test crypto::blind_signature::tests::test_blind_signature_flow ... ok
test crypto::blind_signature::tests::test_verify_signature ... ok
test crypto::blind_signature::tests::test_invalid_signature ... ok
test crypto::blind_signature::tests::test_extended_gcd ... ok
test crypto::blind_signature::tests::test_mod_inverse ... ok

test result: ok. 5 passed
```

### 2. 数学的検証

#### テストコードの確認
```bash
# テストコードを確認
cat src/crypto/blind_signature.rs | grep -A 30 "#\[cfg(test)\]"
```

**検証項目:**
1. **ブラインド化**: `m' = H(m) * r^e mod n`
2. **署名**: `s' = (m')^d mod n`
3. **アンブラインド化**: `s = s' * r^(-1) mod n`
4. **検証**: `s^e mod n == H(m)`

### 3. 実際の投票での検証

後述の「投票機能のテスト」セクションで、実際の投票フローでブラインド署名が機能することを確認します。

**✅ チェックポイント:**
- [ ] 全ユニットテストが通過
- [ ] 拡張ユークリッド互除法が正しく動作
- [ ] モジュラ逆元計算が正確
- [ ] 署名検証が成功する

---

## 投票機能のテスト

### 1. 投票作成テスト

#### 新規投票を作成
1. ホーム画面の「新規投票作成」ボタンをクリック
2. 以下を入力:
   - タイトル: "ランチどこ行く？"
   - 選択肢1: "ラーメン"
   - 選択肢2: "カレー"
   - 選択肢3: "寿司"
   - 締切: 24時間後
3. 「投票を作成」ボタンをクリック

**期待される動作:**
- 投票カードがホーム画面に表示される
- ステータスが「進行中」
- 投票数が「0票」

**ログ確認:**
```bash
tail -f test.log | grep -E "Creating poll|Poll created"
```

**✅ チェックポイント:**
- [ ] 投票が作成される
- [ ] ホーム画面に表示される
- [ ] データベースに保存される

### 2. 投票実行テスト

#### 投票を投じる
1. 作成した投票カードをクリック
2. 選択肢「ラーメン」を選択
3. 「🗳️ 投票する」ボタンをクリック

**期待される動作:**
- ローディング表示
- 成功メッセージ「投票が完了しました！」
- 投票数が「1票」に増える

**ログ確認:**
```bash
tail -f test.log | grep -E "blind|signature|vote"
```

**期待されるログ:**
```
Starting blind signature process for vote
Message blinded successfully
Blind signature created
Signature unblinded successfully
Signature verified successfully
Vote cast successfully
```

**✅ チェックポイント:**
- [ ] 投票ボタンが機能する
- [ ] ブラインド署名フローが完了する
- [ ] 署名検証が成功する
- [ ] 投票が記録される

### 3. 重複投票防止テスト

#### 同じ投票に再度投票
1. 同じ投票カードを再度クリック
2. 別の選択肢「カレー」を選択
3. 「🗳️ 投票する」ボタンをクリック

**期待される動作:**
- エラーメッセージ「投票に失敗しました」
- または「すでに投票済みです」
- 投票数は変わらない

**ログ確認:**
```bash
tail -f test.log | grep -E "already voted|duplicate"
```

**期待されるログ:**
```
Error: "You have already voted in this poll"
```

**✅ チェックポイント:**
- [ ] 重複投票がブロックされる
- [ ] エラーメッセージが表示される
- [ ] 投票数が不正に増えない
- [ ] 匿名IDベースの追跡が機能する

### 4. 異なる匿名IDでの投票テスト

#### 新しい匿名IDを生成
1. 設定画面に移動
2. 「新しいIDを生成」ボタンをクリック
3. 新しい匿名IDが表示される

#### 同じ投票に再度投票
1. ホーム画面に戻る
2. 同じ投票カードをクリック
3. 選択肢「カレー」を選択
4. 投票する

**期待される動作:**
- 投票が成功する（新しい匿名ID）
- 投票数が「2票」に増える

**✅ チェックポイント:**
- [ ] 新しい匿名IDが生成される
- [ ] 新しいIDで投票できる
- [ ] 投票が正しくカウントされる

### 5. 投票結果確認テスト

#### 集計結果を表示
1. 投票カードをクリック
2. 「結果を見る」タブをクリック（進行中の場合は投票タブ）

**期待される表示:**
- 各選択肢の投票数
- パーセンテージ
- 総投票数
- 検証済みバッジ（✓）

**✅ チェックポイント:**
- [ ] 投票数が正確
- [ ] パーセンテージが正しい
- [ ] バーチャートが表示される
- [ ] 検証済みマークが表示される

---

## P2P投票共有テスト（高度）

### 1. 2つのノード間での投票共有

#### ノード1で投票作成
1. ノード1で新規投票を作成
2. タイトル: "テストP2P投票"

#### ノード2で投票を確認
1. ノード2のホーム画面を確認
2. Gossipsub経由で投票が同期される（将来実装）

**現在の状態:**
- ✅ P2Pネットワークは接続済み
- ⏳ 投票データの自動同期は未実装
- 📝 今後の改善: Gossipsubでブロードキャスト

**✅ チェックポイント:**
- [ ] 両ノードがP2P接続されている
- [ ] Gossipsubトピックに購読されている
- [ ] 将来の同期機能の基盤が整っている

---

## トラブルシューティング

### 問題1: ブートストラップノードに接続できない

**症状:**
```
⚠️  Failed to dial bootstrap node
```

**確認項目:**
1. VPSのサービスが起動しているか
   ```bash
   ssh -i abc.pem root@162.43.75.151 "systemctl status prism-bootstrap"
   ```

2. ファイアウォールが開いているか
   ```bash
   ssh -i abc.pem root@162.43.75.151 "ufw status | grep 4001"
   ```

3. Peer IDが正しいか
   ```bash
   # p2p.rsで定義されているPeer ID
   12D3KooWEDyAUWfAegnWFbbFMW2Am8EypBUKJAiNnJf6R3fgPnPa
   ```

**解決策:**
```bash
# VPSでサービス再起動
systemctl restart prism-bootstrap

# ファイアウォール再設定
ufw allow 4001/tcp
ufw reload
```

### 問題2: P2Pノードが起動しない

**症状:**
```
P2P node start failed
```

**確認項目:**
1. ポートが使用中でないか
   ```bash
   netstat -ano | findstr :4001
   ```

2. libp2pの依存関係
   ```bash
   cargo build 2>&1 | grep libp2p
   ```

**解決策:**
```bash
# 既存のプロセスを停止
taskkill //F //IM prism-vote-app.exe

# 再ビルド
cargo clean
cargo build --release
```

### 問題3: 投票に失敗する

**症状:**
```
投票に失敗しました
```

**確認項目:**
1. 匿名IDが生成されているか（設定画面）
2. キーペアが生成されているか
   ```bash
   tail -f test.log | grep "Keypair generated"
   ```

3. ブラインド署名エラー
   ```bash
   tail -f test.log | grep -E "blind|signature|error"
   ```

**解決策:**
```bash
# 設定画面で新しいIDを生成
# アプリを再起動
npm run tauri dev
```

### 問題4: 白画面/フリーズ

**症状:**
- アプリが真っ白
- 操作できない

**確認項目:**
1. Tauri APIの許可設定
   ```json
   // tauri.conf.json
   "allowlist": {
     "all": true
   }
   ```

2. CSPポリシー
   ```json
   "security": {
     "csp": "default-src 'self'; ..."
   }
   ```

**解決策:**
```bash
# 設定ファイル確認
cat src-tauri/tauri.conf.json | grep -A 5 allowlist

# 再起動
npm run tauri dev
```

### 問題5: ビルドエラー

**症状:**
```
error[E0433]: unresolved import
```

**確認項目:**
1. Cargo.tomlの依存関係
   ```bash
   cat Cargo.toml | grep libp2p
   cat src-tauri/Cargo.toml | grep libp2p
   ```

2. Rustツールチェーン
   ```bash
   rustc --version
   cargo --version
   ```

**解決策:**
```bash
# 依存関係を再インストール
cargo clean
cargo update
cargo build

# Rustツールチェーン更新
rustup update
```

---

## 完全テストチェックリスト

### 環境
- [ ] Node.js 18以上がインストールされている
- [ ] Rust 1.70以上がインストールされている
- [ ] 依存関係がインストールされている

### ブートストラップノード
- [ ] VPSで起動している
- [ ] Peer IDが表示される
- [ ] ポート4001が開放されている
- [ ] 外部から接続可能

### クライアントアプリ
- [ ] アプリが起動する
- [ ] UIが正しく表示される
- [ ] P2Pノードが自動起動する
- [ ] ブートストラップノードに接続する

### P2Pネットワーク
- [ ] P2Pノード起動ボタンが機能する
- [ ] VPS側に接続イベントが表示される
- [ ] Identifyプロトコルが動作する
- [ ] Kademlia DHTルーティングが更新される
- [ ] 複数ノードが相互発見できる

### ブラインド署名
- [ ] 全ユニットテストが通過する
- [ ] ブラインド化が正しく動作する
- [ ] 署名検証が成功する
- [ ] モジュラ逆元計算が正確

### 投票機能
- [ ] 投票を作成できる
- [ ] 投票を投じることができる
- [ ] ブラインド署名フローが完了する
- [ ] 重複投票がブロックされる
- [ ] 新しいIDで再投票できる
- [ ] 集計結果が正確

### セキュリティ
- [ ] 匿名性が保証される
- [ ] 投票内容が暗号化される
- [ ] 重複投票が防止される
- [ ] 署名検証が機能する

---

## テスト実行例

### 完全テストフロー（推奨）

```bash
# 1. VPS接続（別ターミナル）
ssh -i C:\Users\Mttm2\Downloads\abc.pem root@162.43.75.151
journalctl -u prism-bootstrap -f

# 2. アプリ起動（メインターミナル）
npm run tauri dev

# 3. ログ監視（別ターミナル）
tail -f test.log

# 4. テスト実行
# - P2Pノードを起動
# - 投票を作成
# - 投票を投じる
# - 重複投票を試す
# - 新IDで再投票

# 5. VPSログで確認
# - 接続イベント
# - Identifyイベント
# - ピア数の増加
```

---

## まとめ

このテストガイドに従うことで、Prism Voteのすべての機能を体系的に検証できます。

**重要なポイント:**
1. ✅ VPSのブートストラップノードが**常に起動している**こと
2. ✅ Peer IDが**正しく含まれている**こと
3. ✅ ファイアウォールで**ポート4001が開放されている**こと
4. ✅ クライアントが**自動的にブートストラップノードに接続する**こと
5. ✅ ブラインド署名が**数学的に正しく実装されている**こと

**次のステップ:**
- [ ] 複数デバイスでの同時テスト
- [ ] 異なるネットワークからの接続テスト
- [ ] 長時間稼働テスト
- [ ] 負荷テスト（多数の投票）

---

**最終更新**: 2026-01-12
**テストステータス**: ✅ すべての基本機能が動作確認済み
