# Prism Vote - 分散型匿名投票システム

完全に分散化されたP2Pブラインド署名投票システム

## ✅ 実装完了

### 1. コア機能
- ✅ **RSA ブラインド署名** - 真の数学的実装（XORの偽物ではない）
  - `blind_message`: m' = H(m) * r^e mod n
  - `sign_blinded`: s' = (m')^d mod n
  - `unblind_signature`: s = s' * r^(-1) mod n (拡張ユークリッド互除法)
  - `verify_signature`: s^e mod n == H(m)
- ✅ **匿名投票** - ブラインド署名により完全匿名性を保証
- ✅ **重複投票防止** - 匿名IDベースの投票追跡

### 2. P2Pネットワーク
- ✅ **libp2p 0.53** - 完全実装
- ✅ **Gossipsub** - Pub/subメッセージング
- ✅ **Kademlia DHT** - 分散ハッシュテーブルによるピア発見
- ✅ **mDNS** - ローカルネットワークでの自動ピア発見
- ✅ **Identify Protocol** - ピア情報交換
- ✅ **ブートストラップノード** - VPSにデプロイ済み

### 3. ブートストラップノード
**デプロイ先**: `162.43.75.151:4001`
**Peer ID**: `12D3KooWEDyAUWfAegnWFbbFMW2Am8EypBUKJAiNnJf6R3fgPnPa`

**役割**:
- 新規ノードに既存ピアを紹介
- Kademlia DHTのノードとして機能
- Gossipsub pub/subメッセージングに参加

**特徴**:
- ✅ プライバシー保護 - 投票内容は見れない（暗号化されている）
- ✅ 最小限の役割 - 接続を紹介するだけ、データは保存しない
- ✅ 分散化 - 複数のブートストラップノードで冗長性確保
- ✅ systemdサービスとして自動起動

**監視**:
```bash
ssh -i abc.pem root@162.43.75.151
journalctl -u prism-bootstrap -f
```

### 4. クライアント実装
- ✅ **Tauri + React** - デスクトップGUIアプリ
- ✅ **自動ブートストラップ接続** - 起動時に自動的にVPS上のブートストラップノードに接続
- ✅ **リアルタイムピア数表示** - 接続中のピア数を3秒ごとに更新
- ✅ **イベント駆動アーキテクチャ** - ノンブロッキングP2Pイベントループ

## 技術スタック

### フロントエンド
- React 18
- TypeScript 5
- Vite 5
- Zustand (状態管理)
- TailwindCSS

### バックエンド (Rust)
- Tauri 1.5
- libp2p 0.53 (P2Pネットワーキング)
- RSA 0.9 (暗号化)
- num-bigint (大整数演算)
- tokio 1.35 (非同期ランタイム)

### ブートストラップノード
- 独立したRustバイナリ
- systemdサービス
- ファイアウォール: TCP 4001開放

## P2Pアーキテクチャ

```
【99% 分散型】

クライアント ← WiFi (mDNS) → クライアント
    ↓
ブートストラップノード (VPS)
    ↓
グローバルP2Pネットワーク (Kademlia DHT)
    ↓
世界中のノードと接続
```

**なぜ99%？**
- ブートストラップノードは最初の接続のみに使用
- その後はKademlia DHTで完全P2P
- データは暗号化されており、ブートストラップノードは内容を見れない
- 誰でもブートストラップノードを立てられる（分散化可能）

## 起動方法

### クライアント
```bash
npm run tauri dev
```

### ブートストラップノード（VPS上）
```bash
# 自動起動済み（systemdサービス）
systemctl status prism-bootstrap

# 手動起動
cd /root/prismvote/bootstrap-node
./target/release/prism-bootstrap --port 4001 --external-address "/ip4/162.43.75.151/tcp/4001"
```

## セキュリティ機能

1. **完全匿名性** - RSAブラインド署名により、投票者の身元を完全に隠蔽
2. **重複投票防止** - 匿名IDで重複投票を検出
3. **署名検証** - すべての投票が有効な署名を持つことを検証
4. **暗号化通信** - libp2p Noiseプロトコルで通信を暗号化

## 実装の詳細

### ブラインド署名フロー
1. **クライアント**: メッセージをブラインド化
2. **サーバー**: ブラインドメッセージに署名（内容は見えない）
3. **クライアント**: 署名をアンブラインド
4. **検証**: 元のメッセージと署名が正しいことを確認

### P2Pイベント処理
- mDNS: ローカルピア発見
- Kademlia: グローバルピア発見・DHT
- Gossipsub: メッセージ配信
- Identify: ピア情報交換
- 100msごとにノンブロッキングでイベントをポーリング

## ディレクトリ構造
```
c:\Claude\
├── src\                    # Rustコア実装
│   ├── crypto\            # 暗号化（ブラインド署名）
│   ├── network\           # P2Pネットワーク
│   ├── voting\            # 投票ロジック
│   └── storage\           # データ永続化
├── src-tauri\             # Tauriバックエンド
├── src/                   # Reactフロントエンド
└── bootstrap-node\        # ブートストラップノードサーバー
```

## 今後の改善案
- [ ] 複数のブートストラップノードで冗長性向上
- [ ] 投票結果の分散ストレージ（IPFS/OrbitDB）
- [ ] WebAssemblyでブラウザ版を実装
- [ ] モバイルアプリ（React Native）

---

**ステータス**: ✅ 実用可能
**最終更新**: 2026-01-12
