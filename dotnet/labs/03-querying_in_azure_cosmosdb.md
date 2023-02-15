# Azure Cosmos DBでのクエリ

Azure Cosmos DB SQL API アカウントでは、最も使い慣れた一般的なクエリ言語の 1 つである構造化クエリ言語 (SQL) を JSON クエリ言語として使用して、アイテムのクエリを実行できます。このラボでは、これらの豊富なクエリ機能を Azure ポータルから直接使用する方法について説明します。個別のツールやクライアント側のコードは必要ありません。

> これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## クエリの概要

SQL を使用して JSON に対してクエリを実行すると、Azure Cosmos DB はレガシ リレーショナル データベースの利点と NoSQL データベースを組み合わせることができます。サブクエリや集計関数など、多くの豊富なクエリ機能を使用できますが、NoSQL データベースでデータをモデル化する多くの利点は引き続き使用できます。

Azure Cosmos DB では、厳密な JSON 項目のみがサポートされています。型システムと式は、JSON 型のみを処理するように制限されています。詳細については、[JSON specification](https://www.json.org/)に関するページを参照してください。

## はじめてのクエリ

このセクションでは、**FoodCollection**に対してクエリを実行します。このラボを開始する前に、[Lab 02 - ADFを使用したデータのインポート](02-load_data_with_adf.md)を完了しておく必要があります。

最初は`SELECT`、`WHERE`や`FROM`といった基本的な句を使用したクエリを実行します。

### データエクスプローラーを開く

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。
2. **データエクスプローラー** セクションで**NutritionDatabase**データベースノードを展開して、さらに**FoodCollection**コンテナーノードを展開します。
3. **FoodCollection**ノードで、**Items**リンクを選択します。
4. コンテナー内のアイテムを確認します。これらのドキュメントに配列を含む多くのプロパティがあることを確認します。

    ![The NutritionDatabase and FoodCollection is displayed and highlighted](../media/04-food_container.jpg "Browse to the FoodCollection and select an item to review its properties")

5. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

      ![New SQL Query is highlighted](../media/04-new_query.jpg "Create a new SQL Query")

6. 以下のクエリを貼り付け、**Execute Query**でクエリを実行します。

    ```sql
    SELECT *
    FROM food
    WHERE food.foodGroup = "Snacks" and food.id = "19015"
    ```

7. クエリが id が "19015" で、foodGroup が "Snacks" である単一のドキュメントを返したことがわかります。このセクションの残りの部分で使用する**FoodCollection**コンテナー内のアイテムを代表するこのアイテムの構造を調べます。

      ![The query results are displayed](../media/04-query_01_results.jpg "Review the results")

## 検索結果の選択

ドット表記を使用して、結果に投影するドキュメントのプロパティを選択できます。アイテムのIDのみを返す場合は、以下のクエリを実行できます。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.id
FROM food
WHERE food.foodGroup = "Snacks" and food.id = "19015"
```

あまり一般的ではありませんが、引用符で囲まれたプロパティ演算子 [""] を使用してプロパティにアクセスすることもできます。たとえば、SELECT food.id と SELECT food["id"] は同等です。この構文は、スペースや特殊文字を含むプロパティ、または SQL キーワードや予約語と同じ名前を持つプロパティをエスケープする場合に便利です。

```sql
SELECT food["id"]
FROM food
WHERE food["foodGroup"] = "Snacks" and food["id"] = "19015"
```

## WHERE句

WHERE 句を調べてみましょう。算術演算子、比較演算子、論理演算子などの複雑なスカラー式を WHERE 句に追加できます。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.id,
food.description,
food.tags,
food.foodGroup,
food.version
FROM food
WHERE (food.manufacturerName = "The Coca-Cola Company" AND food.version > 0)
```

このクエリは、manufacturerNameが"The Coca-Cola Company"でversionが0より大きいアイテムのid、description、tags、foodGroup、versionを返します。

検索結果の1件目は以下のようになります。

```json
{
  "id": "14026",
  "description": "Beverages, Energy Drink, sugar-free with guarana",
  "tags": [
    {
      "name": "beverages"
    },
    {
      "name": "energy drink"
    },
    {
      "name": "sugar-free with guarana"
    }
  ],
  "foodGroup": "Beverages",
  "manufacturerName": "The Coca-Cola Company",
  "version": 1
}
```

クエリがtagsの値としてJSON配列を返したことに気付いてください。

## プロジェクション

Azure Cosmos DB では、結果の JSON でいくつかの形式の変換がサポートされています。最も簡単な方法の 1 つは、結果を投影するときに``AS``エイリアシング キーワードを使用して JSON 要素にエイリアスを付けることです。

以下のクエリを実行すると、要素名が変換されていることがわかります。更にプロジェクションを利用して、WHERE 句で指定されたすべての項目の servings 配列の最初の要素にのみアクセスします。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.description,
food.foodGroup,
food.servings[0].description AS servingDescription,
food.servings[0].weightInGrams AS servingWeight
FROM food
WHERE food.foodGroup = "Fruits and Fruit Juices"
AND food.servings[0].description = "cup"
```

## ORDER BY句

Azure Cosmos DB では、1 つ以上のプロパティに基づいて結果を並べ替える ORDER BY 句の追加がサポートされています

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.description,
food.foodGroup,
food.servings[0].description AS servingDescription,
food.servings[0].weightInGrams AS servingWeight
FROM food
WHERE food.foodGroup = "Fruits and Fruit Juices" AND food.servings[0].description = "cup"
ORDER BY food.servings[0].weightInGrams DESC
```

ORDER BY句に必要なインデックスの構成の詳細については、後のインデックス作成ラボを参照するか[ドキュメント](
https://docs.microsoft.com/en-us/azure/cosmos-db/sql-query-order-by)を参照してください。

## 検索結果のサイズ制限

Azure Cosmos DB では、TOP キーワードがサポートされています。TOP を使用すると、クエリから返される値の数を制限できます。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT TOP 20 food.id,
food.description,
food.tags,
food.foodGroup
FROM food
WHERE food.foodGroup = "Snacks"
```

OFFSET LIMIT句は、クエリからいくつかの値をスキップして取得するオプションの句です。OFFSET countとLIMIT countは、OFFSET LIMIT句で必須です。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.id,
food.description,
food.tags,
food.foodGroup
FROM food
WHERE food.foodGroup = "Snacks"
ORDER BY food.id
OFFSET 10 LIMIT 10
```

OFFSET LIMIT を ORDER BY 句と組み合わせて使用すると、順序付けられた値に対してスキップして取得することによって結果セットが生成されます。ORDER BY 句を使用しない場合は、結果の順序は常に同じになります。

## さらに高度なフィルタリング

INキーワードと BETWEENキーワードをクエリに追加しましょう。INは、指定された値が特定のリスト内のいずれかの要素と一致するかどうかを確認するために使用でき、BETWEENを使用して値の範囲に対してクエリを実行できます。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.id,
food.description,
food.tags,
food.foodGroup,
food.version
FROM food
WHERE food.foodGroup IN ("Poultry Products", "Sausages and Luncheon Meats")
    AND (food.id BETWEEN "05740" AND "07050")
```

## さらに高度なプロジェクション

Azure Cosmos DBでは、クエリ内でJSONプロジェクションがサポートされています。プロパティ名を変更した新しいJSONオブジェクトを投影してみましょう。

**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT {
"Company": food.manufacturerName,
"Brand": food.commonName,
"Serving Description": food.servings[0].description,
"Serving in Grams": food.servings[0].weightInGrams,
"Food Group": food.foodGroup
} AS Food
FROM food
WHERE food.id = "21421"
```

## ドキュメントを結合する

Azure Cosmos DBのJOINでは、ドキュメント内結合と自己結合がサポートされています。Azure Cosmos DBでは、ドキュメントまたはコンテナー間でのJOINはサポートされていません。

前のクエリの例では、food.servings配列の最初のservingsの属性のみを含む結果を返しました。JOINを使用することで、servings配列内のすべての項目で指定した属性を結果に含めることができます。

以下のクエリを実行して、foodドキュメントのservingsを反復処理します。
**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT
food.id as FoodID,
serving.description as ServingDescription
FROM food
JOIN serving IN food.servings
WHERE food.id = "03226"
```

JOINは、配列内のプロパティをフィルター処理する必要がある場合に便利です。ドキュメント内JOINの後にフィルターがある次の例を実行します。

```sql
SELECT VALUE COUNT(1)
FROM c
JOIN t IN c.tags
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND s.amount > 1
```

## システム関数

Azure Cosmos DB では、一般的な操作のための多数の組み込み関数がサポートされています。これらは、ABS、FLOOR、ROUNDなどの数学関数と、IS_ARRAY、IS_BOOL、IS_DEFINEDなどの型チェック関数をカバーしています。サポートされているシステム機能の詳細については、[こちら](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-query-system-functions)をご覧ください。

以下のクエリを実行して、いくつかのシステム関数の使用例を確認してください。
**New SQL Query**で新しいタブを作成し、以下のクエリを貼り付けて**Execute Query**を選択します。

```sql
SELECT food.id,
food.commonName,
food.foodGroup,
ROUND(nutrient.nutritionValue) AS amount,
nutrient.units
FROM food JOIN nutrient IN food.nutrients
WHERE IS_DEFINED(food.commonName)
AND nutrient.description = "Water"
AND food.foodGroup IN ("Sausages and Luncheon Meats", "Legumes and Legume Products")
AND food.id > "42178"
```

## 相関サブクエリ

多くのシナリオでは、サブクエリが効果的です。相関サブクエリは、外部クエリの値を参照するクエリです。ここでは、最も有用な例のいくつかについて説明します。サブクエリの詳細については、[こちら](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-query-subquery)をご覧ください。

サブクエリには、複数値サブクエリとスカラー サブクエリの2種類があります。複数値のサブクエリは一連のドキュメントを返し、常に FROM句内で使用されます。スカラー サブクエリ式は、単一の値に評価されるサブクエリです。

### 複数値サブクエリ

サブクエリを使用して JOIN式を最適化できます。

自己結合を実行し、name、nutritionValue、およびamountにフィルターを適用する次のクエリについて考えてみます。サブクエリを使用して、次の式と結合する前に、結合された配列項目を除外できます。

```sql
SELECT VALUE COUNT(1)
FROM c
JOIN t IN c.tags
JOIN n IN c.nutrients
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND (n.nutritionValue > 0
AND n.nutritionValue < 10) AND s.amount > 1
```

3 つのサブクエリを使用してこのクエリを書き直し、要求ユニット (RU) の料金を最適化して削減できます。複数値のサブクエリが常に外部クエリの FROM 句に表示されることを確認します。

```sql
SELECT VALUE COUNT(1)
FROM c
JOIN (SELECT VALUE t FROM t IN c.tags WHERE t.name = 'infant formula')
JOIN (SELECT VALUE n FROM n IN c.nutrients WHERE n.nutritionValue > 0 AND n.nutritionValue < 10)
JOIN (SELECT VALUE s FROM s IN c.servings WHERE s.amount > 1)
```

### スカラーサブクエリ

スカラーサブクエリのユースケースの1つは、ARRAY_CONTAINSをEXISTSとして書き換えることです。

ARRAY_CONTAINSを使用する次のクエリについて考えてみます。

```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE ARRAY_CONTAINS(f.tags, {name: 'orange'})
```

同じ結果を持つが EXISTS を使用する次のクエリを実行します。

```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE EXISTS(SELECT VALUE t FROM t IN f.tags WHERE t.name = 'orange')
```

EXISTS を使用する主な利点は、ARRAY_CONTAINS許可される単純な等値フィルターだけでなく、EXISTS 関数に複雑なフィルターを含めることができることです。

次に例を示します。

```sql
SELECT VALUE c.description
FROM c
JOIN n IN c.nutrients
WHERE n.units= "mg" AND n.nutritionValue > 0
```

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。