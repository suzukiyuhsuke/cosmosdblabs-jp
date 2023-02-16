# .NETコンソールアプリの作成

After using the Azure Portal's **Data Explorer** to query an Azure Cosmos DB container in Lab 3, you are now going to use the .NET SDK to issue similar queries.
ラボ3では、Azureポータル上で**データエクスプローラー**を使用してAzure Cosmos DBコンテナーに対してクエリを実行しました。本ラボでは、.NET SDKを利用して同様のクエリを発行します。

> これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## .NET Coreプロジェクトを作成する

1. ローカルマシン上で、`ドキュメント`内のCosmosLabsフォルダーを開きます。

2. `Lab05`フォルダーを開きます。

3. `Lab05`フォルダー内で、右クリックから**Open with Code**を選んでVisual Studio Codeを開きます。

   ![Open with Visual Studio Code](../media/03-open_with_code.jpg)

   > 現在のディレクトリでターミナルを実行して、`code .`コマンドを入力することもできます。

4. 開いたVisual Studio Codeのウィンドウで、左側の**Explorer**ペインを右クリックし、**Open in Terminal**メニューを選択します。

   ![Open in Terminal](../media/open_in_terminal.jpg)

   > もしくはショートカット`Ctrl+Shift+@`でもターミナルを開くことができます。

5. ターミナルペインで、以下のコマンドを実行します。

   ```sh
   dotnet restore
   ```

   > このコマンドを実行すると、プロジェクト内の依存関係として指定されたすべてのパッケージを復元します。

6. ターミナルペインで、以下のコマンドを実行します。

   ```sh
   dotnet build
   ```

   > このコマンドを実行すると、プロジェクトがビルドされます。

7. **Explorer**ペインで、プロジェクト内に`DataTypes.cs`ファイルが存在することを確認します。

   > このファイルには、次の手順で使用するで＾多クラスが含まれています。

8. **Explorer**ペインで`Program.cs`ファイルを選択して、エディターで開きます。

   ![Visual Studio Code editor is displayed with the program.cs file highlighted](../media/03-program_editor.jpg "Open the program.cs file")

9. 以前のラボでメモしていたAzure Cosmos DBの資格情報から、`_endpointUri`変数の値に、**URI**を、`_primaryKey`変数の値に**プライマリーキー**を入力してください。資格情報が不明な場合は、[こちら](00-account_setup.md)の手順を参照してください。

    - > 例として **uri** が `https://cosmosacct.documents.azure.com:443/` の場合、記述は以下のようになります

    ```csharp
    private static readonly string _endpointUri = "https://cosmosacct.documents.azure.com:443/";
    ```

    - 例として **プライマリキー** が ``elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==`` の場合、記述は以下のようになります。

    ```csharp
    private static readonly string _primaryKey = "elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==";
    ```

## ReadItemAsyncを使用してAzure Cosmos DBで単一のドキュメントを読み取る

ReadItemAsyncを使用すると、そのIDによって1つの項目をCosmos DBから取得できます。Azure Cosmos DBでは、これが1つのドキュメントを読み取る最も効率的な方法です。

1. `Main()`メソッド内で以下のコードを見つけます。

    ```csharp
    Database database = _client.GetDatabase(_databaseId);
    Container container = database.GetContainer(_containerId);
    ```

1. 次のコードを追加して、`ReadItemAsync()`関数を使用してCosmos DBから`id`に該当する項目を取得し、その`Description`プロパティをコンソールに出力します。

    ```csharp
    ItemResponse<Food> candyResponse = await container.ReadItemAsync<Food>("19130", new PartitionKey("Sweets"));
    Food candy = candyResponse.Resource;
    Console.Out.WriteLine($"Read {candy.Description}");
    ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

   ```sh
   dotnet run
   ```

1. コンソールに次の行出力が表示され、`ReadItemAsync()`が正常に完了したことが示されます。 

   ```sh
   Read Candies, HERSHEY''S POT OF GOLD Almond Bar
   ```

## 単一の Azure Cosmos DB パーティションに対してクエリを実行する

1. エディタ上の`program.cs`ファイルに戻ります。

1. コードの最終行を見つけます。以下の記述です。

    ```csharp
    Console.Out.WriteLine($"Read {candy.Description}");
    ```

1. 次のように、データに対するクエリを作成します。

    ```csharp
    string sqlA = "SELECT f.description, f.manufacturerName, f.servings FROM foods f WHERE f.foodGroup = 'Sweets' and IS_DEFINED(f.description) and IS_DEFINED(f.manufacturerName) and IS_DEFINED(f.servings)";
    ```

    > このクエリは、foodGroupが`Sweets`に設定されているすべてのデータを選択します。また、description、manufacturerName、およびservingsのプロパティが定義されているドキュメントのみが選択されます。この構文は、以前に SQL を使用したことがある場合は、非常に馴染みがあることに気付くでしょう。また、このクエリのWHERE句にはパーティションキーが含まれているため、このクエリは1つのパーティション内で実行できます。

1. 次のコードを追加して、このクエリを実行して結果を読み取ります。

   ```csharp
   FeedIterator<Food> queryA = container.GetItemQueryIterator<Food>(new QueryDefinition(sqlA), requestOptions: new QueryRequestOptions{MaxConcurrency = 1});
   foreach (Food food in await queryA.ReadNextAsync())
   {
       await Console.Out.WriteLineAsync($"{food.Description} by {food.ManufacturerName}");
       foreach (Serving serving in food.Servings)
       {
           await Console.Out.WriteLineAsync($"\t{serving.Amount} {serving.Description}");
       }
       await Console.Out.WriteLineAsync();
   }
   ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. このコードは、SQL クエリの各結果をループ処理し、次のようなメッセージをコンソールに出力します。

    ```sh
    ...

    Puddings, coconut cream, dry mix, instant by
        1 package (3.5 oz)
        1 portion, amount to make 1/2 cup

    ...
    ```

### 複数の Azure Cosmos DB パーティションに対してクエリを実行する

1. エディタ上の`program.cs`ファイルに戻ります。

2. F`foreach`ループの下に、次のようにデータに対する SQL クエリを作成します。

    ```csharp
    string sqlB = @"SELECT f.id, f.description, f.manufacturerName, f.servings FROM foods f WHERE IS_DEFINED(f.manufacturerName)";
    ```

3. `sqlB`の定義の後に次のコード行を追加して、次のアイテム クエリを作成します。

    ```csharp
    FeedIterator<Food> queryB = container.GetItemQueryIterator<Food>(sqlB, requestOptions: new QueryRequestOptions{MaxConcurrency = 5, MaxItemCount = 100});
    ```

    > 前のセクションと比較したこの呼び出しの違いに注意してください。**MaxConcurrency**は5に設定されます。MaxConcurrencyはコンテナーのパーティションへの同時ネットワーク接続の最大数を設定します。このプロパティを-1に設定すると、SDKによって並列処理の数を管理します。MaxConcurrencyが0に設定されている場合、コンテナーのパーティションへのネットワーク接続は1つです。**MaxItemCount**は、クエリの待機時間とクライアント側のメモリ使用率をトレードします。このオプションを省略するか、-1に設定すると、SDKが並列クエリの実行中にバッファリングされるアイテムの数を管理します。記述したクエリではMaxItemCountを100アイテムに制限しています。これにより、クエリに一致するアイテムが100個を超えると、ページングが発生します。

4. 次のコード行を追加して、while ループを使用してこのクエリの結果をページングします。

    ```csharp
    int pageCount = 0;
    while (queryB.HasMoreResults)
    {
        Console.Out.WriteLine($"---Page #{++pageCount:0000}---");
        foreach (var food in await queryB.ReadNextAsync())
        {
            Console.Out.WriteLine($"\t[{food.Id}]\t{food.Description,-20}\t{food.ManufacturerName,-40}");
        }
    }
    ```

5. 開いているエディターの内容を全て保存します。

6. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

7. いくつかの新しい結果が表示され、それぞれがページを示す行で区切られます。結果は複数のパーティションから取得されていることに注意してください。

    ```sql
        [19067] Candies, TWIZZLERS CHERRY BITES Hershey Food Corp.
    ---Page #0017---
        [14644] Beverages, , PEPSICO QUAKER, Gatorade G2, low calorie   Quaker Oats Company - The Gatorade Company,  a unit of Pepsi Co.
    ```

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。
