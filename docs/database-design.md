# 半田港手解体室アプリ データベース設計書

## 概要

半田港手解体室における廃棄物処理・リサイクル業務を管理するためのデータベース設計です。

## システム構成

- **バックエンド**: AWS Amplify Gen2 (GraphQL API + DynamoDB)
- **認証**: AWS Cognito
- **フロントエンド**: Next.js (管理者用Web + 作業者用タブレット)

## データベース構造

### マスタテーブル

#### 1. 企業マスタ (companies)
処理物を持ち込む企業の情報を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| name | String | 企業名（ユニーク） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 2. 品目マスタ (items)
処理物・製品の品目情報を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| name | String | 品目名（ユニーク） |
| category | ItemCategory | 処理物 or 製品 |
| unitType | UnitType | 重量 or 個数 |
| isCRT | Boolean | CRT判定フラグ |
| hasScale | Boolean | 大坪・小坪管理対象フラグ |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 3. 作業者マスタ (workers)
作業者の情報を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| name | String | 作業者名（ユニーク） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 4. 作業内容マスタ (work_types)
解体・分別作業の種類を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| name | String | 作業内容名（ユニーク） |
| description | String? | 説明（任意） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 5. 出荷先マスタ (destinations)
製品の出荷先情報を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| name | String | 出荷先名（ユニーク） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 6. トラブル情報マスタ (trouble_types)
作業中のトラブル種類を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| name | String | トラブル名（ユニーク） |
| description | String? | 説明（任意） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

### 中間テーブル（多対多リレーション）

#### 1. 企業-品目紐づけ (company_items)
企業と処理物品目の関連を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| companyId | String | 企業ID |
| itemId | String | 品目ID |

#### 2. 作業内容-品目紐づけ (work_type_items)
作業内容と処理物・製品の関連を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| workTypeId | String | 作業内容ID |
| itemId | String | 品目ID |
| itemRole | ItemRole | 処理物 or 製品 |

#### 3. 出荷先-製品紐づけ (destination_items)
出荷先と製品の関連・単価を管理

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| destinationId | String | 出荷先ID |
| itemId | String | 品目ID |
| unitPrice | Float | 単価(円/kg) |

### トランザクションテーブル

#### 1. 入荷情報 (arrivals)
処理物の入荷記録

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| date | DateTime | 入荷日 |
| companyId | String | 企業ID |
| totalAmount | Float? | 合計金額（任意） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 2. 入荷品目詳細 (arrival_items)
入荷した品目の詳細

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| arrivalId | String | 入荷情報ID |
| itemId | String | 品目ID |
| weight | Float | 重量 |
| amount | Float | 金額 |

#### 3. 作業記録 (work_records)
作業者の作業実績記録

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| workerId | String | 作業者ID |
| workTypeId | String | 作業内容ID |
| itemId | String | 品目ID |
| startTime | DateTime | 作業開始時刻 |
| endTime | DateTime? | 作業終了時刻（任意） |
| quantity | Float | 処理量 |
| unitType | UnitType | 重量 or 個数 |
| scaleType | ScaleType? | 小坪 or 大坪（対象品目のみ） |
| isConfirmed | Boolean | 確定フラグ |
| troubleTypeId | String? | トラブル情報ID（任意） |
| notes | String? | 備考（任意） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 4. 出荷情報 (shipments)
製品の出荷記録

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| destinationId | String | 出荷先ID |
| factoryName | String | 出荷元工場名（デフォルト: "半田港"） |
| shipmentDate | DateTime | 出荷日 |
| totalAmount | Float | 合計金額 |
| lotNumber | String | トーエイのロットNo. (YYYYMMDDXXN) |
| shipmentType | ShipmentType | 出荷情報1 or 出荷情報2 |
| approvalStatus | String? | 承認状況（出荷情報2用） |
| applicationNo | String? | 申請No.（出荷情報2用） |
| subject | String? | 件名（出荷情報2用） |
| refineryLotNo | String? | 精錬所ロットNo（出荷情報2用） |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

#### 5. 出荷品目詳細 (shipment_items)
出荷した品目の詳細

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| shipmentId | String | 出荷情報ID |
| itemId | String | 品目ID |
| unitPrice | Float | 単価(円/kg) |
| quantity | Float | 数量(kg) |
| location | String | 発生場所（デフォルト: "手解体"） |

#### 6. 在庫管理 (inventories)
品目別在庫情報

| フィールド | 型 | 説明 |
|-----------|----|----|
| id | String | 主キー |
| itemId | String | 品目ID（ユニーク） |
| quantity | Float | 在庫量 |
| unitType | UnitType | 重量 or 個数 |
| lastUpdated | DateTime | 最終更新日時 |

## Enum定義

### ItemCategory (品目カテゴリ)
- `PROCESSING_MATERIAL`: 処理物
- `PRODUCT`: 製品

### UnitType (単位種別)
- `WEIGHT`: 重量
- `COUNT`: 個数

### ItemRole (品目役割)
- `PROCESSING_MATERIAL`: 処理物
- `PRODUCT`: 製品

### ScaleType (秤種別)
- `SMALL_SCALE`: 小坪
- `LARGE_SCALE`: 大坪

### ShipmentType (出荷情報種別)
- `TYPE1`: 出荷情報1（事務所PC入力）
- `TYPE2`: 出荷情報2（JUST.DB自動登録）

## 主要な業務フロー

### 1. 入荷処理
1. 企業選択 → 該当する処理物品目が絞り込まれる
2. 品目・重量・金額を入力
3. 在庫に反映

### 2. 作業処理
1. 作業者・作業内容選択
2. 作業内容選択 → 該当する処理物・製品が絞り込まれる
3. 処理量入力（小秤/大秤の視覚的区別）
4. 大秤集計時に小秤入力を確定

### 3. 出荷処理
1. 出荷先選択 → 該当する製品が絞り込まれる
2. 品目・数量・単価入力
3. ロットNo.自動生成（YYYYMMDDXXN形式）
4. 在庫から減算

## 特殊仕様

### ロットNo.生成ルール
- 形式: `YYYYMMDDXXN`
- YYYY: 年（4桁）
- MM: 月（2桁）
- DD: 日（2桁）
- XX: 工場コード（半田港: "HD"）
- N: 連番（1桁）

### CRT管理
- CRT品目は個数管理も可能
- 後日まとめて入力対応

### 大坪・小坪管理
- 品目マスタの`hasScale`フラグで管理対象を判定
- 小坪入力: 薄い青背景 (#E3F2FD)
- 大坪入力: 薄い緑背景 (#E8F5E8)
- 大坪集計時に小坪入力を確定値として処理

## インデックス設計

### パフォーマンス最適化のための推奨インデックス

```sql
-- 作業記録の日付検索用
CREATE INDEX idx_work_records_start_time ON work_records(start_time);

-- 在庫検索用
CREATE INDEX idx_inventories_item_id ON inventories(item_id);

-- 出荷情報の日付検索用
CREATE INDEX idx_shipments_shipment_date ON shipments(shipment_date);

-- 入荷情報の日付検索用
CREATE INDEX idx_arrivals_date ON arrivals(date);

-- ロットNo.検索用（既にユニーク制約あり）
-- CREATE UNIQUE INDEX idx_shipments_lot_number ON shipments(lot_number);
```

## セキュリティ考慮事項

### 認証・認可
- AWS Cognito による認証
- 管理者・作業者の権限分離
- API レベルでの認可制御

### データ保護
- 機密情報の暗号化
- 監査ログの記録
- バックアップ・復旧手順

## 運用考慮事項

### データ保持期間
- 作業記録: 7年間保持
- 入荷・出荷情報: 7年間保持
- マスタデータ: 論理削除

### バックアップ戦略
- 日次自動バックアップ
- 月次アーカイブ
- 災害復旧計画
