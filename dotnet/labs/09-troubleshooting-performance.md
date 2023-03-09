# Cosmos DBでのパフォーマンスのトラブルシューティング

このラボでは、.NET SDK を使用して Azure Cosmos DBへのリクエストをチューニングし、アプリケーションのパフォーマンスとコストを最適化します。


>  これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## .NET Coreプロジェクトを作成する

1. ローカル コンピューターで、ドキュメントフォルダー内の CosmosLabsフォルダーを見つけ、.NET Coreプロジェクトのコンテンツを格納するために使用される`Lab09`フォルダーを開きます。

1. `Lab09`フォルダで、フォルダを右クリックし、**Codeで開く**メニューオプションを選択します。

    > または、現在のディレクトリでターミナルを実行して``code .``コマンドを実行することもできます。

1. 表示される Visual Studio Code ウィンドウで、**エクスプローラー**ペインを右クリックし、**ターミナルで開く**メニューオプションを選択します。
1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet restore
    ```

    > このコマンドは、プロジェクト内の依存関係として指定されたすべてのパッケージを復元します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet build
    ```

    > このコマンドはプロジェクトをビルドします。

1. **エクスプローラー**ペインで、**DataTypes.cs**を選択します。
1. ファイルを確認し、次の手順で使用するデータ クラスが含まれていることを確認します。

1. **エクスプローラー**ペインで、**Program.cs**を選択します。

1. 以前のラボでメモしていたAzure Cosmos DBの資格情報から、`_endpointUri`変数の値に、**URI**を、`_primaryKey`変数の値に**プライマリーキー**を入力してください。資格情報が不明な場合は、[こちら](00-account_setup.md)の手順を参照してください。

    - 例として **uri** が `https://cosmosacct.documents.azure.com:443/` の場合、記述は以下のようになります

    ```csharp
    private static readonly string _endpointUri = "https://cosmosacct.documents.azure.com:443/";
    ```

    - 例として **プライマリキー** が ``elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==`` の場合、記述は以下のようになります。

    ```csharp
    private static readonly string _primaryKey = "elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==";
    ```

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet build
    ```

## レスポンスヘッダーの確認

Azure Cosmos Dは、リクエストとサーバー側で発生した操作に関するより多くのメタデータを提供できるさまざまなレスポンスヘッダーを返します。.NET SDKは、これらのヘッダーの多くを`ResourceResponse<>`クラスのプロパティとして公開します。

### 大きなアイテムのRUを確認する

1. `Main`メソッド内で次のコードを見つけます。

    ```csharp

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

    ```

1. コードの最後の行の後に、次の手順で作成する新しい関数を呼び出すための新しいコード行を追加します。

    ```csharp
    await CreateMember(peopleContainer);
    ```

1. 次に、新しいオブジェクトを作成し、 `member`という名前の変数に格納する新しい関数を作成します。

    ```csharp
    private static async Task<double> CreateMember(Container peopleContainer)
    {
        object member = new Member { accountHolder = new Bogus.Person() };

    }
    ```

    > **Bogus**ライブラリには、ランダムなプロパティを持つ架空の人物を生成する特別なヘルパークラス(`Bogus.Person`)があります。架空の人物のJSONドキュメントの例を次に示します。

    ```js
    {
        "Gender": 1,
        "FirstName": "Rosalie",
        "LastName": "Dach",
        "FullName": "Rosalie Dach",
        "UserName": "Rosalie_Dach",
        "Avatar": "https://s3.amazonaws.com/uifaces/faces/twitter/mastermindesign/128.jpg",
        "Email": "Rosalie27@gmail.com",
        "DateOfBirth": "1962-02-22T21:48:51.9514906-05:00",
        "Address": {
            "Street": "79569 Wilton Trail",
            "Suite": "Suite 183",
            "City": "Caramouth",
            "ZipCode": "85941-7829",
            "Geo": {
                "Lat": -62.1607,
                "Lng": -123.9278
            }
        },
        "Phone": "303.318.0433 x5168",
        "Website": "gerhard.com",
        "Company": {
            "Name": "Mertz - Gibson",
            "CatchPhrase": "Focused even-keeled policy",
            "Bs": "architect mission-critical markets"
        }
    }
    ```

1. **member**変数をパラメーターとして使用して、**Container**インスタンスの**CreateItemAsync**メソッドを呼び出す新しいコード行を追加します。

    ```csharp
    ItemResponse<object> response = await peopleContainer.CreateItemAsync(member);
    ```

1. ブロックの最後のコード行の後に、新しいコード行を追加して、**ItemResponse<>** インスタンスの **RequestCharge**プロパティの値を出力し、**RequestCharge**を返します (この値は後の演習で使用します)。

    ```csharp
    await Console.Out.WriteLineAsync($"{response.RequestCharge} RU/s");
    return response.RequestCharge;
    ```

1. `Main`メソッドと`CreateMember`メソッドは次のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        await CreateMember(peopleContainer);
    }

    private static async Task<double> CreateMember(Container peopleContainer)
    {
        object member = new Member { accountHolder = new Bogus.Person() };
        ItemResponse<object> response = await peopleContainer.CreateItemAsync(member);
        await Console.Out.WriteLineAsync($"{response.RequestCharge} RU/s");
        return response.RequestCharge;
    }
    ```

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールプロジェクトの結果を確認します。ドキュメント作成操作で約`15 RU/s`を使用しています。

1. **Azure Portal** (<http://portal.azure.com>)に戻ります。

1. ポータルの左側で、**Resource groups**リンクを選択します。

1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

1. **cosmoslabs**ブレードで、作成されている**Azure Cosmos DB**を選択します。

1.  **Azure Cosmos DB**ブレードで、左側にある**データエクスプローラー**リンクを見つけて選択します。

1. 右側に表示された**データエクスプローラー**内で、**FinancialDatabase**データベースのノードを展開し、**PeopleCollection**コンテナーのノードを展開します。

1. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. クエリエディターの内容を次のSQLクエリに置き換えます。

    ```sql
    SELECT * FROM c
    ```

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウで、クエリの結果を確認します。**Query Stats** をクリックすると、~3 RU/秒の実行コストが表示されます。

1. .NET Corプロジェクトを含む現在開いている**Visual Studio Code**エディターに戻ります。

1. Visual Studio Codeで、**Program.cs**ファイルを選択して、ファイルのエディタータブを開きます。

1. 非常に大きなドキュメントの挿入に対する RU 料金を表示するには、**Bogus**ライブラリを使用して、Memberオブジェクトに架空のファミリを作成します。架空の家族を作成するために、配偶者と4人の架空の子供の配列を生成します。

    ```js
    {
        "accountHolder":  { ... },
        "relatives": {
            "spouse": { ... },
            "children": [
                { ... },
                { ... },
                { ... },
                { ... }
            ]
        }
    }
    ```

    各プロパティには、**Bogus**が生成した架空の人物がいます。これにより、RUを監視するために使用できる大きなJSONドキュメントが作成されます。

1. **Program.cs**エディタータブで、`CreateMember`メソッドを見つけます。

1. `CreateMember`メソッド内で、次のコード行を見つけます。

    ```csharp
    object member = new Member { accountHolder = new Bogus.Person() };
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    object member = new Member
    {
        accountHolder = new Bogus.Person(),
        relatives = new Family
        {
            spouse = new Bogus.Person(),
            children = Enumerable.Range(0, 4).Select(r => new Bogus.Person())
        }
    };
    ```

    > この新しいコードブロックは、上記で説明した大きなJSONオブジェクトを作成します。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールプロジェクトの結果を確認します。この新しい操作には、~50 RU/sが必要なことがわかります。単純なJSON キュメントよりもはるかに多くのRUが消費されました。 (最後の操作は ~15RU/秒でした)。

1. **データエクスプローラー**内で、**FinancialDatabase**データベースのノードを展開し、**PeopleCollection**コンテナーのノードを展開します。

1. Itemsの右側のオプションメニューから、**New SQL Query**を選択します。

1. クエリエディターの内容を次のSQLクエリに置き換えます。このクエリは、**Children**という名前のプロパティを持つコンテナー内の唯一の項目を返します

    ```sql
    SELECT * FROM c WHERE IS_DEFINED(c.relatives)
    ```

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

1. **Results**ウィンドウでクエリの結果を確認すると、返されるデータが多くなり、RUがわずかに高くなります。

### インデックスポリシーの調整

1. **データエクスプローラー**内で、**FinancialDatabase**データベースのノード、**PeopleCollection**コンテナーのノードの順に展開し、**Scale & Settings**オプションを選択します。

1. **Settings**セクションで、**Indexing Policy**フィールドを見つけて、現在の既定のインデックス作成ポリシーを確認します。

    ```js
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
                    "LineString",
                    "Polygon",
                    "MultiPolygon"
                ]
            }
        ]
    }
    ```

    > このポリシーは、クエリで使用されない_etagを除く、JSON ドキュメント内のすべてのパスにインデックスを付けます。このポリシーは、空間データのインデックスも作成します。

1. インデックス作成ポリシーを新しいポリシーに置き換えます。この新しいポリシーは、インデックス作成からパス`/relatives/*`を除外し、大きJSONドキュメントの**Children**プロパティをインデックスから効果的に削除します。

    ```js
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
            },
            {
                "path":"/relatives/*"
            }
        ],
        "spatialIndexes": [
            {
                "path": "/*",
                "types": [
                    "Point",
                    "LineString",
                    "Polygon",
                    "MultiPolygon"
                ]
            }
        ]
    }
    ```

1. セクションの上部にある**Save**ボタンを選択して、新しいインデックス作成ポリシーを保持します。

1. **データ エクスプローラー**セクションの上部にある**New SQL Query**ボタンを選択します。

1. クエリエディターの内容を次の SQL クエリに置き換えます。

    ```sql
    SELECT * FROM c WHERE IS_DEFINED(c.relatives)
    ```

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

    > **/relatives**パスが定義されているかどうかを判断できることがすぐにわかります。

1. クエリエディターの内容を次の SQLクエリに置き換えます。

    ```sql
    SELECT * FROM c WHERE IS_DEFINED(c.relatives) ORDER BY c.relatives.Spouse.FirstName
    ```

1. クエリタブの**Execute Query**ボタンを選択して、クエリを実行します。

    > このプロパティにはインデックスが作成されないため、このクエリはすぐに失敗します。インデックスを定義するときは、クエリ条件で使用できるのはインデックス付きプロパティのみであることに注意してください。

1. .NET Corプロジェクトを含む現在開いている**Visual Studio Code**エディターに戻ります。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソール プロジェクトの結果を確認します。この項目の作成に必要な RU/s (~26 RU/sと以前の ~48 RU/s) に違いがあることがわかります。これは、除外したパスをインデクサーがスキップしたことが原因です。

## リクエストのトラブルシューティング

まず、.NET SDK を使用して、コンテナーに割り当てられた容量を超えリクエストを発行します。要求ユニットの消費量は、秒単位のレートで評価されます。プロビジョニングされた要求ユニットレートを超えるアプリケーションの場合、レートがプロビジョニングされたスループットレベルを下回るまで、要求はレート制限されます。リクエストがレート制限されている場合、サーバーはHTTPステータスコード`429 RequestRateTooLargeException`でリクエストを終了し、`x-ms-retry-after-ms`ヘッダーを返します。ヘッダーは、クライアントが要求を再試行する前に待機する必要がある時間をミリ秒単位で示します。サンプルアプリケーションでリクエストのレート制限を確認します。

### コンテナのR/Uスループットの削減

1. **データエクスプローラー**セクションで、**FinancialDatabase**データベースノード、**TransactionCollection**ノードの順に展開し、**Scale & Settings**オプションを選択します。

1. **Settings**セクションで、**Throughput**フィールドを見つけ、その値を**400**に更新します。

    > これは、コンテナーに割り当てることができる最小スループットです。

1. セクションの上部にある**Save**ボタンを選択して、新しいスループット割り当てを保持します。

### スロットリングの監視 (HTTP 429)

1. .NET Corプロジェクトを含む現在開いている**Visual Studio Code**エディターに戻ります。

1. Visual Studio Codeで、**Program.cs**ファイルを選択して、ファイルのエディタータブを開きます。

1. `Main`メソッド内の`await CreateMember(peopleContainer)`行を見つけます。この行をコメントアウトし、次のように新しい行を追加します。

    ```csharp
    public static async Task Main(string[] args)
    {

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        //await CreateMember(peopleContainer);
        await CreateTransactions(transactionContainer);
    }
    ```

1. `CreateMember`メソッドの下に新しい`CreateTransactions`メソッドを作成します。

    ```csharp
    private static async Task CreateTransactions(Container transactionContainer)
    {

    }
    ```

1. 次のコードを追加して、`Transaction`インスタンスのコレクションを作成します。

    ```csharp
    var transactions = new Bogus.Faker<Transaction>()
        .RuleFor(t => t.id, (fake) => Guid.NewGuid().ToString())
        .RuleFor(t => t.amount, (fake) => Math.Round(fake.Random.Double(5, 500), 2))
        .RuleFor(t => t.processed, (fake) => fake.Random.Bool(0.6f))
        .RuleFor(t => t.paidBy, (fake) => $"{fake.Name.FirstName().ToLower()}.{fake.Name.LastName().ToLower()}")
        .RuleFor(t => t.costCenter, (fake) => fake.Commerce.Department(1).ToLower())
        .GenerateLazy(100);
    ```

1. 次の foreach ブロックを追加して、`Transaction`インスタンスを反復処理します。

    ```csharp
    foreach(var transaction in transactions)
    {

    }
    ```

1. `foreach`ブロック内に次のコード行を追加して、項目を非同期的に作成し、作成タスクの結果を変数に保存します。

    ```csharp
    ItemResponse<Transaction> result = await transactionContainer.CreateItemAsync(transaction);
    ```

    > `Container`クラスの`CreateItemAsync`メソッドは、JSOにシリアル化し、指定されたコレクション内の項目として格納するオブジェクトを受け取ります。

1. `foreach`ブロック内に、次のコード行を追加して、新しく作成されたリソースの`id`プロパティの値をコンソールに書き込みます。

    ```csharp
    await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
    ```

    > この`ItemResponse`型には、操作の結果としての項目インスタンスへのアクセスを可能にする名前付きプロパティ`Resource`があります。

1. `CreateTransactions`メソッドは次のようになります。

    ```csharp
    private static async Task CreateTransactions(Container transactionContainer)
    {
        var transactions = new Bogus.Faker<Transaction>()
            .RuleFor(t => t.id, (fake) => Guid.NewGuid().ToString())
            .RuleFor(t => t.amount, (fake) => Math.Round(fake.Random.Double(5, 500), 2))
            .RuleFor(t => t.processed, (fake) => fake.Random.Bool(0.6f))
            .RuleFor(t => t.paidBy, (fake) => $"{fake.Name.FirstName().ToLower()}.{fake.Name.LastName().ToLower()}")
            .RuleFor(t => t.costCenter, (fake) => fake.Commerce.Department(1).ToLower())
            .GenerateLazy(100);

        foreach(var transaction in transactions)
        {
            ItemResponse<Transaction> result = await transactionContainer.CreateItemAsync(transaction);
            await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
        }
    }
    ```

    > Bogusライブラリは一連のテストデータを生成します。この例では、偽のライブラリと上記のルールを使用して 100個のアイテムを作成しています。**GenerateLazy**メソッドは、`IEnumerable<Transaction>`型の変数を返すことによって 100項目の要求を準備するように Bogus ライブラリに指示します。LINQは既定で遅延実行を使用するため、コレクションが反復されるまで項目は実際には作成されません。このコードブロックの末尾にある**foreach**ループは、コレクションを反復処理し、Azure Cosmos DBに項目を作成します。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。このツールによって作成される新しいアイテムに関連付けられているアイテIDのリストが表示されます。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    foreach (var transaction in transactions)
    {
        ItemResponse<Transaction> result = await transactionContainer.CreateItemAsync(transaction);
        await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
    }
    ```

    これらのコード行を次のコードに置き換えます。

    ```csharp
    List<Task<ItemResponse<Transaction>>> tasks = new List<Task<ItemResponse<Transaction>>>();
    foreach (var transaction in transactions)
    {
        Task<ItemResponse<Transaction>> resultTask = transactionContainer.CreateItemAsync(transaction);
        tasks.Add(resultTask);
    }
    Task.WaitAll(tasks.ToArray());
    foreach (var task in tasks)
    {
        await Console.Out.WriteLineAsync($"Item Created\t{task.Result.Resource.id}");
    }
    ```

    > これらの作成タスクをできるだけ多く並行して実行しようとします。コンテナーは最小400 RU/秒で構成されていることに注意してください。

1. `CreateTransactions`メソッドは次のようになります。

    ```csharp
    private static async Task CreateTransactions(Container transactionContainer)
    {
        var transactions = new Bogus.Faker<Transaction>()
            .RuleFor(t => t.id, (fake) => Guid.NewGuid().ToString())
            .RuleFor(t => t.amount, (fake) => Math.Round(fake.Random.Double(5, 500), 2))
            .RuleFor(t => t.processed, (fake) => fake.Random.Bool(0.6f))
            .RuleFor(t => t.paidBy, (fake) => $"{fake.Name.FirstName().ToLower()}.{fake.Name.LastName().ToLower()}")
            .RuleFor(t => t.costCenter, (fake) => fake.Commerce.Department(1).ToLower())
            .GenerateLazy(100);

        List<Task<ItemResponse<Transaction>>> tasks = new List<Task<ItemResponse<Transaction>>>();

        foreach (var transaction in transactions)
        {
            Task<ItemResponse<Transaction>> resultTask = transactionContainer.CreateItemAsync(transaction);
            tasks.Add(resultTask);
        }

        Task.WaitAll(tasks.ToArray());

        foreach (var task in tasks)
        {
            await Console.Out.WriteLineAsync($"Item Created\t{task.Result.Resource.id}");
        }
    }
    ```

    - 最初の foreach ループは、作成されたトランザクションを反復処理し、`List`に格納された各非同期タスクは、Azure Cosmos DB に要求を発行します。これらの要求は並列で発行され、コンテナーには要求の量を処理するのに十分なスループットがプロビジョニングされていないため、`429 - too many requests`例外が発生する可能性があります。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > このクエリは正常に実行されます。作成しているアイテムは 100 個のみであり、ここではスループットの問題は発生しない可能性があります。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    .GenerateLazy(100);
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    .GenerateLazy(7000);
    ```

    > 7000個のアイテムを並行して作成して、スループットの制限に達することができるかどうかを確認します。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. しばらくするとアプリケーションがクラッシュすることを確認します。

    > このクエリは、スループットの制限に達する可能性があります。特定の要求が失敗したことを示す複数のエラーメッセージが表示されます。

### R/Uスループットを増やしてスロットリングを減らす

1.  **データ エクスプローラー**セクションで、**FinancialDatabase**データベースノード、**TransactionCollection**ノードの順に展開し、**Scale & Settings**オプションを選択します。

1. **Settings**セクションで、**Throughput**フィールドを見つけ、その値を**10000**に更新します

1. セクションの上部にある**Save**ボタンを選択して、新しいスループット割り当てを保持します。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. しばらくするとアプリケーションが完了します。

1. **Azure Portal**の**Settings**セクションに戻り、**Throughput**の値を**400**に戻します。

1. セクションの上部にある**Save**ボタンを選択して、新しいスループット割り当てを保持します。

## クエリと読み取りの調整

次に、.NET SDKの**RequestOptions**クラスのSQLクエリとプロパティを操作して、Azure Cosmos DBへのリクエストをチューニングします。

### RU使用量の測定

1. `Main`メソッドから`CreateTransactions`メソッド呼び出しをコメントアウトして、新しい行`await QueryTransactions(transactionContainer);`を追加します。`Main`メソッドは次のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {
        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        //await CreateMember(peopleContainer);
        //await CreateTransactions(transactionContainer);
        await QueryTransactions(transactionContainer);
    }
    ```

1. 新しい`QueryTransactions`メソッドを作成し、SQL クエリを文字列変数に格納する次のコード行を追加します。

    ```csharp
    private static async Task QueryTransactions(Container transactionContainer)
    {
        string sql = "SELECT TOP 1000 * FROM c WHERE c.processed = true ORDER BY c.amount DESC";

    }
    ```

    > このクエリは、パーティション間の ORDER BY を実行し、1000 項目のうち上位 50000 項目のみを返します。

1. 次のコード行を追加して、アイテムクエリインスタンスを作成します。

    ```csharp
    FeedIterator<Transaction> query = transactionContainer.GetItemQueryIterator<Transaction>(sql);
    ```

1. 次のコード行を追加して、結果の最初の"ページ"を取得します。

    ```csharp
    var result = await query.ReadNextAsync();
    ```

    > 完全な結果セットは列挙しません。結果の最初のページのメトリックにのみ関心があります。

1. 次のコード行を追加して、クエリの要求料金メトリックをコンソールに出力します。

    ```csharp
    await Console.Out.WriteLineAsync($"Request Charge: {result.RequestCharge} RU/s");
    ```

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソーアプリケーションの出力を確認します。**Request Charge**メトリックがコンソール ウィンドウに出力されます。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    string sql = "SELECT TOP 1000 * FROM c WHERE c.processed = true ORDER BY c.amount DESC";
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    string sql = "SELECT * FROM c WHERE c.processed = true";
    ```

    > この新しいクエリは、パーティション間のORDER BYを実行しません。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。**Request Charge**の両方の値が ~35 RU/sにわずかに減少します。

1. コーエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    string sql = "SELECT * FROM c WHERE c.processed = true";
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    string sql = "SELECT * FROM c";
    ```

    > この新しいクエリでは、結果セットはフィルター処理されません。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。このクエリは~21 RU/sで処理されます。最後のいくつかのクエリからのさまざまなメトリック値のわずかな違いを観察します。

1. コードエディタータブに戻り、次のコード行を見つけます。

     ```csharp
     string sql = "SELECT * FROM c";
     ```

    そのコード行を次のコードに置き換えます。

     ```csharp
     string sql = "SELECT c.id FROM c";
     ```

     > この新しいクエリは結果セットをフィルター処理せず、ドキュメントの一部のみを返します。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。これは約 ~22 RU/sです。返されるペイロードははるかに小さくなりますが、クエリエンジンが特定のプロパティを返すように機能したため、RU/sはわずかに高くなります。

### SDKクエリオプションの管理

1. `CreateTransactions`メソッドを見つけて、前のセクションで追加したコードを削除して、再び次のようになります。

    ```csharp
    private static async Task QueryTransactions(Container transactionContainer)
    {

    }
    ```

1. 次のコード行を追加して、クエリオプションを構成する変数を作成します。

    ```csharp
    int maxItemCount = 100;
    int maxDegreeOfParallelism = 1;
    int maxBufferedItemCount = 0;
    ```

1. 次のコード行を追加して、変数からのクエリのオプションを構成します。

    ```csharp
    QueryRequestOptions options = new QueryRequestOptions
    {
        MaxItemCount = maxItemCount,
        MaxBufferedItemCount = maxBufferedItemCount,
        MaxConcurrency = maxDegreeOfParallelism
    };
    ```

1. 次のコード行を追加して、コンソールウィンドウにさまざまな値を書き込みます。

    ```csharp
    await Console.Out.WriteLineAsync($"MaxItemCount:\t{maxItemCount}");
    await Console.Out.WriteLineAsync($"MaxDegreeOfParallelism:\t{maxDegreeOfParallelism}");
    await Console.Out.WriteLineAsync($"MaxBufferedItemCount:\t{maxBufferedItemCount}");
    ```

1. SQLクエリを文字列変数に格納する次のコード行を追加します。

    ```csharp
    string sql = "SELECT * FROM c WHERE c.processed = true ORDER BY c.amount DESC";
    ```

    > このクエリは、フィルター処理された結果セットに対してパーティション間のORDER BYを実行します。

1. 次のコード行を追加して、高精度タイマーを作成して新しいタイマーを開始します。

    ```csharp
    Stopwatch timer = Stopwatch.StartNew();
    ```

1. 次のコード行を追加して、アイテムクエリインスタンスを作成します。

    ```csharp
    FeedIterator<Transaction> query = transactionContainer.GetItemQueryIterator<Transaction>(sql, requestOptions: options);
    ```

1. 次のコード行を追加して、結果セットを列挙します。

    ```csharp
    while (query.HasMoreResults)  
    {
        var result = await query.ReadNextAsync();
    }
    ```

    > 結果はページングされるため、while ループで`ReadNextAsync`メソッドを複数回呼び出す必要があります。

1. 次のコード行を追加して、タイマーを停止します。

    ```csharp
    timer.Stop();
    ```

1. 次のコード行を追加して、タイマーの結果をコンソール ウィンドウに書き込みます。

    ```csharp
    await Console.Out.WriteLineAsync($"Elapsed Time:\t{timer.Elapsed.TotalSeconds}");
    ```

1. `QueryTransactions`メソッドは次のようになります。

    ```csharp
    private static async Task QueryTransactions(Container transactionContainer)
    {
        int maxItemCount = 100;
        int maxDegreeOfParallelism = 1;
        int maxBufferedItemCount = 0;

        QueryRequestOptions options = new QueryRequestOptions
        {
            MaxItemCount = maxItemCount,
            MaxBufferedItemCount = maxBufferedItemCount,
            MaxConcurrency = maxDegreeOfParallelism
        };

        await Console.Out.WriteLineAsync($"MaxItemCount:\t{maxItemCount}");
        await Console.Out.WriteLineAsync($"MaxDegreeOfParallelism:\t{maxDegreeOfParallelism}");
        await Console.Out.WriteLineAsync($"MaxBufferedItemCount:\t{maxBufferedItemCount}");

        string sql = "SELECT * FROM c WHERE c.processed = true ORDER BY c.amount DESC";

        Stopwatch timer = Stopwatch.StartNew();

        FeedIterator<Transaction> query = transactionContainer.GetItemQueryIterator<Transaction>(sql, requestOptions: options);
        while (query.HasMoreResults)  
        {
            var result = await query.ReadNextAsync();
        }
        timer.Stop();
        await Console.Out.WriteLineAsync($"Elapsed Time:\t{timer.Elapsed.TotalSeconds}");
    }
    ```

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > この最初のクエリには、予想外に長い時間がかかります。これには、SDKオプションを最適化する必要があります。

1. コード エディター タブに戻り、次のコード行を見つけます。

    ```csharp
    int maxDegreeOfParallelism = 1;
    ```

   そのコード行を次のように置き換えます。

    ```csharp
    int maxDegreeOfParallelism = 5;
    ```

    > `maxDegreeOfParallelism`クエリパラメーターの値を`1`に設定すると、並列処理が実質的に排除されます。ここでは、並列処理の値を`5`に"バンプアップ"します。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > 何らかの形の並列処理があることを考えると、ごくわずかなプラスの影響が見られるはずです。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    int maxBufferedItemCount = 0;
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    int maxBufferedItemCount = -1;
    ```

    > `MaxBufferedItemCount`プロパティの値を`-1`に設定すると、この設定を使用するようにSDKに効果的に指示されます。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。
    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > 繰り返しになりますが、これはパフォーマンス時間にわずかなプラスの影響を与えるはずです。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    int maxDegreeOfParallelism = 5;
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    int maxDegreeOfParallelism = -1;
    ```

    > 並列クエリは、複数のパーティションを並列にクエリすることで機能します。ただし、個々のパーティション分割されたコンテナーからのデータは、`maxBufferedItemCount`プロパティの値を`-1`に設定するクエリに関して、この設定を使用するように SDK に効果的に指示することで順次フェッチされます。**MaxDegreeOfParallelism**をパーティション数に設定すると、他のすべてのシステム条件が同じであれば、最もパフォーマンスの高いクエリを達成できる可能性があります。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > 繰り返しになりますが、これはパフォーマンス時間にわずかな影響を与えるはずです。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    int maxItemCount = 100;
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    int maxItemCount = 500;
    ```

    > クエリのパフォーマンスを向上させるために、"ページ"ごとに返されるアイテムの量を増やしています。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > クエリのパフォーマンスが大幅に向上したことがわかります。これは、クエリがクライアントコンピューターによってボトルネックになっていることを示している可能性があります。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    int maxItemCount = 500;
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    int maxItemCount = 1000;
    ```

    > 大きなクエリの場合は、ページサイズを1000の値まで増やすことをお勧めします。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。
    ```sh
    dotnet run
    ```

1. コンソール アプリケーションの出力を確認します。

    > ページ サイズを大きくすることで、クエリがさらに高速化されます。

1. コードエディタータブに戻り、次のコード行を見つけます。

    ```csharp
    int maxBufferedItemCount = -1;
    ```

    そのコード行を次のコードに置き換えます。

    ```csharp
    int maxBufferedItemCount = 50000;
    ```

    > 並列クエリは、結果の現在のバッチがクライアントによって処理されている間に結果をプリフェッチするように設計されています。プリフェッチは、クエリの全体的な待機時間の改善に役立ちます。**MaxBufferedItemCount**は、プリフェッチされる結果の数を制限するパラメーターです。**MaxBufferedItemCount**を返される結果の予想される数 (またはそれ以上の数) に設定すると、クエリはプリフェッチから最大のメリットを得ることができます。

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > ほとんどの場合、この変更によりクエリ時間がわずかに短縮されます。

### アイテムの読み取りとクエリ

1. **Azure Cosmos DB**ブレードで、左側にある**データエクスプローラー**リンクを見つけて選択します。

1. **データエクスプローラー**セクションで、**FinancialDatabase**データベースノード、**PeaopleCollection**ノードの順に展開し、**Items**オプションを選択します。

1. ドキュメントの**id**プロパティ値と、そのドキュメントの**partition key**をメモします。

    ![The PeopleCollection is displayed with an item selected.  The id and Lastname is highlighted.](../media/09-find-id-key.jpg "Find an item and record its id property")

1. Locate the `Main`メソッドを見つけて最後の行にコメントを付け、新しい行`await QueryMember(peopleContainer);`を追加して次のようにします。

    ```csharp
    public static async Task Main(string[] args)
    {

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        //await CreateMember(peopleContainer);
        //await CreateTransactions(transactionContainer);
        //await QueryTransactions(transactionContainer);
        await QueryMember(peopleContainer);
    }
    ```

1. `QueryTransactions`メソッドの下に、次のような新しいメソッドを作成します。

    ```csharp
    private static async Task QueryMember(Container peopleContainer)
    {

    }
    ```

1. SQLクエリを文字列変数に格納する次のコード行を追加します (**example.document**を前にメモした idの値に置き換えます)。

    ```csharp
    string sql = "SELECT TOP 1 * FROM c WHERE c.id = 'example.document'";
    ```

    > このクエリは、指定された一意のidに一致する1つのアイテムを検索します。

1. 次のコード行を追加して、アイテムクエリインスタンスを作成します。

    ```csharp
    FeedIterator<object> query = peopleContainer.GetItemQueryIterator<object>(sql);
    ```

1. 次のコード行を追加して結果の最初のページを取得し、**FeedResponse<>**型の変数に格納します。

    ```csharp
    FeedResponse<object> response = await query.ReadNextAsync();
    ```

    > ``TOP 1``からアイテムを取得するため、1つのページを取得するだけで済みます。

1. 次のコード行を追加して、**FeedResponse<>**インスタンスの **RequestCharge**プロパティの値を出力し、取得した項目の内容を出力します。

    ```csharp
    await Console.Out.WriteLineAsync($"{response.Resource.First()}");
    await Console.Out.WriteLineAsync($"{response.RequestCharge} RU/s");
    ```

1. メソッドは次のようになります。
    ```csharp
    private static async Task QueryMember(Container peopleContainer)
    {
        string sql = "SELECT TOP 1 * FROM c WHERE c.id = '372a9e8e-da22-4f7a-aff8-3a86f31b2934'";
        FeedIterator<object> query = peopleContainer.GetItemQueryIterator<object>(sql);
        FeedResponse<object> response = await query.ReadNextAsync();

        await Console.Out.WriteLineAsync($"{response.Resource.First()}");
        await Console.Out.WriteLineAsync($"{response.RequestCharge} RU/s");
    }
    ```

1. 開いているすべてのエディター タブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > 項目のクエリに使用された ~ 3 RU/sの量が表示されます。**LastName**オブジェクトのプロパティ値は、次の手順で使用するのでメモしておいてください。

1. Locate the `Main`メソッドを見つけて最後の行にコメントを付け、新しい行`await ReadMember(peopleContainer);`を追加して次のようにします。

    ```csharp
    public static async Task Main(string[] args)
    {

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        //await CreateMember(peopleContainer);
        //await CreateTransactions(transactionContainer);
        //await QueryTransactions(transactionContainer);
        //await QueryMember(peopleContainer);
        await ReadMember(peopleContainer);
    }
    ```

1. `QueryMember`メソッドの下に、次のような新しいメソッドを作成します。

    ```csharp
    private static async Task<double> ReadMember(Container peopleContainer)
    {

    }
    ```

1. 次のコードを追加して、`Container`クラスの`ReadItemAsync`メソッドを使用し、前の手順の一意のidとパーティションキーをそれぞれ`example.document`、`<Last Name>`と置き換えてアイテムを読み取ります。

    ```csharp
    ItemResponse<object> response = await peopleContainer.ReadItemAsync<object>("example.document", new PartitionKey("<Last Name>"));
    ```

1. 次のコード行を追加して、`ItemResponse<T>`インスタンスの**RequestCharge**プロパティの値を出力し、要求料金を返します (この値は後の演習で使用します)。

    ```csharp
    await Console.Out.WriteLineAsync($"{response.RequestCharge} RU/s");
    return response.RequestCharge;
    ```

1. メソッドは次のようになります。

    ```csharp
    private static async Task<double> ReadMember(Container peopleContainer)
    {
        ItemResponse<object> response = await peopleContainer.ReadItemAsync<object>("372a9e8e-da22-4f7a-aff8-3a86f31b2934", new PartitionKey("Batz"));
        await Console.Out.WriteLineAsync($"{response.RequestCharge} RU/s");
        return response.RequestCharge;
    }
    ```

1. 開いているすべてのエディタータブを保存します。

1.ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソール アプリケーションの出力を確認します。

    > 項目の ID とパーティションキーの値がある場合は、項目を直接取得するのにかかったRU/s(1 RU/sと ~3 RU/s) が少なくなることがわかります。これが非常に効率的である理由は、ReadItemAsync()がクエリエンジンを完全にバイパスし、バックエンドストアに直接移動してアイテムを取得するためです。この種類の読み取りで1KB以下のデータを読み取ると、常に1 RU/sのコストがかかります。

## 予想されるワークロードのスループットの設定

コンテナーまたはデータベースのスループットに適切な RU/s設定を使用すると、最小限のコストで目的のパフォーマンスを実現できます。適切なベースラインを決定し、予想される使用パターンに基づいて設定を変更することは、どちらも役立つ戦略です。

### スループットのニーズの見積もり

1. **Azure Cosmos DB**ブレードで、ブレードの左側にある**監視**セクションにある**メトリック（クラシック）**リンクを見つけて選択します。
1. **要求数**グラフの値を確認して、ラボで実行した内容ががコンテナーに対して行った要求の量を確認します。

    ![The Metrics dashboard is displayed](../media/09-metrics.jpg "Review your Cosmos DB metrics dashboard")

    > さまざまなパラメータを変更して、グラフに表示されるデータを調整でき、さらに分析するためにデータをcsvにエクスポートするオプションもあります。既存のアプリケーションの場合、これはクエリの量を決定するのに役立ちます。

1. Visual Studio Codeに戻り、`Main`メソッドを見つけます。次のような新しい行`await EstimateThroughput(peopleContainer);`を追加します。

    ```csharp
    public static async Task Main(string[] args)
    {

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        //await CreateMember(peopleContainer);
        //await CreateTransactions(transactionContainer);
        //await QueryTransactions(transactionContainer);
        //await QueryMember(peopleContainer);
        //await ReadMember(peopleContainer);
        await EstimateThroughput(peopleContainer);

    }
    ```

1. クラスの最後に、次のコードを含む新しい`EstimateThroughput`メソッドを追加します。

    ```csharp
    private static async Task EstimateThroughput(Container peopleContainer)
    {

    }
    ```

1. このメソッドに次のコード行を追加します。これらの変数は、アプリケーションの推定ワークロードを表します。

    ```csharp
    int expectedWritesPerSec = 200;
    int expectedReadsPerSec = 800;
    ```

    > これらのタイプの数値は、新しいアプリケーションの計画や既存のアプリケーションの実際の使用状況の追跡から得られる可能性があります。ワークロードの決定の詳細については、このラボの範囲外です。

1. 次に、`CreateMember`と`ReadMember`の呼び出しを追加し、それらから返された要求量をキャプチャします。これらの変数は、アプリケーションのこれらの操作の実際のコストを表します。

    ```csharp
    double writeCost = await CreateMember(peopleContainer);
    double readCost = await ReadMember(peopleContainer);
    ```


1. 次のコード行をこのメソッドの最後の行として追加して、テスト クエリに基づいてアプリケーションの推定スループット ニーズを出力します。
    ```csharp
    await Console.Out.WriteLineAsync($"Estimated load: {writeCost * expectedWritesPerSec + readCost * expectedReadsPerSec} RU/s");
    ```

1. `EstimateThroughput`メソッドは次のようになります。

    ```csharp
    private static async Task EstimateThroughput(Container peopleContainer)
    {
        int expectedWritesPerSec = 200;
        int expectedReadsPerSec = 800;

        double writeCost = await CreateMember(peopleContainer);
        double readCost = await ReadMember(peopleContainer);

        await Console.Out.WriteLineAsync($"Estimated load: {writeCost * expectedWritesPerSec + readCost * expectedReadsPerSec} RU/s");
    }
    ```

1. 開いているすべてのエディタータブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソールアプリケーションの出力を確認します。

    > 推定値に基づいて、アプリケーションに必要な合計スループットが表示されます。その後、これを使用して、アプリケーションにプロビジョニングするスループットの量をガイドできます。アプリケーションの RU/sのニーズを最も正確に見積もるには、同じパターンに従って、アプリケーション内のすべての操作の RU/sのニーズを推定し、1秒あたりに予想される操作の数を掛けます。または、ポータルのメトリックタブを使用して、平均スループットを測定することもできます。

### 使用パターンの調整

多くのアプリケーションには、予測可能な方法で時間の経過と共に変化するワークロードがあります。たとえば、5から9営業日の間に重いワークロードを持ち、その時間外の使用は最小限であるビジネス アプリケーションなどです。Cosmos のスループット設定は、この種類の使用パターンに合わせて変更することもできます。

1. `Main`メソッドを見つけて最後の行をコメントアウトし、新しい行`await UpdateThroughput(peopleContainer);`を追加して次のようにします。

    ```csharp
    public static async Task Main(string[] args)
    {

        Database database = _client.GetDatabase(_databaseId);
        Container peopleContainer = database.GetContainer(_peopleContainerId);
        Container transactionContainer = database.GetContainer(_transactionContainerId);

        //await CreateMember(peopleContainer);
        //await CreateTransactions(transactionContainer);
        //await QueryTransactions(transactionContainer);
        //await QueryMember(peopleContainer);
        //await ReadMember(peopleContainer);
        //await EstimateThroughput(peopleContainer);
        await UpdateThroughput(peopleContainer);
    }
    ```

1. クラスの一番下に新しい`UpdateThroughput`メソッドを作成します。

    ```csharp
    private static async Task UpdateThroughput(Container peopleContainer)
    {

    }
    ```

1. 次のコードを追加して、コンテナーの現在の RU/s設定を取得します。

    ```csharp
    int? throughput = await peopleContainer.ReadThroughputAsync();
    await Console.Out.WriteLineAsync($"{throughput} RU/s");
    ```

    > プロパティはnullを許容することに注意してください。プロビジョニングされたスループットは、コンテナーレベルまたはデータベースレベルで設定できます。データベースレベルで設定した場合、コンテナーから読み取られたこのプロパティはnullを返します。コンテナー レベルで設定すると、Databaseの同じメソッドはnullを返します

1. 次のコード行を追加して、コンテナーの最小スループット値を出力します。

    ```csharp
    ThroughputResponse throughputResponse = await peopleContainer.ReadThroughputAsync(new RequestOptions());
    int? minThroughput = throughputResponse.MinThroughput;
    await Console.Out.WriteLineAsync($"Minimum Throughput {minThroughput} RU/s");
    ```

    > 設定できる全体的な最小スループットは400 RU/秒ですが、格納されているデータのサイズ、以前の最大スループット設定、またはデータベース内のコンテナーの数によっては、特定のコンテナーまたはデータベースの制限が高くなることがあります。使用可能な最小値を下回る値を設定しようとすると、ここで例外が発生します。現在許容される最小値は、**ThroughputResponse.MinThroughput**プロパティで確認できます。

1. 次のコードを追加して、コンテナーの RU/秒設定を更新し、コンテナーの更新された RU/sを出力します。

    ```csharp
    await peopleContainer.ReplaceThroughputAsync(1000);
    throughput = await peopleContainer.ReadThroughputAsync();
    await Console.Out.WriteLineAsync($"New Throughput {throughput} RU/s");
    ```

1. 完成したメソッドは次のようになります。

    ```csharp
    private static async Task UpdateThroughput(Container peopleContainer)
    {
        int? throughput = await peopleContainer.ReadThroughputAsync();
        await Console.Out.WriteLineAsync($"Current Throughput {throughput} RU/s");

        ThroughputResponse throughputResponse = await container.ReadThroughputAsync(new RequestOptions());
        int? minThroughput = throughputResponse.MinThroughput;
        await Console.Out.WriteLineAsync($"Minimum Throughput {minThroughput} RU/s");

        await peopleContainer.ReplaceThroughputAsync(1000);
        throughput = await peopleContainer.ReadThroughputAsync();
        await Console.Out.WriteLineAsync($"New Throughput {throughput} RU/s");
    }
    ```

1. 開いているすべてのエディター タブを保存します。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソール アプリケーションの出力を確認します。

    > **1000**に変更する前に、最初にプロビジョニングされた値が表示されます。

1.  **Azure Cosmos DB**ブレードで、左側にある**データエクスプローラー**リンクを見つけて選択します。

1.  **データエクスプローラー**内で、**FinancialDatabase**データベースのノード、**PeopleCollection**コンテナーのノードの順に展開し、**Scale & Settings**オプションを選択します。

1. **Settings**セクションで、**Throughput**フィールドを見つけ、**1000**に設定されていることを確認します。

> 新しい値を表示するには、データ エクスプローラーを更新する必要がある場合があります。

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。
