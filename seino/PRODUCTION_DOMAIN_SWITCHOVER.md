# 本番ドメイン切替手順（→ `agent.seino-ss.jp`）

想定本番URL: `https://agent.seino-ss.jp/`

## 1. WordPress 設定
WordPress 管理画面 → 設定 → 一般
- **WordPress アドレス (URL)**: `https://agent.seino-ss.jp`
- **サイトアドレス (URL)**: `https://agent.seino-ss.jp`

DBに直接書く場合（または WP-CLI 推奨）:
```bash
wp option update siteurl 'https://agent.seino-ss.jp'
wp option update home 'https://agent.seino-ss.jp'
wp search-replace 'http://localhost:8090' 'https://agent.seino-ss.jp' --all-tables
```

## 2. SEO/sitemap.xml の更新
`/wp-content/themes/seino-agent/` のテーマでは `home_url()` ベースで動的に解決するため、SEOメタタグ（canonical, og:url, JSON-LD url など）は自動で本番URLになります。

ただし静的にデプロイした GitHub Pages 版 `seino/sitemap.xml` は手動で書き換えが必要:
```
https://anyreach.github.io/seino/  →  https://agent.seino-ss.jp/
*.html  →  / (またはWPのクリーンURL構造に揃える)
```

## 3. JSON-LD の `url` 確認
`functions.php` の `agent_seo_meta()` 内 JSON-LD は `home_url('/')` を参照しているので自動切替。要確認のみ。

## 4. robots.txt
`agent.seino-ss.jp` のルートに以下を配置（または `/seino/robots.txt` 同等）:
```
User-agent: *
Allow: /

Sitemap: https://agent.seino-ss.jp/sitemap.xml
```

## 5. DNS / SSL（セイノー情報サービス様への依頼事項）
- `agent.seino-ss.jp` の A レコード / CNAME を AWS（または契約サーバー）IP へ
- SSL 証明書発行（Let's Encrypt or ACM）
- 反映確認は `dig agent.seino-ss.jp` で

## 6. Search Console / Analytics
- Google Search Console に `https://agent.seino-ss.jp/` を新規プロパティ追加
- sitemap.xml を送信
- Google Analytics 4 のプロパティ作成・GTM タグ設置

## 7. 旧URLからのリダイレクト
GitHub Pages の `anyreach.github.io/seino/` から本番への 301 リダイレクトは Pages では不可（静的のため）。プレビュー用なのでそのまま残し、検索インデックスから外したい場合は `robots.txt` で `Disallow: /` に戻す。

---

## まだ未確定の項目
- **本番サーバー**: AWS（セイノー情報サービス様のアカウント配下 or AnyMind AWS）/ レンタルサーバー — 検討中
- **DNS 反映タイミング**: セイノー情報サービス様の作業リードタイム次第
- **Toroo プラグインの設置先 URL** の確定: NDA 締結後にダトラ社へ FTP/SFTP 情報共有
