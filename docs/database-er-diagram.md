# 半田港手解体室アプリ データベースER図

## 概要

半田港手解体室アプリのデータベース設計をER図で視覚化したものです。

## ER図

```mermaid
erDiagram
    %% マスタテーブル
    Company {
        string id PK
        string name UK
        datetime createdAt
        datetime updatedAt
    }
    
    Item {
        string id PK
        string name UK
        ItemCategory category
        UnitType unitType
        boolean isCRT
        boolean hasScale
        datetime createdAt
        datetime updatedAt
    }
    
    Worker {
        string id PK
        string name UK
        datetime createdAt
        datetime updatedAt
    }
    
    WorkType {
        string id PK
        string name UK
        string description
        datetime createdAt
        datetime updatedAt
    }
    
    Destination {
        string id PK
        string name UK
        datetime createdAt
        datetime updatedAt
    }
    
    TroubleType {
        string id PK
        string name UK
        string description
        datetime createdAt
        datetime updatedAt
    }
    
    %% 中間テーブル
    CompanyItem {
        string id PK
        string companyId FK
        string itemId FK
    }
    
    WorkTypeItem {
        string id PK
        string workTypeId FK
        string itemId FK
        ItemRole itemRole
    }
    
    DestinationItem {
        string id PK
        string destinationId FK
        string itemId FK
        float unitPrice
    }
    
    %% トランザクションテーブル
    Arrival {
        string id PK
        datetime date
        string companyId FK
        float totalAmount
        datetime createdAt
        datetime updatedAt
    }
    
    ArrivalItem {
        string id PK
        string arrivalId FK
        string itemId FK
        float weight
        float amount
    }
    
    WorkRecord {
        string id PK
        string workerId FK
        string workTypeId FK
        string itemId FK
        datetime startTime
        datetime endTime
        float quantity
        UnitType unitType
        ScaleType scaleType
        boolean isConfirmed
        string troubleTypeId FK
        string notes
        datetime createdAt
        datetime updatedAt
    }
    
    Shipment {
        string id PK
        string destinationId FK
        string factoryName
        datetime shipmentDate
        float totalAmount
        string lotNumber UK
        ShipmentType shipmentType
        string approvalStatus
        string applicationNo
        string subject
        string refineryLotNo
        datetime createdAt
        datetime updatedAt
    }
    
    ShipmentItem {
        string id PK
        string shipmentId FK
        string itemId FK
        float unitPrice
        float quantity
        string location
    }
    
    Inventory {
        string id PK
        string itemId FK,UK
        float quantity
        UnitType unitType
        datetime lastUpdated
    }
    
    %% リレーション定義
    
    %% 企業関連
    Company ||--o{ CompanyItem : "has"
    Company ||--o{ Arrival : "supplies"
    
    %% 品目関連
    Item ||--o{ CompanyItem : "belongs to"
    Item ||--o{ WorkTypeItem : "used in"
    Item ||--o{ ArrivalItem : "arrived as"
    Item ||--o{ WorkRecord : "processed as"
    Item ||--o{ ShipmentItem : "shipped as"
    Item ||--o{ DestinationItem : "sold to"
    Item ||--|| Inventory : "has stock"
    
    %% 作業者関連
    Worker ||--o{ WorkRecord : "performs"
    
    %% 作業内容関連
    WorkType ||--o{ WorkTypeItem : "involves"
    WorkType ||--o{ WorkRecord : "performed as"
    
    %% 出荷先関連
    Destination ||--o{ DestinationItem : "receives"
    Destination ||--o{ Shipment : "receives"
    
    %% トラブル関連
    TroubleType ||--o{ WorkRecord : "occurs in"
    
    %% 入荷関連
    Arrival ||--o{ ArrivalItem : "contains"
    
    %% 出荷関連
    Shipment ||--o{ ShipmentItem : "contains"
```

## エンティティ詳細

### マスタエンティティ

#### Company（企業マスタ）
- **目的**: 処理物を持ち込む企業の管理
- **主要属性**: 企業名
- **関連**: 入荷情報、企業-品目紐づけ

#### Item（品目マスタ）
- **目的**: 処理物・製品の品目管理
- **主要属性**: 品目名、カテゴリ（処理物/製品）、単位種別（重量/個数）、CRT判定
- **関連**: 全ての業務プロセスで使用

#### Worker（作業者マスタ）
- **目的**: 作業者の管理
- **主要属性**: 作業者名
- **関連**: 作業記録

#### WorkType（作業内容マスタ）
- **目的**: 解体・分別作業の種類管理
- **主要属性**: 作業内容名、説明
- **関連**: 作業記録、作業内容-品目紐づけ

#### Destination（出荷先マスタ）
- **目的**: 製品の出荷先管理
- **主要属性**: 出荷先名
- **関連**: 出荷情報、出荷先-製品紐づけ

#### TroubleType（トラブル情報マスタ）
- **目的**: 作業中のトラブル種類管理
- **主要属性**: トラブル名、説明
- **関連**: 作業記録

### 中間エンティティ

#### CompanyItem（企業-品目紐づけ）
- **目的**: 企業が持ち込む処理物品目の関連管理
- **ビジネスルール**: 企業選択時に該当する処理物品目を絞り込み

#### WorkTypeItem（作業内容-品目紐づけ）
- **目的**: 作業内容と処理物・製品の関連管理
- **ビジネスルール**: 作業内容選択時に該当する処理物・製品を絞り込み

#### DestinationItem（出荷先-製品紐づけ）
- **目的**: 出荷先と製品の関連・単価管理
- **ビジネスルール**: 出荷先選択時に該当する製品と単価を絞り込み

### トランザクションエンティティ

#### Arrival / ArrivalItem（入荷情報）
- **目的**: 処理物の入荷記録
- **ビジネスルール**: 入荷時に在庫を増加

#### WorkRecord（作業記録）
- **目的**: 作業者の作業実績記録
- **ビジネスルール**: 
  - 大坪・小坪管理（対象品目のみ）
  - 大坪集計時に小坪入力を確定
  - CRT品目は個数管理も可能

#### Shipment / ShipmentItem（出荷情報）
- **目的**: 製品の出荷記録
- **ビジネスルール**: 
  - ロットNo.自動生成（YYYYMMDDXXN）
  - 出荷時に在庫を減少

#### Inventory（在庫管理）
- **目的**: 品目別在庫情報
- **ビジネスルール**: 入荷・作業・出荷により自動更新

## 主要なビジネスルール

### 1. マスタ選択による絞り込み
```mermaid
graph LR
    A[企業選択] --> B[CompanyItem検索]
    B --> C[該当処理物品目表示]
    
    D[作業内容選択] --> E[WorkTypeItem検索]
    E --> F[該当処理物・製品表示]
    
    G[出荷先選択] --> H[DestinationItem検索]
    H --> I[該当製品・単価表示]
```

### 2. 在庫更新フロー
```mermaid
graph TD
    A[入荷] --> B[在庫増加]
    C[作業完了] --> D[処理物在庫減少]
    D --> E[製品在庫増加]
    F[出荷] --> G[製品在庫減少]
```

### 3. 小秤・大秤管理
```mermaid
graph TD
    A[小秤入力] --> B[未確定状態]
    B --> C[大秤集計]
    C --> D[小秤入力確定]
    D --> E[在庫更新]
```

## データ整合性制約

### 主キー制約
- 全テーブルで`id`フィールドが主キー
- `lotNumber`（ロットNo.）はユニーク制約

### 外部キー制約
- 全ての関連テーブル間で外部キー制約を設定
- カスケード削除は中間テーブルのみ適用

### ビジネス制約
- 企業名、品目名、作業者名等はユニーク制約
- CRT品目のみ個数管理可能
- 大秤入力時は対応する小秤入力が存在する必要がある

## インデックス設計

### パフォーマンス重視のインデックス
- `work_records.start_time`: 作業記録の日付検索
- `arrivals.date`: 入荷情報の日付検索
- `shipments.shipment_date`: 出荷情報の日付検索
- `shipments.lot_number`: ロットNo.検索（ユニーク）

### 検索頻度の高いフィールド
- 各マスタテーブルの`name`フィールド
- 各中間テーブルの外部キーフィールド
