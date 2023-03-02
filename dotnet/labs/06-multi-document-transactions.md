# 複数ドキュメントトランザクション用の Azure Cosmos DB ストアドプロシージャの作成

このラボでは、Azure Cosmos DBインスタンス内で複数のストアドプロシージャを作成して実行します。トランザクションロールバックのためのエラーのスロー、JavaScriptコンソールを使用したログ記録、制限された実行環境内での継続モデルの実装（トランザクション管理）など、JavaScriptストアドプロシージャに固有の機能について説明します。

> これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## 単純なストアドプロシージャの作成

このラボでは、データベース トランザクションの一部として1つ以上の項目を追加するなど、一般的なサーバー側タスクを実装する単純なストアドプロシージャを作成することから始めます。

> **註** 次の手順でクエリまたはストアドプロシージャを編集できない場合は、**データエクスプローラー**ウィンドウから移動し、再度開きます。

### 単純なストアドプロシージャの作成

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。

1. **データエクスプローラー** セクションで**NutritionDatabase**データベースノードを展開して、さらに**FoodCollection**コンテナーノードを展開します。

1. **FoodCollection**ノードで、**Items**リンクを選択します。

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

    ![The New Stored Procedure menu item is highlighted](../media/06-new_storedprocedure.jpg "Create a new Stored Procedure")

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**greetCaller**を入力します。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。

    ```js
    function greetCaller(name) {
        var context = getContext();
        var response = context.getResponse();
        response.setBody("Hello " + name);
    }
    ```

    ![A new stored procedure called greetCaller is displayed](../media/06-new_greet_caller_sp.jpg "Create a new stored procedure")

    > この単純なストアドプロシージャは、`Hello`をプレフィックスとして入力パラメーター文字列をエコーします。

1. タブの上部にある **Save**ボタンを選択します。
> **註**: Saveボタンがアクティブになっていないばあいは、**Stored Procedure Id**フィールドにマウスカーソルを併せて、テキストボックス内で値の変更操作をしてみてください。（例：１文字消去して、再度入力するなど）

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - In the  section, use Type  and enter the value: .
    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`example`を入力します。

    - パラメーターフィールドが一覧に表示されない場合は、**Add New Param**ボタンを選択します。

    - パラメーターフィールドで、**Type**を**String**にし、Valueとして`Person`を入力します。

    - **Execute**ボタンを選択します。

        ![The stored procedure parameters are populated](../media/06-execute_sp.jpg "Execute the stored procedure")

1. タブの下部にある**Result**ウィンドウで、ストアドプロシージャの実行結果を確認します。

    > 出力は次のようになります。`"Hello Person"`

### ネストされたコールバックを持つストアドプロシージャの作成

ストアド プロシージャ内のすべての Azure Cosmos DB 操作は非同期であり、JavaScript 関数コールバックに依存します。**コールバック関数**は、別の JavaScript 関数のパラメーターとして使用される JavaScript関数です。Azure Cosmos DBのコンテキストでは、コールバック関数には2つのパラメーターがあり、1つは操作が失敗した場合のエラーオブジェクト用、もう1つは作成されたオブジェクト用です。

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**createDocument**を入力します。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。

    ```js
    function createDocument(doc) {
        var context = getContext();
        var container = context.getCollection();
        var accepted = container.createDocument(
            container.getSelfLink(),
            doc,
            function (err, newItem) {
                if (err) throw new Error('Error' + err.message);
                context.getResponse().setBody(newItem);
            }
        );
        if (!accepted) return;
    }
    ```

1. ストアド プロシージャのコードを確認します。JavaScript コールバック内では、ユーザーが例外を処理するか、エラーをスローできることに注意してください。コールバックが指定されておらず、エラーが発生した場合、Azure Cosmos DB ランタイムはエラーをスローします。このストアド プロシージャは、新しい項目を作成し、入れ子になったコールバック関数を使用して、項目を応答の本文として返します。

1. タブの上部にある **Save**ボタンを選択します。

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`My Recipes`を入力します。

    - パラメーターフィールドが一覧に表示されない場合は、**Add New Param**ボタンを選択します。

    - パラメーターフィールドで、**Type**を**String**にし、Valueとして以下のjsonを入力します。

        ```json
        { "foodGroup": "My Recipes", "description": "Cookies" }
        ```

    - **Execute**ボタンを選択します。

1. タブの下部にある**Result**ウィンドウで、ストアドプロシージャの実行結果を確認します。

    ![The item created from the stored procedure is displayed](../media/06-execute_sp_02.jpg "review the item created")

    > コンテナーに新しい項目が表示されます。Azure Cosmos DB では、項目に追加のフィールド（`id`、`_etag`）が割り当てられています。

5. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. クエリタブで、クエリエディターの内容を次の SQL クエリに置き換えます。

    ```sql
    SELECT * FROM foods WHERE foods.foodGroup = "My Recipes" AND foods.description = "Cookies"
    ```

    > このクエリは、作成したばかりのアイテムを取得します。

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウで、クエリの結果を確認します。

1. **Query**タブを閉じます。

### ログを出力するストアドプロシージャの作成

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**createDocumentWithLogging**を入力します。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。

    ```js
    function createDocumentWithLogging(doc) {
        console.log("procedural-start");
        var context = getContext();
        var container = context.getCollection();
        console.log("metadata-retrieved");
        var accepted = container.createDocument(
            container.getSelfLink(),
            doc,
            function (err, newDoc) {
                console.log("callback-started");
                if (err) throw new Error('Error' + err.message);
                context.getResponse().setBody(newDoc.id);
            }
        );
        console.log("async-doc-creation-started");
        if (!accepted) return;
        console.log("procedural-end");
    }
    ```

    > このストアドプロシージャは、コンソールに出力を書き込むためにブラウザーベースのJavaScriptで通常使用される**console.log**機能を使用します。Azure Cosmos DBのコンテキストでは、この機能を使用して、ストアドプロシージャの実行後に返される診断ログ情報をキャプチャできます。

1. タブの上部にある **Save**ボタンを選択します。

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`My Recipes`を入力します。

    - パラメーターフィールドが一覧に表示されない場合は、**Add New Param**ボタンを選択します。

    - パラメーターフィールドで、**Type**を**String**にし、Valueとして以下のjsonを入力します。

        ```json
        { "foodGroup": "My Recipes", "description": "Cookies" }
        ```

    - **Execute**ボタンを選択します。


1. タブの下部にある**Result**ウィンドウで、ストアドプロシージャの実行結果を確認します

    > コンテナー内の新しい項目の一意の ID が表示されます。

1. **Result**ウィンドウの`console.log`リンクを選択して、ストアドプロシージャ実行のログデータを表示します。

    > ストアドプロシージャのプロシージャ コンポーネントが最初に終了し、項目が作成されるとコールバック関数が実行されたことがわかります。これは、JavaScript コールバックの非同期性を理解するのに役立ちます。

### コールバック関数を使用したストアドプロシージャの作成

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**createDocumentWithFunction**を入力します。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。

    ```js
    function createDocumentWithFunction(document) {
        var context = getContext();
        var container = context.getCollection();
        if (!container.createDocument(container.getSelfLink(), document, itemCreated))
            return;
        function itemCreated(error, newItem) {
            if (error) throw new Error('Error' + error.message);
            context.getResponse().setBody(newItem);
        }
    }
    ```

    > これは以前に作成したものと同じストアドプロシージャですが、暗黙的なコールバック関数の代わりに名前付き関数をインラインで使用しています。

1. タブの上部にある **Save**ボタンを選択します。

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`Packaged Foods`を入力します。

    - パラメーターフィールドが一覧に表示されない場合は、**Add New Param**ボタンを選択します。

    - パラメーターフィールドで、**Type**を**String**にし、Valueとして以下のjsonを入力します。

        ```json
        { "foodGroup": "My Recipes" }
        ```

    - **Execute**ボタンを選択します。

1. タブの下部にある**Result**ウィンドウで、ストアド プロシージャの実行が失敗したことを確認します。

    > ストアドプロシージャは、特定のパーティションキーにバインドされます。この例では、**Packaged Foods**パーティションキーのコンテキスト内でストアドプロシージャを実行しようとしました。ストアドプロシージャ内で、**My Recipes**パーティションキーを使用して新しい項目を作成しようとしました。ストアドプロシージャは、ストアドプロシージャの実行時に指定されたパーティションキー以外のパーティションキーに新しい項目を作成 (または既存の項目にアクセス) できませんでした。これにより、ストアドプロシージャが失敗しました。ストアドプロシージャ内のパーティションキー間で項目を作成または操作することはできません。

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`Packaged Foods`を入力します。

    - パラメーターフィールドが一覧に表示されない場合は、**Add New Param**ボタンを選択します。

    - パラメーターフィールドで、**Type**を**String**にし、Valueとして以下のjsonを入力します。

        ```json
        { "foodGroup": "Packaged Foods" }
        ```

    4. **Execute**ボタンを選択します。

1. タブの下部にある**Result**ウィンドウで、ストアドプロシージャの実行結果を確認します


    > コンテナーに新しい項目が表示されます。Azure Cosmos DB では、項目に追加のフィールド（`id`、`_etag`）が割り当てられています。

1. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. クエリタブで、クエリエディターの内容を次の SQL クエリに置き換えます。

    ```sql
    SELECT * FROM foods WHERE foods.foodGroup = "Packaged Foods"
    ```

    > このクエリは、作成したばかりのアイテムを取得します。

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウで、クエリの結果を確認します。

1. **Query**タブを閉じます。

### エラー処理を使用したストアドプロシージャの作成

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**createTwoDocuments**を入力します。


1. ストアドプロシージャエディターのテキスト領域の内容を次のJavaScriptコードに置き換えます。

    ```js
    function createTwoDocuments(foodGroupName, foodDescription, mealName) {
        var context = getContext();
        var container = context.getCollection();
        var firstItem = {
            foodGroup: foodGroupName,
            description: foodDescription
        };
        var secondItem = {
            foodGroup: foodGroupName,
            eaten: {
                meal: mealName
            }
        };
        var firstAccepted = container.createDocument(container.getSelfLink(), firstItem,
            function (firstError, newFirstItem) {
                if (firstError) throw new Error('Error' + firstError.message);
                var secondAccepted = container.createDocument(container.getSelfLink(), secondItem,
                    function (secondError, newSecondItem) {
                        if (secondError) throw new Error('Error' + secondError.message);
                        context.getResponse().setBody({
                            foodRecord: newFirstItem,
                            mealRecord: newSecondItem
                        });
                    }
                );
                if (!secondAccepted) return;
            }
        );
        if (!firstAccepted) return;
    }
    ```

    > このストアドプロシージャは、入れ子になったコールバックを使用して2つの個別の項目を作成します。データが複数のJSONドキュメントに分割され、1つのストアドプロシージャで複数の項目を追加または変更する必要があるシナリオがある場合があります。

1. タブの上部にある **Save**ボタンを選択します。

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`Vitamins`を入力します。

    - **Add New Param**ボタンを 2 回選択します。

    - 表示される最初のフィールドに、`Vitamins`を入力します。

    - 表示される 2 番目のフィールドに、`Calcium`を入力します。

    - 表示される 3 番目のフィールドに、`Breakfast`を入力します。

    - **Execute**ボタンを選択します。

1. タブの下部にある**Result**ウィンドウで、ストアドプロシージャの実行結果を確認します。

    > コンテナーに新しい項目が表示されます。Azure Cosmos DB では、項目に追加のフィールド（`id`、`_etag`）が割り当てられています。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。

    ```js
    function createTwoDocuments(foodGroupName, foodDescription, mealName) {
        var context = getContext();
        var container = context.getCollection();
        var firstItem = {
            foodGroup: foodGroupName,
            description: foodDescription
        };
        var secondItem = {
            foodGroup: foodGroupName + "_meal",
            eaten: {
                meal: mealName
            }
        };
        var firstAccepted = container.createDocument(container.getSelfLink(), firstItem,
            function (firstError, newFirstItem) {
                if (firstError) throw new Error('Error' + firstError.message);
                console.log('Created: ' + newFirstItem.id);
                var secondAccepted = container.createDocument(container.getSelfLink(), secondItem,
                    function (secondError, newSecondItem) {
                        if (secondError) throw new Error('Error' + secondError.message);
                        console.log('Created: ' + newSecondItem.id);
                        context.getResponse().setBody({
                            foodRecord: newFirstItem,
                            mealRecord: newSecondItem
                        });
                    }
                );
                if (!secondAccepted) return;
            }
        );
        if (!firstAccepted) return;
    }
    ```

    - トランザクションは、Cosmos DBのJavaScriptプログラミングモデルに深くネイティブに統合されています。
    - JavaScript 関数内では、すべての操作が1つのトランザクションに自動的にラップされます。
    - JavaScriptが例外なく完了すると、データベースに対する操作がコミットされます。

ストアドプロシージャを変更して、2番目の項目に別のfoodGroup名を入力します。これにより、2番目の項目が別のパーティションキーを使用するため、ストアドプロシージャが失敗します。スクリプトから伝達された例外がある場合、Cosmos DBのJavaScriptランタイムはトランザクション全体をロールバックします。これにより、1番目または2 番目の項目がデータベースにコミットされないことが効果的に保証されます。

1. タブの上部にある **Update**ボタンを選択します。

1. タブの上部にある **Execute**ボタンを選択します。

1. 表示される **Input parameters**ポップアップで、次の操作を実行します。

    - **Partition key value**セクションで、**Type**を**String**にし、Valueとして`Junk Food`を入力します。

    - **Add New Param**ボタンを 2 回選択します。

    - 表示される最初のフィールドに、`Junk Food`を入力します。

    - 表示される 2 番目のフィールドに、`Chips`を入力します。

    - 表示される 3 番目のフィールドに、`Midnight Snack`を入力します。

    - **Execute**ボタンを選択します。

1. タブの下部にある**Result**ウィンドウで、ストアド プロシージャの実行が失敗したことを確認します。

    > このストアドプロシージャは 2 番目の項目の作成に失敗したため、トランザクション全体がロールバックされました。

1. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. クエリタブで、クエリエディターの内容を次の SQL クエリに置き換えます。

    ```sql
    SELECT * FROM foods WHERE foods.foodGroup = "Junk Food"
    ```

    > このクエリでは、トランザクションがロールバックされてからアイテムは取得されません。

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。空の配列のみが表示されます。

1. **Results**ウィンドウで、クエリの結果を確認します。

1. **Query**タブを閉じます。

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。
