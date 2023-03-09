# 継続トークンを使用した Azure Cosmos DB ストアド プロシージャの作成

このラボでは、Azure Cosmos DBインスタンス内で複数のストアドプロシージャを作成して実行します。トランザクションロールバックのためのエラーのスロー、JavaScriptコンソールを使用したログ記録、制限された実行環境内での継続モデルの実装など、JavaScriptストアドプロシージャに固有の機能について説明します。

> これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## 継続モデルを使用したストアド プロシージャの作成

次に、サーバー上の制限された実行制限よりも長く実行される可能性のあるストアドプロシージャを実装します。継続モデルを実装して、ストアドプロシージャが前の実行で時間切れになった後に "中断したところから再開" できるようにします。

### 一括アップロードと一括削除のストアドプロシージャを作成する

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。

1. **データエクスプローラー** セクションで**NutritionDatabase**データベースノードを展開して、さらに**FoodCollection**コンテナーノードを展開します。

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**bulkUpload**を入力します。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。

   ```js
   function bulkUpload(docs) {
     var container = getContext().getCollection();
     var containerLink = container.getSelfLink();
     var count = 0;
     if (!docs) throw new Error("The array is undefined or null.");
     var docsLength = docs.length;
     if (docsLength == 0) {
       getContext()
         .getResponse()
         .setBody(0);
       return;
     }
     tryCreate(docs[count], callback);
     function tryCreate(doc, callback) {
       var isAccepted = container.createDocument(containerLink, doc, callback);
       if (!isAccepted)
         getContext()
           .getResponse()
           .setBody(count);
     }
     function callback(err, doc, options) {
       if (err) throw err;
       count++;
       if (count >= docsLength) {
         getContext()
           .getResponse()
           .setBody(count);
       } else {
         tryCreate(docs[count], callback);
       }
     }
   }
   ```

   > このストアドプロシージャは、ドキュメントの配列を1つのバッチでアップロードします。バッチ全体が完了していない場合、ストアドプロシージャは応答本文をインポートされたドキュメントの数に設定します。クライアント側のコードは、すべてのドキュメントがインポートされるまで、このストアドプロシージャを複数回呼び出す必要があります。

   上記のストアド プロシージャのコピーに問題がある場合は、このストアド プロシージャの完全なソース コードが次の場所にあります： [bulk_upload.js](../solutions/05-authoring_stored_procedures/bulk_upload.js)

1. タブの上部にある **Save**ボタンを選択します。

1. **データエクスプローラー**セクションの上部にある**New Stored Procedure**ボタン (2 つの歯車アイコン) を選択します。

1. ストアドプロシージャタブで、**Stored Procedure Id**フィールドを見つけ、**bulkDelete**を入力します。

1. ストアドプロシージャエディターのテキスト領域の内容を次の JavaScript コードに置き換えます。


   ```js
   function bulkDelete(query) {
     var container = getContext().getCollection();
     var containerLink = container.getSelfLink();
     var response = getContext().getResponse();
     var responseBody = {
       deleted: 0,
       continuation: true
     };
     if (!query) throw new Error("The query is undefined or null.");
     tryQueryAndDelete();
     function tryQueryAndDelete(continuation) {
       var requestOptions = { continuation: continuation };
       var isAccepted = container.queryDocuments(
         containerLink,
         query,
         requestOptions,
         function(err, retrievedDocs, responseOptions) {
           if (err) throw err;
           if (retrievedDocs.length > 0) {
             tryDelete(retrievedDocs);
           } else if (responseOptions.continuation) {
             tryQueryAndDelete(responseOptions.continuation);
           } else {
             responseBody.continuation = false;
             response.setBody(responseBody);
           }
         }
       );
       if (!isAccepted) {
         response.setBody(responseBody);
       }
     }
     function tryDelete(documents) {
       if (documents.length > 0) {
         var isAccepted = container.deleteDocument(
           documents[0]._self,
           {},
           function(err, responseOptions) {
             if (err) throw err;
             responseBody.deleted++;
             documents.shift();
             tryDelete(documents);
           }
         );
         if (!isAccepted) {
           response.setBody(responseBody);
         }
       } else {
         tryQueryAndDelete();
       }
     }
   }
   ```

   > このストアドプロシージャは、特定のクエリに一致するすべてのドキュメントを反復処理し、ドキュメントを削除します。ストアドプロシージャがすべてのドキュメントを削除できない場合は、継続トークンが返されます。クライアント側のコードは、ストアド プロシージャが継続トークンを返さなくなるまで、継続トークンを渡してストアド プロシージャを繰り返し呼び出すことが想定されています。

   上記のストアド プロシージャのコピーに問題がある場合は、このストアド プロシージャの完全なソース コードを次に示します: [bulk_delete.js](../solutions/05-authoring_stored_procedures/bulk_delete.js)
   
1. タブの上部にある **Save**ボタンを選択します。

### .NET Core プロジェクトを作成する

1. ローカル コンピューターで、ドキュメント フォルダー内の CosmosLabs フォルダーを見つけ、.NET Core プロジェクトのコンテンツを格納するために使用される`Lab07`フォルダーを開きます。マイクロソフトハンズオンラボを通じてこのラボを完了する場合、CosmosLabs フォルダーは**C:\labs\CosmosLabs**のパスにあります。

1. **Lab07**フォルダーで、フォルダーを右クリックし、**Codeで開く**メニューオプションを選択します。

   > または、現在のディレクトリでターミナルを実行して`code .`コマンドを実行することもできます。

1. 表示される Visual Studio Code ウィンドウで、**Explorer**ペインを右クリックし、**Open in Terminal** メニューオプションを選択します。

   ![Open in Terminal](../media/open_in_terminal.jpg)

1. ターミナルペインで、次のコマンドを入力して実行します。

   ```sh
   dotnet restore
   ```

   > このコマンドは、プロジェクト内の依存関係として指定されたすべてのパッケージを復元します。

1. ターミナルペインで、次のコマンドを入力して実行します。

   ```sh
   dotnet build
   ```

   > このコマンドは、プロジェクトをビルドします。

1. **Explorer** ウィンドウで、プロジェクト フォルダーに`DataTypes.cs`ファイルがあることを確認します。

   > このファイルには、次の手順で使用するデータ クラスが含まれています。

1. **Explorer**ウィンドウで`Program.cs`をダブルクリックして、エディターでファイルを開きます。クラスの先頭は次のようになります。

   ```csharp
   private static readonly string _endpointUri = "";
   private static readonly string _primaryKey = "";
   private static CosmosClient _client = new CosmosClient(_endpointUri, _primaryKey);
   ```

1. 以前のラボでメモしていたAzure Cosmos DBの資格情報から、`_endpointUri`変数の値に、**URI**を、`_primaryKey`変数の値に**プライマリーキー**を入力してください。資格情報が不明な場合は、[こちら](00-account_setup.md)の手順を参照してください。

    - > 例として **uri** が `https://cosmosacct.documents.azure.com:443/` の場合、記述は以下のようになります

    ```csharp
    private static readonly string _endpointUri = "https://cosmosacct.documents.azure.com:443/";
    ```

    - 例として **プライマリキー** が ``elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==`` の場合、記述は以下のようになります。

    ```csharp
    private static readonly string _primaryKey = "elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==";
    ```

1. 開いているエディターの内容を全て保存します。

1. Visual Studio Code ウィンドウで、**Explorer**ペインを右クリックし、**Open in Terminal**メニューオプションを選択します。

1. 開いているターミナルペインで、次のコマンドを入力して実行します。

   ```sh
   dotnet build
   ```

   > このコマンドは、コンソール プロジェクトをビルドします。`await`オペレーターの不足に関連する警告が表示される可能性があります。今のところ、これらは無視してかまいません。

### .NET Core SDK からの一括アップロードストアドプロシージャの実行

1. **Explorer**ウィンドウで`Program.cs`をダブルクリックして、エディターでファイルを開きます。

1. `Program`クラス内の`BulkUpload()`メソッドを見つけます。

   ```csharp
   private static async Task BulkUpload(Container container)
   {
      List<Food> foods = new Bogus.Faker<Food>()
      .RuleFor(p => p.Id, f => (-1 - f.IndexGlobal).ToString())
      .RuleFor(p => p.Description, f => f.Commerce.ProductName())
      .RuleFor(p => p.ManufacturerName, f => f.Company.CompanyName())
      .RuleFor(p => p.FoodGroup, f => "Energy Bars")
      .Generate(10000);
   }
   ```

   > Boguライブラリは一連のテストデータを生成します。この例では、Bogusライブラリとリストされているルールを使用して 10,000個のアイテムを作成しています。**Generate**メソッドは、ルールを使用して指定された数のエンティティを作成し、それらを汎用**List<T>**に格納するようにBogusライブラリに指示します。

1. `.Generate(10000)`の下に次のコード行を追加して、既定値が0の**pointer**という名前の変数を作成します。

   ```csharp
   int pointer = 0;
   ```

   > この変数を使用して、ストアドプロシージャによってアップロードされたドキュメントの数を決定します。

1. `pointer`フィールドの値**foods**コレクション内の項目の量より小さい限り、コードの反復処理を続行します。

   ```csharp
   while (pointer < foods.Count)
   {

   }
   ```

   > ポインタの値がオブジェクトセット内の食品オブジェクトの量以上になるまでドキュメントをアップロードし続けるwhileループを作成します。

1. `while`ブロック内に、ストアドプロシージャを実行するための次のコード行を追加します。

   ```csharp
   StoredProcedureExecuteResponse<int> result = await container.Scripts.ExecuteStoredProcedureAsync<int>("bulkUpload", new PartitionKey("Energy Bars"), new dynamic[] {foods.Skip(pointer)});
   ```

   > このコード行は、3つのパラメーターを使用してストアドプロシージャを実行します。それぞれのパラメータは、実行対象のデータセットのパーティションキー、ストアドプロシージャの名前、およびストアドプロシージャに送信する**food**オブジェクトのリストです。

1. `while`ブロック内に、次のコード行を追加して、ストアドプロシージャによって返される数値を`pointer`変数に格納します。

   ```csharp
   pointer += result.Resource;
   ```

   > ストアド プロシージャが処理されたドキュメントの数を返すたびに、カウンターをインクリメントします。

1. `while`ブロック内に、次のコード行を追加して、現在のイテレーションでアップロードされたドキュメントの量を出力します。

   ```csharp
   await Console.Out.WriteLineAsync($"{pointer} Total Items\t{result.Resource} Items Uploaded in this Iteration");
   ```

1. `Main()`メソッドを見つけて、次のコード行を追加します。

   ```csharp
   await BulkUpload(container);
   ```

1. `Main()`メソッドと`BulkUpload()`メソッドは次のようになります。

   ```csharp
   public static async Task Main(string[] args)
   {
      Database database = _client.GetDatabase(_databaseId);
      Container container = database.GetContainer(_containerId);

      await BulkUpload();
   }

   Private static Task BulkUpload()
   {
      List<Food> foods = new Bogus.Faker<Food>()
      .RuleFor(p => p.Id, f => (-1 - f.IndexGlobal).ToString())
      .RuleFor(p => p.Description, f => f.Commerce.ProductName())
      .RuleFor(p => p.ManufacturerName, f => f.Company.CompanyName())
      .RuleFor(p => p.FoodGroup, f => "Energy Bars")
      .Generate(10000);

      int pointer = 0;
      while (pointer < foods.Count)
      {
            StoredProcedureExecuteResponse<int> result = await container.Scripts.ExecuteStoredProcedureAsync<int>("bulkUpload", new PartitionKey("Energy Bars"), new dynamic[] {foods.Skip(pointer)});
            pointer += result.Resource;
            await Console.Out.WriteLineAsync($"{pointer} Total Items\t{result.Resource} Items Uploaded in this Iteration");
      }
   }
   ```

      - LINQライブラリの**Skip**メソッドを使用する C#コードでは、まだアップロードされていないドキュメントのサブセットのみを送信していることがわかります。
      - while ループの最初の実行時に、**0**個のドキュメントをスキップし、すべてのドキュメントのアップロードを試みます。
      - ストアドプロシージャの実行が完了すると、アップロードされたドキュメントの数を示す応答が返されます。例として、**5000**件のドキュメントがアップロードされたとします。
      - ポインタは値 **5000** にインクリメントされます。whileループの状態の次のチェックでは、**5000**が**10000**未満と評価され、whileループでコードが再び実行されます。
      - LINQ メソッドは、**5000**個のドキュメントをスキップし、残りの**5000**個のドキュメントをストアド プロシージャに送信してアップロードします。
      - このループは、すべてのドキュメントがアップロードされるまで続きます。

      > また、この記事の執筆時点では、Cosmos DBではすべての呼び出しに対して2MBの要求サイズ制限があることにも注意してください。データがこのテストデータよりも大きい場合は、要求ごとに小さいペイロードを送信するように検討してください。

> [!NOTE]
> このコードを実行する前に、Azure ポータルに戻り、フード コレクション コンテナーが 10,000 RU/秒にスケールアップされていることを確認してください。

1. 開いているターミナルペインで、次のコマンドを入力して実行します。

   ```sh
   dotnet run
   ```

   > このコマンドは、コンソールプロジェクトをビルドして実行します。

1. コンソールプロジェクトの実行結果を確認します。

   > このストアド プロシージャは、指定したパーティションキー内のコレクションに10,000個のドキュメントをバッチアップロードします。

### Azure ポータルでアップロードされたドキュメントを確認する

1. **Azure Portal** (<http://portal.azure.com>)に戻ります。

1. ポータルの左側で、**Resource groups**リンクを選択します。

1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

1. **cosmoslabs**ブレードで、作成されている**Azure Cosmos DB**を選択します。

1.  **Azure Cosmos DB**ブレードで、左側にある**データエクスプローラー**リンクを見つけて選択します。

1. 右側に表示された**データエクスプローラー**内で、**NutritionDatabase**データベースのノードを展開し、**FoodCollection**コンテナーのノードを展開します。

1. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. ドキュメントがアップロードされたことを検証するために、ストアドプロシージャの実行に以前に使用したパーティションキーを持つすべてのドキュメントを選択するクエリを発行します。クエリタブで、クエリエディターの内容を次のSQLクエリに置き換えます。

   ```sql
   SELECT * FROM foods f WHERE f.foodGroup = "Energy Bars"
   ```

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウで、クエリの結果を確認します。

1. クエリタブで、クエリエディターの内容を次の SQL クエリに置き換えます。

   ```sql
   SELECT COUNT(1) FROM foods f WHERE f.foodGroup = "Energy Bars"
   ```

   > このクエリは、**Energy Bars**パーティション キーにあるドキュメントの数を返します。

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウで、クエリの結果を確認します。

### .NET Core SDK からの一括削除ストアドプロシージャの実行

1. Visual Studio Codeで`Program.cs`をダブルクリックして、エディターでファイルを開きます。クラスの先頭は次のようになります。


   > The next stored procedure returns a complex JSON object instead of a simple typed value. We use a custom  C# class to deserialize the JSON object so we can use its data in our C# code.次のストアド プロシージャは、単純な型指定された値ではなく、複雑なJSONオブジェクトを返します。`DeleteStatus`クラスを使用しJSONオブジェクトを逆シリアル化し、そのデータをC#コードで使用できるようにします。

1. `BulkDelete()`メソッドを見つけて、次のコードを追加します。

   ```csharp
    private static async Task BulkDelete(Container container)
    {
        bool resume = true;
        do
        {
            string query = "SELECT * FROM foods f WHERE f.foodGroup = 'Energy Bars'";
            StoredProcedureExecuteResponse<DeleteStatus> result = await container.Scripts.ExecuteStoredProcedureAsync<DeleteStatus>("bulkDelete", new PartitionKey("Energy Bars"),  new dynamic[] {query});

            await Console.Out.WriteLineAsync($"Batch Delete Completed.\tDeleted: {result.Resource.Deleted}\tContinue: {result.Resource.Continuation}");
            resume = result.Resource.Continuation;
        }
        while (resume);
    }
   ```

   > このコードは、`resume`変数がtrueに設定されている限り、ドキュメントを削除するストアドプロシージャを実行します。ストアドプロシージャ自体は、ドキュメントの削除を続行する必要があるかどうかを示すブール値と、この実行の一部として削除されたドキュメントの数を示す数値を持つ、DeleteStatusとしてシリアル化されたオブジェクトを常に返します。do-whileループ内では、ストアドプロシージャから返されたブール値の値を`resume`変数に格納し、すべてのドキュメントが削除されたことを示す false値が返されるまでストアドプロシージャの実行を続けます。

1. `Main()`メソッドを見つけて`await BulkUpdate(container)`コメントアウトし、`BulkDelete()`メソッドを呼び出します。メソッドは次のようになります。

   ```csharp
    public static async Task Main(string[] args)
    {
        Database database = _client.GetDatabase(_databaseId);
        Container container = database.GetContainer(_containerId);

        //await BulkUpload(container);
        await BulkDelete(container);
    }
   ```

1. ターミナルペインで、次のコマンドを入力して実行します。

   ```sh
   dotnet run
   ```

   > このコマンドは、コンソールプロジェクトをビルドして実行します。

1. コンソールプロジェクトの実行結果を確認します。

   > このストアドプロシージャは、指定したパーティションキーに関連付けられているすべてのドキュメントを削除します。このデモでは、これは、以前にバッチアップロードしたドキュメントを削除することを意味します。

1. 開いているすべてのエディター タブを閉じます。

### Azure ポータルでのパーティションキー内のドキュメントのクエリ

1. **Azure Portal** (<http://portal.azure.com>)に戻ります。

1. ポータルの左側で、**Resource groups**リンクを選択します。

1. 1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

1. **cosmoslabs**ブレードで、作成されている**Azure Cosmos DB**を選択します。

1.  **Azure Cosmos DB**ブレードで、左側にある**データエクスプローラー**リンクを見つけて選択します。

1. 右側に表示された**データエクスプローラー**内で、**NutritionDatabase**データベースのノードを展開し、**FoodCollection**コンテナーのノードを展開します。

1. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. クエリタブで、クエリエディターの内容を次のSQLクエリに置き換えます。

   ```sql
   SELECT COUNT(1) FROM foods f WHERE f.foodGroup = "Energy Bars"
   ```

   > このクエリは、**Energy Bars**パーティション キーにあるドキュメントの数を返します。このカウントでは、すべてのドキュメントが削除されたことを確認する必要があります。

1. Sクエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウで、クエリの結果を確認します。

> [!NOTE]
>  このコンテナーを保持する場合は、400 RU/秒にスケールダウンすることを検討してください。

1. ブラウザアプリケーションを閉じます。

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。

