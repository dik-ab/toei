# 半田港手解体室アプリ 設計ドキュメント

## 概要

半田港手解体室における廃棄物処理・リサイクル業務を効率化するためのWebアプリケーションの設計ドキュメント集です。

## プロジェクト構成

```
半田港手解体室アプリ
├── 管理者用Webアプリ（事務所PC）
├── 作業者用タブレットアプリ（作業場）
└── AWS Amplify Gen2バックエンド
```

## ドキュメント一覧

### 📊 [データベースER図](./database-er-diagram.md)
- エンティティ関係図（Mermaid形式）
- テーブル間のリレーション
- ビジネスルールの視覚化

### 🗄️ [データベース設計書](./database-design.md)
- テーブル構造詳細
- フィールド定義
- インデックス設計
- セキュリティ・運用考慮事項

### 💾 [Prismaスキーマ](./database-schema.prisma)
- データベーススキーマ定義
- リレーション設定
- Enum定義

## 主要機能

### 管理者用機能（事務所PC）
- ✅ 入荷情報管理
- ✅ 出荷情報管理
- ✅ マスタデータ管理
- ✅ 在庫管理・分析
- ✅ レポート・CSV出力

### 作業者用機能（タブレット）
- ✅ 作業記録入力
- ✅ 処理量入力（小秤・大秤対応）
- ✅ 在庫確認
- ✅ トラブル報告

## 技術スタック

### フロントエンド
- **フレームワーク**: Next.js 14 (App Router)
- **言語**: TypeScript
- **スタイリング**: Tailwind CSS
- **UI**: Radix UI + shadcn/ui

### バックエンド
- **API**: AWS Amplify Gen2 GraphQL
- **認証**: AWS Cognito
- **データベース**: DynamoDB
- **ストレージ**: S3

### 開発環境
- **モノレポ**: Turborepo
- **パッケージマネージャー**: npm

## データベース概要

### マスタテーブル
- **Company**: 企業マスタ
- **Item**: 品目マスタ（処理物・製品）
- **Worker**: 作業者マスタ
- **WorkType**: 作業内容マスタ
- **Destination**: 出荷先マスタ
- **TroubleType**: トラブル情報マスタ

### 中間テーブル
- **CompanyItem**: 企業-品目紐づけ
- **WorkTypeItem**: 作業内容-品目紐づけ
- **DestinationItem**: 出荷先-製品紐づけ

### トランザクションテーブル
- **Arrival/ArrivalItem**: 入荷情報
- **WorkRecord**: 作業記録
- **Shipment/ShipmentItem**: 出荷情報
- **Inventory**: 在庫管理

## 主要なビジネスルール

### 1. マスタ選択による絞り込み
- 企業選択 → 該当処理物品目を絞り込み
- 作業内容選択 → 該当処理物・製品を絞り込み
- 出荷先選択 → 該当製品・単価を絞り込み

### 2. 小秤・大秤管理
- 小秤入力: 薄い青背景 (#E3F2FD)
- 大秤入力: 薄い緑背景 (#E8F5E8)
- 大秤集計時に小秤入力を確定値として処理

### 3. CRT特別管理
- CRT品目のみ個数管理も可能
- 後日まとめて入力対応

### 認証・認可
- **AWS Cognito**による認証
- **管理者**（admin）: 管理アプリ
- **作業者**（worker）: 作業記録・在庫確認


## 参考資料

### AWS Amplify Gen2
- [公式ドキュメント](https://docs.amplify.aws/gen2/)
- [GraphQL API](https://docs.amplify.aws/gen2/build-a-backend/data/)
- [認証](https://docs.amplify.aws/gen2/build-a-backend/auth/)

### Next.js
- [公式ドキュメント](https://nextjs.org/docs)
- [App Router](https://nextjs.org/docs/app)

### Turborepo
- [公式ドキュメント](https://turbo.build/repo/docs)

## 更新履歴

| 日付 | バージョン | 更新内容 |
|------|-----------|----------|
| 2025-06-08 | 1.0.0 | 初版作成 |

---

**作成者**: システム設計チーム  
**最終更新**: 2025年6月8日
