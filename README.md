# Shimo's Card Cabinet

メルカリ購入者への特典として提供する、**閲覧専用**のポケモンカードコレクション・ビューワーです。

購入者はビューワーとして閲覧するのみで、カードの追加や削除は開発者（自分）側のみが行います。

---

## アーキテクチャ

```
pokemon-viewer/
├── index.html          ← ビューワー本体（共通）
├── data/
│   ├── taro.json       ← たろうさん用のコレクションデータ
│   ├── jiro.json       ← じろうさん用のコレクションデータ
│   └── demo.json       ← デモ用
└── README.md
```

### URLの仕組み

- `https://<username>.github.io/pokemon-viewer/#taro` → `data/taro.json` を読み込む
- `https://<username>.github.io/pokemon-viewer/#jiro` → `data/jiro.json` を読み込む
- `#` なしでアクセス → デモ表示

URLハッシュ（`#taro`の部分）は、購入者識別のためのスラグです。英数字・ハイフン・アンダースコアのみ有効。

### 編集権限

- GitHubリポジトリをpublicでもprivateでも構いませんが、**編集権限は自分のみ**に設定しておけば購入者は編集できません。
- 購入者はブラウザで表示するだけ。localStorageも使わないので、ユーザー側のデータ改変は反映されません。

---

## デプロイ手順（初回）

### GitHub Pages を使う場合

1. GitHub で `pokemon-viewer` という名前のpublicリポジトリを作成
2. このフォルダ一式をpush
3. リポジトリの **Settings → Pages** で、ブランチを `main` / フォルダを `/ (root)` に設定
4. 数分後、`https://<username>.github.io/pokemon-viewer/` で公開される

### Cloudflare Pages を使う場合（推奨・より高速）

1. [Cloudflare Pages](https://pages.cloudflare.com/) にログイン
2. GitHubリポジトリと連携
3. Build command: なし / Build output: `/`
4. 独自ドメインもタダで付けられる（例: `shimo-cards.dev`）

---

## 購入者ごとのコレクション追加手順

### 新規購入者のために新しいキャビネットを作る

1. `data/` 配下に `{購入者スラグ}.json` を作成（例: `data/hanako.json`）
2. 下記テンプレートをベースに編集

```json
{
  "owner": {
    "name": "はなこ 様"
  },
  "collection": {
    "title": "HANAKO'S COLLECTION",
    "welcome": "このたびはお取引いただきありがとうございました。",
    "updated_at": "2026-04-19"
  },
  "cards": [
    {
      "id": "hanako-001",
      "name": "リザードンex SAR",
      "set": "SV2a ポケモンカード151",
      "number": "201/198",
      "rarity": "SAR",
      "front_image": "https://example.com/cards/charizard.jpg",
      "back_image": "",
      "psa": {
        "grade": 10,
        "cert_number": "12345678",
        "year": 2024
      },
      "notes": "センタリング良好。",
      "acquired_at": "2026-04-18",
      "added_at": "2026-04-19"
    }
  ]
}
```

3. `git add data/hanako.json && git commit -m "add: hanako collection" && git push`
4. 購入者へ送るURL: `https://<username>.github.io/pokemon-viewer/#hanako`
5. メルカリ取引メッセージでURLを送付

### 既存購入者にカードを追加する（2回目以降のお取引）

1. 該当JSONを編集し、`cards` 配列に追記
2. `collection.updated_at` を今日の日付に更新
3. commit & push
4. 購入者は**同じURL**をリロードするだけで新しいカードが見える（再送信不要）

---

## カードデータ仕様

### 必須フィールド

| フィールド | 型 | 例 |
|---|---|---|
| `id` | string | `"taro-001"`（ユニーク識別子、何でも可） |
| `name` | string | `"リザードンex SAR"` |

### 推奨フィールド

| フィールド | 型 | 説明 |
|---|---|---|
| `set` | string | セット名（例: `"SV2a ポケモンカード151"`） |
| `number` | string | カードナンバー（例: `"201/198"`） |
| `rarity` | string | `"SAR"`, `"SR"`, `"UR"`, `"AR"`, `"PROMO"` など |
| `front_image` | string (URL) | カード表面の画像URL。省略時はプレースホルダー表示 |
| `back_image` | string (URL) | カード裏面の画像URL（任意） |
| `acquired_at` | string | 購入日 (`"YYYY-MM-DD"`) |
| `added_at` | string | キャビネット追加日。ソートに使用 |
| `notes` | string | 備考（コンディション、梱包メモなど） |

### PSA鑑定済みの場合

```json
"psa": {
  "grade": 10,
  "cert_number": "12345678",
  "year": 2024
}
```

- `grade`: `1`〜`10`（`10`でGEM MINT表示・金ラベル、`9`赤、`7-8`赤、`5-6`ブロンズ、`1-4`グレー）
- `cert_number`: PSA認証番号
- `year`: 鑑定年

生カード（未鑑定）の場合は `"psa": null` または省略。

---

## 画像のホスティング

カード画像の置き場所は以下の選択肢があります：

| 方法 | メリット | デメリット |
|---|---|---|
| **同じGitHubリポジトリの `images/` 配下** | シンプル・無料・確実 | リポジトリ容量制限（1GB推奨） |
| **Cloudflare R2** | 無料枠が太い・高速 | 初期設定やや必要 |
| **imgur等の無料画像ホスト** | 即席で使える | 規約変更リスク |
| **自分のさくらVPS等** | フルコントロール | コスト |

おすすめは **同じリポジトリに `images/taro/001.jpg` のように置く** 方式。`front_image` には `"images/taro/001.jpg"` と相対パスで書けばOK。

---

## 共有方法（メルカリ以外の選択肢）

| 方法 | 用途 |
|---|---|
| **メルカリ取引メッセージ** | メインの送付手段。URL直貼り |
| **独自ドメイン** | `cards.shimo.dev/#taro` のように覚えやすく。技術者ブランディングにも |
| **QRコード化** | メルカリ発送時の挨拶カードに印刷 |
| **短縮URL** | `bit.ly` 等。ただし改ざん不可性を損なうので非推奨 |
| **プロフィール誘導** | メルカリのプロフ欄に「ご購入者様へ → bit.ly/...」と記載し、購入後個別にスラグを教える |

もっともシンプルかつ洗練されているのは、**独自ドメイン + QRコード印刷**の組み合わせ。

---

## セキュリティ上の注意

- JSONファイル名（スラグ）は、推測されにくい値にすると安心（例: `taro-a3f9k`）
- ただしGitHub publicリポジトリの場合、ファイル一覧は誰でも見られるので注意
- **privateリポジトリ + Cloudflare Pages** なら、スラグを知らなければアクセスできず、かつ一覧も見えない
- 機微情報（購入金額など）はJSONに書かない
