# .NET SDK を使用したパーティション分割されたコンテナーの作成

このラボでは、異なるパーティション キーと設定を使用して、複数の Azure Cosmos DB コンテナーを作成します。ラボの後半では、SQL API と .NET SDK を使用して、1 つのパーティション キーまたは複数のパーティション キーを使用して特定のコンテナーに対してクエリを実行します。

> ラボ コンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## .NET SDK を使用したコンテナーの作成

.NET SDK を使用して、このラボと次のラボで使用するコンテナーを作成します。

### .NET Core プロジェクトを作成する

1. ローカルのコンピューター上でエクスプローラを開いて、 **ドキュメント** フォルダ内の **CosmosLabs** を見つけます。
1. .NET Core プロジェクトのコンテンツを格納するために使用される **Lab01** フォルダーを開きます。

   - このフォルダが存在しない場合は、[アカウントのセットアップ](00-account_setup.md)の手順で`labCodeSetup.ps1`を実行していない状態です。手動でコピーした場合には、コピー先のフォルダ内の **Lab1** フォルダを開いて下さい。

1. **Lab01** フォルダーで、フォルダーを右クリックし、**Codeで開く**メニューオプションを選択します。

    ![Open with Code is highlighted](../media/02-open_with_code.jpg "Open the directory with Visual Studio Code")

    > または、現在のディレクトリをターミナルで開いて `code .` コマンドを実行します。

1. 表示される Visual Studio Code ウィンドウで、**エクスプローラー** ペインを右クリックし、**ターミナルで開く** メニュー オプションを選択します。

    ![Open in Terminal is highlighted](../media/open_in_terminal.jpg "Open a terminal in Visual Studio Code")

1. 開いているターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet new console --output .
    ```

    > このコマンドを実行すると新しい .NET Coreプロジェクトが作成されます。 このプロジェクトは **console** プロジェクトになり、`--output .` オプションを指定することで現在のフォルダ上に直接作成されます。

    > 使用している **.NET SDKのバージョンが6.0以降** である場合には、`--use-program-main` オプションを追加で指定して、作成される **Program.cs** 内で **Main** メソッドが作成されるように明示します。

1. Visual Studio Codeでは、 **.NET Core** または **Azure Cosmos DB** の開発に関連するさまざまな拡張機能をインストールするように求められる可能性がありますが、これらの拡張機能はラボを完了するために必要なものではありません。

1. ターミナルペインで、次のコマンドを入力して実行します。

    ```sh
    dotnet add package Microsoft.Azure.Cosmos
    ```

    > このコマンドは [Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) NuGet パッケージをプロジェクトの依存関係に追加します。このラボの手順は、NuGetパッケージのバージョン3.12.0を使用してテストされています。

    `dotnet add package` が失敗する場合には、NuGetソースの参照先が間違っている可能性があります。次のコマンドを実行して、ソースをNuGetクライアントの設定に追加してください。
    ```sh
    dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org
    ```

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet add package Bogus
    ```

    > このコマンドは [Bogus](../media/https://www.nuget.org/packages/Bogus/) NuGet パッケージをプロジェクトの依存関係に追加します。このライブラリは、簡便なコードを記述することでテストデータを素早く生成することができるヘルパーライブラリです。ラボでは、このライブラリを使用して Azure Cosmos DB に登録するテストデータを生成します。このラボの手順は、NuGetパッケージのバージョン`30.0.2`を使用してテストされています。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet restore
    ```

    > このコマンドはプロジェクト内の依存関係として指定された全てのパッケージを復元します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet build
    ```

    > このコマンドはプロジェクトをビルドします。

1. .NET Core CLIによって作成された **Program.cs** ファイルと **[folder name].csproj** ファイルを確認します。

    ![The project file and the program.cs file are highlighted](../media/02-project_files.jpg "Review the Project files")

### CosmosClient インスタンスの作成

**CosmosClient** クラスは、Azure Cosmos DB でコア (SQL) API を使用するための主要な "エントリ ポイント" です。クラスのコンストラクターのパラメーターとして接続メタデータを渡すことによって、**CosmosClient** クラスのインスタンスを作成します。その後、ラボ全体でこのクラス インスタンスを使用します。



1. **Program.cs** をエディターで開き、以下のusingブロックを記述します。

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Microsoft.Azure.Cosmos;
    ```

1. `Program` クラス内に次のコード行を追加して、Azure Cosmos DBへの接続情報と **CosmosClient** の変数を作成します。

    ```csharp
    private static readonly string _endpointUri = "";
    private static readonly string _primaryKey = "";
    private static CosmosClient _client = new CosmosClient(_endpointUri, _primaryKey);
    ```

1. `_endpointUri` の値を、 Azure Cosmos DBアカウントの **URI** の値に置き換えます。 

    > 例として **uri** が `https://cosmosacct.documents.azure.com:443/` の場合、置き換えた結果は以下のようになります。
    
    ```csharp
    private static readonly string _endpointUri = "https://cosmosacct.documents.azure.com:443/";
    ```

1. `_primaryKey` の値を、 Azure Cosmos DBアカウントの **プライマリキー** の値に置き換えます。

    > 例として **プライマリキー** が ``elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==`` の場合、置き換えた結果は以下のようになります。

    ```csharp
    private static readonly string _primaryKey = "elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==";
    ```

    > **URI** と **PRIMARY KEY** の値はメモをしておいてください。ラボの後半で再度使用します。

1. `Main` メソッドを *async* に書き換えます。

    ```csharp
    public class Program
    {
        public static async Task Main(string[] args)
        {

        }
    }
    ```

1. この時点で、`Program` クラスは以下のように記述されている状態です。

    ```csharp
    public class Program
    {
        private static readonly string _endpointUri = "<your uri>";
        private static readonly string _primaryKey = "<your key>";
        private static CosmosClient _client = new CosmosClient(_endpointUri, _primaryKey);

        public static async Task Main(string[] args)
        {

        }
    }
    ```

    > 次に、アプリケーションをビルドしてコードが正常にコンパイルされることを確認します。

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet build
    ```

    > このコマンドは、プロジェクトをビルドし、エラーがないことを確認します。

### SDKを使用したデータベースの作成

1. `Main()` メソッドの下に、新しいメソッド `InitializeDatabase()` を追加します。

```csharp
    private static async Task<Database> InitializeDatabase(CosmosClient client, string databaseId)
    {
        Database database = await client.CreateDatabaseIfNotExistsAsync(databaseId);
        await Console.Out.WriteLineAsync($"Database Id:\t{database.Id}");
        return database;
    }
```

> このコードでは、引数 `databaseId` で指定した名前のデータベースがAzure Cosmos DBアカウントに存在するかを確認し、一致するデータベースが存在しない場合には新しく作成してそのデータベース名を返します。

1. `Main` メソッドに次の行を追加します。

    ```csharp
    Database database = await InitializeDatabase(_client, "EntertainmentDatabase");
    ```

1. `Main` メソッドの記述は以下のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {
        Database database = await InitializeDatabase(_client, "EntertainmentDatabase");
    }
    ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

    > 実行時のコマンドの出力を確認して下さい。コンソール上に Azure Cosmos DBアカウント内に作成されたデータベースのID文字列が表示されます。

### SDKを使用したパーティション分割されたコンテナーの作成

To create a container, you must specify a name and a partition key path. A partition key is a logical hint for distributing data onto a scaled out underlying set of physical partitions and for efficiently routing queries to the appropriate underlying partition. To learn more, refer to .

コンテナーを作成するには、名前とパーティション キーのパスを指定する必要があります。パーティション キーは、スケールアウトされた基になる物理パーティションのセットにデータを分散し、クエリを適切な基になるパーティションに効率的にルーティングするための論理的なヒントです。詳細については、[docs.microsoft.com/azure/cosmos-db/partition-data](../media/https://docs.microsoft.com/azure/cosmos-db/partition-data)を参照してください。

1. `InitializeDatabase` メソッドの下に以下のように新しいメソッドを追加します。

```csharp
    private static async Task<Container> InitializeContainer(Database database, string containerId)
    {

    }
```

1. 次のコードを追加して、カスタムインデックス作成ポリシーとして新しい `IndexingPolicy` インスタンスを作成します。

    ```csharp
    IndexingPolicy indexingPolicy = new IndexingPolicy
    {
        IndexingMode = IndexingMode.Consistent,
        Automatic = true,
        IncludedPaths =
        {
            new IncludedPath
            {
                Path = "/*"
            }
        },
        ExcludedPaths =
        {
            new ExcludedPath
            {
                Path = "/\"_etag\"/?"
            }
        }
    };
    ```

    > このインデックス作成ポリシーは、カスタムしたインデックス作成ポリシーが定義されていない場合に既定で適用されるインデックス作成ポリシーと同じ内容です。カスタムインデックス作成ポリシーを利用しない場合でも、全てのAzure Cosmos DBデータに対して既定のインデックス作成ポリシーを使用して自動的にインデックスが作成されます。多くのケースでは既定のインデックス作成ポリシーで十分ですが、カスタムインデックス作成ポリシーを利用することで除外パスを設定し、書き込みのパフォーマンスを向上させることができる可能性がありますが、除外したパスはクエリで利用することができなくなります。

1. インデックス作成ポリシーの下に次のコードを追加して、`type`をパーティションキーに持つ新しい `ContainerProperties` インスタンスを作成し、作成した `IndexingPolicy` を使用するように指定します。

    ```csharp
    ContainerProperties containerProperties = new ContainerProperties(containerId, "/type")
    {
        IndexingPolicy = indexingPolicy
    };
    ```

    > この定義により、`/type`パス上にパーティション キーが作成されます。パーティション キー パスでは、大文字と小文字が区別されます。これは、JSON プロパティの大文字と小文字を .NET CLR オブジェクトから JSON オブジェクトへのシリアル化のコンテキストで検討する場合に特に重要です。

1. 次のコード行を追加して、データベース内に新しい`Container`インスタンスが存在しない場合は作成します。以前に作成した設定とスループットの値を指定します。

    ```csharp
    Container container = await database.CreateContainerIfNotExistsAsync(containerProperties, 400);
    ```

    > このコードは、指定されたすべてのパラメーターを満たすコンテナーがデータベースに存在するかどうかを確認します。一致するコンテナーが存在しない場合は、新しいコンテナーが作成されます。ここで、新しく作成されたコンテナーに割り当てられた RU/秒を指定できます。これが指定されていない場合、SDK は既定値の 400 RU/秒でコンテナーを作成します。

1. 次のコードを追加して、データベースの ID を出力し、コンテナーを返します。

    ```csharp
    await Console.Out.WriteLineAsync($"Container Id:\t{container.Id}");
    return container;
    ```

    > `container`変数には、新しいコンテナーが作成されるか、既存のコンテナーが読み取られるかに関係なく、コンテナーに関するメタデータが含まれます。

1. `Main`メソッド内の`InitializeDatabase()`行を見つけ、その下に`InitializeContainer()`メソッドを追加して、新しい`Container`インスタンスを作成するようにします。

    ```csharp
    Container container = await InitializeContainer(database, "EntertainmentContainer");
    ```

1. `Main`メソッドの記述は以下のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {
        Database database = await InitializeDatabase(_client, "EntertainmentDatabase");
        Container container = await InitializeContainer(database, "EntertainmentContainer");
    }
    ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. 実行中のコマンドの出力を確認します。

## SDKを使用したコンテナへのアイテムの登録

> You will now use the .NET SDK to populate your container with various items of varying schemas. These items will be serialized instances of multiple C# classes in your project.
> .NET SDKを使用して、様々なスキーマのアイテムをコンテナに登録します。これらのアイテムは、プロジェクト内の複数のC#クラスでシリアル化されたインスタンスとして扱います。

### コンテナにデータを登録する。

1. Visual Studio Codeのコードウィンドウで、**エクスプローラー**ペインを開き、プロジェクトフォルダーに***DataType.cs**ファイルがあることを確認します。このファイルは、次の手順で使用するデータクラスです。プロジェクトフォルダにない場合は、次のパスからコピーします。`\labs\dotnet\setup\templates\Lab01\DataTypes.cs`

1. **Program.cs**ファイルを開きます。

1. `InitializeContainer()`メソッドの下に、次の新しいメソッドを追加します。

    ```csharp
    private static async Task LoadFoodAndBeverage(Container container)
    {

    }
    ```

    > 次のいくつかの手順では、**Bogus**ライブラリを使用してテストデータを作成します。このライブラリを使用すると、各オブジェクトのプロパティにダミーのデータセットを持つオブジェクトのコレクションを作成できます。このラボでは**Azure Cosmos DB に焦点を当てる**ことを目的としていますが、次の一連の手順ではこのライブラリを使用してテストデータを効率的に作成します。

1. 上記のメソッドに次のコードを追加して、`PurchaseFoodOrBeverage`インスタンスの列挙型コレクションを作成します。

    ```csharp
    var foodInteractions = new Bogus.Faker<PurchaseFoodOrBeverage>()
        .RuleFor(i => i.id, (fake) => Guid.NewGuid().ToString())
        .RuleFor(i => i.type, (fake) => nameof(PurchaseFoodOrBeverage))
        .RuleFor(i => i.unitPrice, (fake) => Math.Round(fake.Random.Decimal(1.99m, 15.99m), 2))
        .RuleFor(i => i.quantity, (fake) => fake.Random.Number(1, 5))
        .RuleFor(i => i.totalPrice, (fake, user) => Math.Round(user.unitPrice * user.quantity, 2))
        .GenerateLazy(100);
    ```

    > Bogusライブラリは一連のテストデータを生成します。この例では、Bogusライブラリと上記のルールを使用して 100 個のアイテムを作成しています。GenerateLazyメソッドは、IEnumerable型の変数を返すことによって、100 個のアイテムを準備するようにBogusライブラリに指示します。LINQは既定で遅延実行を使用するため、コレクションが参照されるまで項目は実際には作成されません。

1. 次の foreach ブロックを追加して、`PurchaseFoodOrBeverage`インスタンスを反復処理します。

    ```csharp
    foreach(var interaction in foodInteractions)
    {

    }
    ```

1. `foreach`ブロック内に次のコード行を追加して、コンテナー項目を非同期的に作成し、作成タスクの結果を変数に保存します。

    ```csharp
    ItemResponse<PurchaseFoodOrBeverage> result = await container.CreateItemAsync(interaction, new PartitionKey(interaction.type));
    ```

    > `CreateItemAsync`メソッドは、JSON にシリアル化されて指定されたコンテナー内にドキュメントとして格納されたオブジェクトを受け取ります。このデータの論理パーティションを指定することもできます。この場合は、PurchaseFoodOrBeverageクラスの名前です。`DataType.cs`クラスに示すように、新しいオブジェクトごとに一意のGuidとして生成されるプロパティは、インデックス作成に使用されるCosmos DBの特別な必須値であり、論理パーティション内のすべての項目に対して一意である必要があります。

1. `foreach` ブロック内に、次のコード行を追加して、新しく作成されたリソースの`id`プロパティの値をコンソールに書き込みます。

    ```csharp
    await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
    ```

    > `CosmosItemResponse`型には、項目を表すオブジェクトを含む名前付きプロパティ`Resource`と、挿入操作を実行するための要求料金やその ETag などの項目に関する興味深いデータにアクセスできるようにするその他のプロパティがあります。

1. `LoadFoodAndBeverage()`メソッドは次のようになります。

    ```csharp
    private static async Task LoadFoodAndBeverage(Container container)
    {
        var foodInteractions = new Bogus.Faker<PurchaseFoodOrBeverage>()
            .RuleFor(i => i.id, (fake) => Guid.NewGuid().ToString())
            .RuleFor(i => i.type, (fake) => nameof(PurchaseFoodOrBeverage))
            .RuleFor(i => i.unitPrice, (fake) => Math.Round(fake.Random.Decimal(1.99m, 15.99m), 2))
            .RuleFor(i => i.quantity, (fake) => fake.Random.Number(1, 5))
            .RuleFor(i => i.totalPrice, (fake, user) => Math.Round(user.unitPrice * user.quantity, 2))
            .GenerateLazy(100);

        foreach (var interaction in foodInteractions)
        {
            ItemResponse<PurchaseFoodOrBeverage> result = await container.CreateItemAsync(interaction, new PartitionKey(interaction.type));
            await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
        }
    }
    ```

1. `Main`メソッド内で`InitalizeContainer()`メソッドを見つけ、`LoadFoodAndBeverage()`メソッドを呼び出す次のコードを追加します。

    ```csharp
    await LoadFoodAndBeverage(container);
    ```

1. `Main`メソッドの記述は以下のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {
        Database database = await InitializeDatabase(_client, "EntertainmentDatabase");
        Container container = await InitializeContainer(database, "EntertainmentContainer");

        await LoadFoodAndBeverage(container);
    }
    ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソール アプリケーションの出力を確認します。このツールによって作成される新しいアイテムに関連付けられているアイテム ID のリストが表示されます。

    ![Terminal output displayed with Items being created](../media/01-item-creation.png "Review the output, notice unique document ids being created")

### コンテナに異なるタイプのデータを登録する

1. `LoadFoodAndBeverage()`メソッドの下に、次の新しいメソッドを作成します。

    ```csharp
    private static async Task LoadTelevision(Container container)
    {

    }
    ```

1. 上記のメソッドに次のコードを追加して、`WatchLiveTelevisionChannel`インスタンスの列挙型コレクションを作成します。

    ```csharp
    var tvInteractions = new Bogus.Faker<WatchLiveTelevisionChannel>()
        .RuleFor(i => i.id, (fake) => Guid.NewGuid().ToString())
        .RuleFor(i => i.type, (fake) => nameof(WatchLiveTelevisionChannel))
        .RuleFor(i => i.minutesViewed, (fake) => fake.Random.Number(1, 45))
        .RuleFor(i => i.channelName, (fake) => fake.PickRandom(new List<string> { "NEWS-6", "DRAMA-15", "ACTION-12", "DOCUMENTARY-4", "SPORTS-8" }))
        .GenerateLazy(100);

    foreach (var interaction in tvInteractions)
    {
        ItemResponse<WatchLiveTelevisionChannel> result = await container.CreateItemAsync(interaction, new PartitionKey(interaction.type));
        await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
    }
    ```

1. `Main`メソッドに移動し、`LoadFoodAndBeverage()`メソッドをコメントアウトして`LoadTelevision()`を呼び出す新しい行を追加します。メソッドは次のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {
        Database database = await InitializeDatabase(_client, "EntertainmentDatabase");
        Container container = await InitializeContainer(database, "EntertainmentContainer");

        //await LoadFoodAndBeverage(container);
        await LoadTelevision(container);
    }
    ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソール アプリケーションの出力を確認します。このツールによって作成される新しいアイテムに関連付けられているアイテム ID のリストが表示されます。

1. `LoadTelevision()` メソッドの下に、次の実装で新しい`LoadMapViews()`メソッドを作成します。

    ```csharp
    private static async Task LoadMapViews(Container container)
    {
        var mapInteractions = new Bogus.Faker<ViewMap>()
            .RuleFor(i => i.id, (fake) => Guid.NewGuid().ToString())
            .RuleFor(i => i.type, (fake) => nameof(ViewMap))
            .RuleFor(i => i.minutesViewed, (fake) => fake.Random.Number(1, 45))
            .GenerateLazy(100);

        foreach (var interaction in mapInteractions)
        {
            ItemResponse<ViewMap> result = await container.CreateItemAsync(interaction);
            await Console.Out.WriteLineAsync($"Item Created\t{result.Resource.id}");
        }
    }
    ```

1. `Main`メソッドに移動し、`LoadTelevision()`メソッドをコメントアウトして`LoadMapViews()`を呼び出す新しい行を追加します。メソッドは次のようになります。

    ```csharp
    public static async Task Main(string[] args)
    {
        Database database = await InitializeDatabase(_client, "EntertainmentDatabase");
        Container container = await InitializeContainer(database, "EntertainmentContainer");

        //await LoadFoodAndBeverage(container);
        //await LoadTelevision(container);
        await LoadMapViews(container);
    }
    ```

1. 開いているエディターの内容を全て保存します。

1. ターミナルペインで以下のコマンドを入力して実行します。

    ```sh
    dotnet run
    ```

1. コンソール アプリケーションの出力を確認します。このツールによって作成される新しいアイテムに関連付けられているアイテム ID のリストが表示されます。これで、３種類のドキュメント(PurchaseFoodOrBeverage, WatchLiveTelevisionChannel, ViewMap) が`EntertainmentContainer`コンテナに登録できました。このことは、Cosmos DBがスキーマレスであることを示しています。

1. Visual Studio Codeのフォルダーを閉じます。

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。
