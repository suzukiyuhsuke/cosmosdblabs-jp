# Azure Cosmos DBでのインデックス作成

このラボでは、Azure Cosmos DB コンテナーのインデックス作成ポリシーを変更します。書き込みまたは読み取り負荷の高いワークロードのインデックス作成ポリシーを最適化する方法と、さまざまな SQL API クエリ機能のインデックス作成要件について説明します。

> これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## インデックス作成の概要

Azure Cosmos DB はスキーマに依存しないデータベースであり、スキーマやインデックスの管理を処理することなく、アプリケーションを反復処理できます。既定では、Azure Cosmos DB は、スキーマを定義したり、セカンダリ インデックスを構成したりしなくても、コンテナー内のすべての項目のすべてのプロパティに自動的にインデックスを付けます。インデックス作成ポリシーを既定の設定のままにすることを選択した場合は、ほとんどのクエリを最適なパフォーマンスで実行でき、インデックス作成を明示的に検討する必要はありません。ただし、インデックスのプロパティの追加または削除を制御する場合は、Azure Portal、ARM テンプレート、PowerShell、Azure CLI、または任意の Cosmos DB SDK を使用して変更できます。

Azure Cosmos DB では、データをツリー形式で表す転置インデックスが使用されます。このしくみの簡単な概要については、ラボを続行する前に[インデックス作成の概要](https://docs.microsoft.com/en-us/azure/cosmos-db/index-overview)をお読みください。

## インデックス作成ポリシーのカスタマイズ

このラボセクションでは、**FoodCollection**のインデックス作成ポリシーを表示および変更します。

### データエクスプローラーを開く

1. Azureポータルの左側で、**Resource groups**を選択します。

1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

3. **cosmoslabs**ブレードで、**Azure Cosmos DB**アカウントを選択します。

4. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。

5. **データエクスプローラー** セクションで**NutritionDatabase**データベースノードを展開して、さらに**FoodCollection**コンテナーノードを展開します。

6. **FoodCollection**ノードで、**Items**リンクを選択します。

7. コンテナー内の項目を表示します。これらのドキュメントに配列を含む多くのプロパティがあることを確認します。WHERE 句、ORDER BY 句、または JOIN で特定のプロパティを使用しない場合、プロパティのインデックスを作成してもパフォーマンス上の利点はありません。

8. Still within the  node, select the  link.引き続き**FoodCollection**ノード内で、**Scale & Settings**リンクを選択します。
9. **Indexing Policy**セクションを確認する。

    - コンテナーのインデックスを定義する JSON ファイルを編集できることに注意してください。
    - インデックス作成ポリシーは、任意の Azure Cosmos DB SDK および ARM テンプレート、PowerShell、または Azure CLI を使用して変更することもできます。
    - このラボでは、Azure ポータルを使用してインデックス作成ポリシーを変更します。

   ![The Indexing Policy window is highlighted](../media/04-indexingpolicy-initial.jpg "Review the indexing policy of the FoodCollection")

### インデックスの包含と除外

既定ですべてのプロパティにインデックスを含める代わりに、インデックスに特定のパスを含めるか除外するかを選択できます。いくつかの簡単な例を見てみましょう (Azure ポータルに入力する必要はなく、ここで確認できます)。

**FoodCollection**内では、ドキュメントに次のスキーマがあります (わかりやすくするために一部のプロパティが削除されています)。

```json
{
    "id": "36000",
    "_rid": "LYwNAKzLG9ADAAAAAAAAAA==",
    "_self": "dbs/LYwNAA==/colls/LYwNAKzLG9A=/docs/LYwNAKzLG9ADAAAAAAAAAA==/",
    "_etag": "\"0b008d85-0000-0700-0000-5d1a47e60000\"",
    "description": "APPLEBEE'S, 9 oz house sirloin steak",
    "tags": [
        {
            "name": "applebee's"
        },
        {
            "name": "9 oz house sirloin steak"
        }
    ],
    "manufacturerName": "Applebee's",
    "foodGroup": "Restaurant Foods",
    "nutrients": [
        {
            "id": "301",
            "description": "Calcium, Ca",
            "nutritionValue": 16,
            "units": "mg"
        },
        {
            "id": "312",
            "description": "Copper, Cu",
            "nutritionValue": 0.076,
            "units": "mg"
        },
    ]
}
```

manufacturerName、foodGroup、およびnutrientsの配列のみをインデックス化する場合は、次のインデックス ポリシーを定義する必要があります。この例では、ワイルドカード文字`*`を使用して、nutrients配列内のすべてのパスにインデックスを付けることを示します。

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/manufacturerName/*"
        },
        {
            "path": "/foodGroup/*"
        },
        {
            "path": "/nutrients/[]/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/*"
        }
    ]
}
```

ただし、各配列要素の nutritionValue にインデックスを付けるだけにすることもできます。

次の例では、インデックス作成ポリシーで、nutrition配列の nutritionValueパスにインデックスを付ける必要があることを明示的に指定します。配列に対してワイルドカード文字`*`を使用しないため、配列内の追加のパスにはインデックスが付けられません。

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/manufacturerName/*"
        },
        {
            "path": "/foodGroup/*"
        },
        {
            "path": "/nutrients/[]/nutritionValue/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/*"
        }
    ]
}
```

最後に、`*`と`?`の違いを理解することが重要です

`*`は、Azure Cosmos DB がその特定のノードを超えるすべてのパスにインデックスを付ける必要があることを示します。

`?`は、Azure Cosmos DB でこのノードを超えるパスのインデックスを作成しないことを示します。

上記の例では、nutritionValueの下に追加のパスはありません。ドキュメントを変更してここにパスを追加する場合、上記の例でワイルドカード文字を使用すると、名前を明示的に指定せずにプロパティのインデックスが作成されます。

### クエリ要件を理解する

インデックス作成ポリシーを変更する前に、コレクションでデータがどのように使用されるかを理解することが重要です。

ワークロードが書き込み負荷が高い場合、またはドキュメントが大きい場合は、必要なパスのみをインデックス化する必要があります。これにより、挿入、更新、および削除に必要な RU の量が大幅に減少します。

次のクエリが、**FoodCollection**コンテナーで実行される読み取り操作のすべてであると想像してみましょう。

**クエリ#1**

```sql
SELECT * FROM c WHERE c.manufacturerName = <manufacturerName>
```

**クエリ#2**

```sql
SELECT * FROM c WHERE c.foodGroup = <foodGroup>
```

これらのクエリでは、**manufacturerName**と**foodGroup**でインデックスを定義するだけで済みます。インデックス作成ポリシーを変更して、これらのプロパティのみにインデックスを付けることができます。

### パスを含めてインデックス作成ポリシーを編集する

1. Azureポータルでデータエクスプローラーを使い、**FoodCollection**を開きます。
2. **Scale & Settings**リンクを選択します。
3. **Indexing Policy**セクションで、既存の json ファイルを次のように置き換えます。

    ```json
    {
        "indexingMode": "consistent",
        "automatic": true,
        "includedPaths": [
            {
                "path": "/manufacturerName/*"
            },
            {
                "path": "/foodGroup/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
    ```

    > この新しいインデックス作成ポリシーは、manufacturerNameとfoodGroupのプロパティのみにインデックスを作成します。他のすべてのプロパティのインデックスが削除されます。

4. **Save**を選択します。Azure Cosmos DB によってコンテナー内のインデックスが更新され、プロビジョニングされた超過スループットを使用して更新が行われます。

    > コンテナーのインデックスの再作成中、書き込みパフォーマンスは影響を受けません。インデックスの更新中に実行されるクエリは、再構築されるまで新しいインデックス ポリシーを使用しません。

5. メニューで、**New SQL Query**アイコンを選択します。
6. 次の SQL クエリを貼り付け、**Execute Query**を選択します。

    ```sql
    SELECT * FROM c WHERE c.manufacturerName = "Kellogg, Co."
    ```

7. **Query Stats**タブに移動します。このクエリの RU 料金は、インデックスから一部のプロパティを削除した後でも低いことに注意してください。**manufacturerName**はクエリでフィルターとして使用される唯一のプロパティであったため、必要な唯一のインデックスでした。

    ![Query metrics are displayed for the previous query](../media/04-querymetrics_01.JPG "Review the query metrics")

8. クエリテキストを次のように置き換え、**Execute Query**を選択します。

    ```sql
    SELECT * FROM c WHERE c.description = "Bread, blue corn, somiviki (Hopi)"
    ```

9. このクエリでは、1 つのドキュメントしか返されないにもかかわらず、非常に高い RU 料金が発生することに注意してください。これは、`description`プロパティにインデックスが現在定義されていないためです。

10. **Query Stats**を確認します。

    ![Query metrics are displayed for the previous query](../media/04-querymetrics_02.JPG "Review the query metrics")

    > クエリでインデックスを使用しない場合、**Index hit document count**は0になります。上記のように、クエリは8,618のドキュメントを取得する必要があり、最終的には1つのドキュメントしか返さなかったことがわかります。

### パスを除外してインデックス作成ポリシーを編集する

インデックスを作成する特定のパスを手動で含めるだけでなく、特定のパスを除外することもできます。多くの場合、この方法は、ドキュメント内のすべての新しいプロパティに既定でインデックスを作成できるため、より簡単です。クエリで決して使用しないことが確実なプロパティがある場合は、このパスを明示的に除外する必要があります。

**description**プロパティを除くすべてのパスにインデックスを付けるインデックス作成ポリシーを作成します。

1. Azureポータルでデータエクスプローラーを使い、**FoodCollection**を開きます。
2. **Scale & Settings**リンクを選択します。
3. **Indexing Policy**セクションで、既存の json ファイルを次のように置き換えます。

    ```json
    {
        "indexingMode": "consistent",
        "automatic": true,
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/description/*"
            }
        ]
    }
    ```

    > この新しいインデックス作成ポリシーは、descriptionを**除外した**すべてのプロパティにインデックスを作成します。

4. **Save**を選択します。Azure Cosmos DBによってコンテナー内のインデックスが更新され、プロビジョニングされた超過スループットを使用して更新が行われます。

    > コンテナーのインデックスの再作成中、書き込みパフォーマンスは影響を受けません。インデックスの更新中に実行されるクエリは、再構築されるまで新しいインデックスポリシーを使用しません。

5. 新しいインデックス作成ポリシーを定義したら、**FoodCollection**に移動し、**Add New SQL Query**アイコンを選択します。次のSQLクエリを貼り付け、**Execute Query**を選択します。

    ```sql
    SELECT * FROM c WHERE c.manufacturerName = "Kellogg, Co."
    ```

6. **Query Stats**タブに移動します。manufacturerNameにインデックスが作成されるため、このクエリの RU料金はまだ低いことに注意してください。

7. クエリ テキストを次のように置き換え、**Execute Query**を選択します。

    ```sql
    SELECT * FROM c WHERE c.description = "Bread, blue corn, somiviki (Hopi)"
    ```

8. このクエリでは、1 つのドキュメントしか返されないにもかかわらず、非常に高い RU 料金が発生することに注意してください。これは、`description`プロパティがインデックス作成ポリシーで明示的に除外されているためです

## 複合インデックスの追加

複数のプロパティで並べ替える ORDER BY クエリの場合は、複合インデックスが必要です。複合インデックスは複数のプロパティで定義され、手動で作成する必要があります。

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。
2. **データエクスプローラー** セクションで**NutritionDatabase**データベースノードを展開して、さらに**FoodCollection**コンテナーノードを展開します。
3. メニューで、**New SQL Query**アイコンを選択します。
4. 次の SQL クエリを貼り付け、**Execute Query**を選択します。

    ```sql
    SELECT * FROM c ORDER BY c.foodGroup ASC, c.manufacturerName ASC
    ```

    > このクエリは、次のエラーで失敗します。

    ```sql
    "The order by query does not have a corresponding composite index that it can be served from."
    ```

    > 1 つのプロパティを持つ ORDER BY 句を持つクエリを実行するには、既定のインデックスで十分です。ORDER BY 句に複数のプロパティがあるクエリには、複合インデックスが必要です。

5. Still within the  node, select the  link. In the  section, you will add a composite index.引き続き**FoodCollection**ノード内で、**Scale & Settings**リンクを選択します。**Indexing Policy**[スケールと設定] セクションで、複合インデックスを追加します。

6. Replace the **Indexing Policy** with the following text:

    ```json
    {
        "indexingMode": "consistent",
        "automatic": true,
        "includedPaths": [
            {
                "path": "/manufacturerName/*"
            },
            {
                "path": "/foodGroup/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            },
            {
                "path": "/\"_etag\"/?"
            }
        ],
        "compositeIndexes": [
            [
                {
                    "path": "/foodGroup",
                    "order": "ascending"
                },
                {
                    "path": "/manufacturerName",
                    "order": "ascending"
                }
            ]
        ]
    }
    ```

7. **Save**を選択して、 この新しいインデックス作成ポリシーを保存します。更新プログラムがコンテナーに適用されるまでに約 10 秒から 15 秒かかります。
    > このインデックス作成ポリシーは、次のORDER BYクエリを許可する複合インデックスを定義します。これらのそれぞれを、**データエクスプローラー**の既存の開いているクエリタブで実行してテストします。複合インデックスでプロパティの順序を定義する場合、プロパティはORDER BY句の順序と完全に一致するか、いずれの場合も反対の値である必要があります。

8. 次のクエリを実行します。

    ```sql
    SELECT * FROM c ORDER BY c.foodGroup ASC, c.manufacturerName ASC
    ```

9. 現在の複合インデックスでサポートされていない次のクエリを実行します。

    ```sql
    SELECT * FROM c ORDER BY c.foodGroup DESC, c.manufacturerName ASC
    ```

10. このクエリは、追加の複合インデックスがないと実行できません。インデックス作成ポリシーを変更して、追加の複合インデックスを含めます。

    ```json
    {
        "indexingMode": "consistent",
        "automatic": true,
        "includedPaths": [
            {
                "path": "/manufacturerName/*"
            },
            {
                "path": "/foodGroup/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            },
            {
                "path": "/\"_etag\"/?"
            }
        ],
        "compositeIndexes": [
            [
                {
                    "path": "/foodGroup",
                    "order": "ascending"
                },
                {
                    "path": "/manufacturerName",
                    "order": "ascending"
                }
            ],
            [
                {
                    "path": "/foodGroup",
                    "order": "descending"
                },
                {
                    "path": "/manufacturerName",
                    "order": "ascending"
                }
            ]
        ]
    }
    ```

11. クエリを再実行すると、成功するはずです。

> [複合インデックスの定義の詳細については、こちらを参照してください。](https://docs.microsoft.com/azure/cosmos-db/how-to-manage-indexing-policy#composite-indexing-policy-examples).

## 空間インデックスの追加

### 火山データを含む新しいコンテナーを作成する

Azure Cosmos DBでは、GeoJSON形式でのデータのクエリがサポートされています。このラボでは、この形式で指定されたこのコンテナーにサンプルデータをアップロードします。このvolcano.jsonサンプル データは、既存の栄養データセットよりも地理空間クエリに適しています。データセットには、世界中の多くの火山の座標と基本情報が含まれています。

まず、新しいデータベース内にvolcanoesという名前の新しいCosmosコンテナーを作成します。

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。

2. **New Container**アイコンを選択して、新しいコンテナーを追加します

3. **Add Container**ポップアップで、次の操作を実行します。

   - **Database id**フィールドで、**Create new**オプションを選択し、値として**VolcanoDatabase**を入力します。

   - **Share throughput across containers**オプションをオフにします。

      > **Share throughput across containers**をオンにすると、そのデータベースに属するすべてのコンテナー間でスループットが共有されます。**Azure Cosmos DB**データベース内には、スループットを共有するコンテナーのセットと、専用のスループットを持つコンテナーを含めることができます。

   - **Container Id**フィールドには、**VolcanoContainer**を入力します。

   - **Partition key**フィールドには、``/Country``を入力します。

   - **Throughput**フィールドには、``5000``を入力します。

   - **OK**ボタンを選択します。

### サンプルデータのアップロード

1. Azure ポータルで**VolcanoesContainer**に移動する。
2. **Items**セクションを選択します。
3. **Upload Item**を選択します。
4. ポップアップで[VolcanoData.json](../setup/VolcanoData.json)ファイルを選択します。
5. **Upload**を選択します。

### **Volcanoes**コンテナーでの地理空間インデックスの作成

1. **Scale & Settings**リンクを選択します。
2. **Indexing Policy**セクションで、既存のjsonファイルを次のように置き換えます。

    ```json
    {
        "indexingMode": "consistent",
        "automatic": true,
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/\"_etag\"/?"
            }
        ],
        "spatialIndexes": [
            {
                "path": "/*",
                "types": [
                    "Point",
                    "Polygon",
                    "MultiPolygon",
                    "LineString"
                ]
            }
        ]
    }
    ```

> 地理空間インデックス作成は、既定では無効になっています。このインデックス作成ポリシーは、ポイント、ポリゴン、マルチポリゴン、ラインストリングを含むすべての可能な GeoJSON タイプの地理空間インデックスを有効にします。範囲インデックスや複合インデックスと同様に、地理空間インデックスの精度設定はありません。

[Azure Cosmos DB での地理空間データのクエリの詳細については、こちらを参照してください。](https://docs.microsoft.com/en-us/azure/cosmos-db/geospatial#introduction-to-spatial-data).

### Volcano Dataのクエリ

1. Azure ポータルで**VolcanoesContainer**に移動する。
2. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。
3. 以下のクエリを貼り付け、**Execute Query**でクエリを実行します。

    ```sql
    SELECT *
    FROM volcanoes v
    WHERE ST_DISTANCE(v.Location, {
    "type": "Point",
    "coordinates": [-122.19, 47.36]
    }) < 100 * 1000
    AND v.Type = "Stratovolcano"
    AND v["Last Known Eruption"] = "Last known eruption from 1800-1899, inclusive"
    ```

    > このクエリは、座標(-122.19, 47.36)から100km 以内にある1800年から1899年の間に最後に噴火したすべての成層火山を返します。座標はワシントン州レドモンドの座標です。


4. **Query Stats**を確認します。コンテナーには Points の地理空間インデックスがあるため、このクエリでは少量の RU が消費されました。

    ![Query metrics are displayed for the previous query](../media/04-querymetrics_geo.jpg "Review the query metrics")

### サンプルPolygonデータのクエリ

Polygon内のポイントを反時計回りに指定する場合は、座標内の領域をPolygonの領域として定義します。時計回りに指定された Polygonは、その領域を除外した領域を表します。

この概念は、サンプルクエリを通じて調べることができます。

1. Azure ポータルで**VolcanoesContainer**に移動する。
2. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。
3. 以下のクエリを貼り付け、**Execute Query**でクエリを実行します。

    ```sql
    SELECT *
    FROM volcanoes v
    WHERE ST_WITHIN(v.Location, {
        "type":"Polygon",
        "coordinates":[[
            [-123.8, 48.8],
            [-123.8, 44.8],
            [-119.8, 44.8],
            [-119.8, 48.8],
            [-123.8, 48.8]
        ]]
        })
    ```

4. 結果を確認してください、この場合、この領域の中に8つの火山があります。

5. *クエリエディター*で、テキストを次のクエリに置き換えます。

    ```sql
    SELECT *
    FROM volcanoes v
    WHERE ST_WITHIN(v.Location, {
        "type":"Polygon",
        "coordinates":[[
            [-123.8, 48.8],
            [-119.8, 48.8],
            [-119.8, 44.8],
            [-123.8, 44.8],
            [-123.8, 48.8]
        ]]
        })
    ```

6. 結果を確認すると、多くのアイテムが返されていることがわかります。指定した領域の外側に何千もの火山があります。

> GeoJSON polygonを作成する場合、それがクエリ内であろうとアイテム内であろうと、指定された座標の順序が重要です。Azure Cosmos DB では、多角形の形状のを除外する座標は拒否されません。さらに、GeoJSON では、(経度、緯度) の形式で座標を指定する必要があります。

## クリーンアップ

### **FoodCollection**インデックス作成ポリシーの復元

**FoodCollection** インデックス作成ポリシーを、すべてのパスにインデックスが作成されるデフォルト設定に復元する必要があります。

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。
2. **データエクスプローラー** セクションで**NutritionDatabase**データベースノードを展開して、さらに**FoodCollection**コンテナーノードを展開します。
3. **FoodCollection**ノード内で、**Scale & Settings**リンクを選択します。
4. **Indexing Policy**セクションで、既存の json ファイルを次のように置き換えます。

    ```json
    {
        "indexingMode": "consistent",
        "automatic": true,
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/\"_etag\"/?"
            }
        ]
    }
    ```

5. **Save**を選択して、これらの変更を適用します。このインデックス作成ポリシーは、ラボを開始したときと同じインデックス作成ポリシーであり、後続のラボで必要です。

### **VolcanoContainer**を削除する

以降のラボでは、**VolcanoContainer**は必要ありません。このコンテナーは今すぐ削除できます。それ以外の場合は、RU/秒を 400 RU/秒に減らします。

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。
2. **VolcanoContainer**の右にあるオプション``...``を選択して、メニューから**Delete Container**を選択します。
3. コンテナーの名前を確認し、コンテナーを削除します。
4. ブラウザー ウィンドウを閉じます。これで、インデックス作成の実習セクションは完了です。

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。
