# Azure Cosmos DB変更フィード

このラボでは、変更フィード プロセッサ ライブラリと Azure Functionsを使用して、Azure Cosmos DB 変更フィードの 3 つのユース ケースを実装します。

>  これが初めてのラボであり、ラボコンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。

## データ作成の.NETコンソールアプリを実装する

e-コマースWebサイトでの操作を想定してカート内のデータをシミュレートするために、ドキュメントを生成してCosmos DBのCartContainerに登録する単純な.NETコンソールアプリを構築します。

1. ローカルコンピューターで、ドキュメントフォルダー内のCosmosLabsフォルダーを見つけ、.NET Core プロジェクトのコンテンツを格納するために使用される`Lab08`フォルダーを開きます。マイクロソフト ハンズオン ラボを通じてこのラボを完了する場合、CosmosLabsフォルダーは**C:\labs\CosmosLabs**のパスにあります。

1. `Lab08`フォルダーでフォルダを右クリックし、**Codeで開く**メニューオプションを選択します。

   > または、現在のディレクトリでターミナルを実行して`code .`コマンドを実行することもできます。

1. 左側のエクスプローラーウィンドウで、**DataGenerator**フォルダーを見つけて展開します。

1. **Explorer**ウィンドウで`Program.cs`リンクを選択して、エディターでファイルを開きます。

   ![The program.cs is displayed](../media/08-console-main-default.jpg "Open the program.cs file")

1. 以前のラボでメモしていたAzure Cosmos DBの資格情報から、`_endpointUri`変数の値に、**URI**を、`_primaryKey`変数の値に**プライマリーキー**を入力してください。資格情報が不明な場合は、[こちら](00-account_setup.md)の手順を参照してください。

    - 例として **uri** が `https://cosmosacct.documents.azure.com:443/` の場合、記述は以下のようになります

    ```csharp
    private static readonly string _endpointUri = "https://cosmosacct.documents.azure.com:443/";
    ```

    - 例として **プライマリキー** が ``elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==`` の場合、記述は以下のようになります。

    ```csharp
    private static readonly string _primaryKey = "elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==";
    ```

### Cosmos DBにドキュメントを登録する関数を作成する

コンソールアプリケーションの主な機能は、ドキュメントを Cosmos DBに追加して、eコマースWebサイトでのアクティビティをシミュレートすることです。ここでは、これらのドキュメントのデータ定義を作成し、それらを追加する関数を定義します。

1. **DataGenerator**フォルダー内の`Program.cs`ファイル内で、`AddItem()`メソッドを見つけて確認します。このメソッドの目的は、**CartAction**のインスタンスをCosmosDBコンテナーに追加することです

   > Cosmos DBコンテナーにドキュメントを追加する方法を確認する場合は、[Lab 01](01-creating_partitioned_collection.md)を参照してください。

### ランダムなショッピングデータを生成する関数を作成する

1. **DataGenerator**フォルダー内の`Program.cs`ファイル内で、`GeneratingActions()`メソッドを見つけて確認します。このメソッドの目的は、Cosmos DBの変更フィードを利用してランダム化された**CartAction**オブジェクトを作成することです。


### アプリケーションの実行と機能の確認

コンソールアプリケーションを実行する準備ができたら、アプリケーションを実行し、テストデータが期待通りに書き込まれていることを確認します。

1. ターミナルウィンドウを開きます。
2. ターミナルウィンドウで、次のコマンドを入力してコンソールアプリを実行します。

   ```sh
   cd DataGenerator

   dotnet run
   ```

3. アプリケーションがビルドされると、データが生成されてCosmos DBに登録されます。その際、コンソールにはアスタリスクが出力されます。

   ![The terminal window is displayed showing the program running outputting asterisks](../media/08-console-running.jpg "Run the program, let it run for a minute or two")

4. アプリケーションを1～2分程度実行してから、任意のキーを押してアプリケーションを停止します。

5. Azure PortalでCosmos DB Accountを表示します。

6. **Azure Cosmos DB**ブレードで、ブレードの左側にある**データエクスプローラー**リンクを見つけて選択します。

   ![The Cosmos DB resource with the Data Explorer highlighted](../media/08-cosmos-overview-final.jpg "Open the Data Explorer")

7. **StoreDatabase**を展開し、さらに**CartContainer**を展開して、**Items**を選択します。次のスクリーンショットのように表示されます。

   > データはランダムに生成されているため、表示は異なります。データが登録されていることを確認して下さい。

   ![An item in the StoreDatabase is selected](../media/08-cosmos-data-explorer-with-data.jpg "Select an item in the StoreDatabase")

## 変更フィードプロセッサ経由でCosmos DB変更フィードを使用する

変更フィードを使用するための主な方法は、Azure Functionsと変更フィードプロセッサライブラリです。ここでは、コンソールアプリケーションで変更フィードプロセッサを使用します。

### 変更フィードに接続する

変更フィードを試す初めのユースケースは、ライブマイグレーションです。Cosmos DBコンテナーを設計する際の一般的な懸念事項は、パーティションキーの適切な選択です。このラボでは、`CartContainer`のパーティションキーは`/Item`です。しかし、実際に`/Item`が適切に機能するのは書き込みのみで、読み取りの場合は`/BuyerState`のほうが適切であるとすると、`CartContainer`は読み取り性能に問題があると言えます。変更フィードを利用すれば、別のパーティションキーを持つコンテナーにリアルタイムにデータを連係し、このようなパフォーマンス問題を回避することができます。

1. Visual Studio Codeに戻ります。

2. **エクスプローラー**ウィンドウから、**ChangeFeedConsole**フォルダーの下にある`program.cs`を選択してエディタで開きます。

3. 以前のラボでメモしていたAzure Cosmos DBの資格情報から、`_endpointUri`変数の値に、**URI**を、`_primaryKey`変数の値に**プライマリーキー**を入力してください。資格情報が不明な場合は、[こちら](00-account_setup.md)の手順を参照してください。

4. `program.cs`ファイルの先頭にある、宛先コンテナの名前（`_containerId`）の値を確認して下さい。

   ```csharp
   private static readonly string _destinationContainerId = "CartContainerByState";
   ```

   > この場合、同じデータベース内の別のコンテナーにデータを移行します。データを完全に別のデータベースに移行したい場合にも、同様な考え方で実現できます。

5. 変更フィードを使用するために、**リースコンテナー**を使用します。`//todo: Add lab code here`の箇所を以下のコードに書き換えて、リースコンテナーを作成します。

   ```csharp
   ContainerProperties leaseContainerProperties = new ContainerProperties("consoleLeases", "/id");
   Container leaseContainer = await database.CreateContainerIfNotExistsAsync(leaseContainerProperties, throughput: 400);
   ```

   > The **Lease Container** stores information to allow for parallel processing of the change feed, and acts as a book mark for where we last processed changes from the feed.
   > **リースコンテナー**には、変更フィードの並列処理を可能にする情報が格納され、フィードからの変更を最後に処理した場所のブックマークとして機能します。

6. 次に、変更プロセッサのインスタンスを取得するために**leaseContainer**を作成したコードの直後に、次のコードを追加します。

   ```csharp
   var builder = container.GetChangeFeedProcessorBuilder("migrationProcessor", (IReadOnlyCollection<object> input, CancellationToken cancellationToken) => {
       Console.WriteLine(input.Count + " Changes Received");
       //todo: Add processor code here
   });

   var processor = builder
                   .WithInstanceName("changeFeedConsole")
                   .WithLeaseContainer(leaseContainer)
                   .Build();
   ```

   > 変更を受信する度に、`GetChangeFeedProcessorBuilder`に定義された`Func<T>`が呼び出されます。現時点では、これらの変更の処理をスキップしています。

7. プロセッサを実行するには、プロセッサを起動する必要があります。先ほどのコードの下に次のコード行を追加します。

   ```csharp
   await processor.StartAsync();
   ```

8. 最後に、プロセッサを終了するためにキーが押されたら、プロセッサを終了する必要があります。`//todo: Add stop code here`と書かれた行を見つけて、次のコードに置き換えます。

   ```csharp
   await processor.StopAsync();
   ```

9. At this point, your `Program.cs` file should look like this:
この時点で、`Program.cs`ファイルは次のようになります。

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Threading;
   using System.Threading.Tasks;
   using Microsoft.Azure.Cosmos;

   namespace ChangeFeedConsole
   {
       class Program
       {
           private static readonly string _endpointUrl = "<your-endpoint-url>";
           private static readonly string _primaryKey = "<your-primary-key>";
           private static readonly string _databaseId = "StoreDatabase";
           private static readonly string _containerId = "CartContainer";
           private static readonly string _destinationContainerId = "CartContainerByState";
           private static CosmosClient _client = new CosmosClient(_endpointUrl, _primaryKey);

           static async Task Main(string[] args)
           {

               Database database = _client.GetDatabase(_databaseId);
               Container container = db.GetContainer(_containerId);
               Container destinationContainer = db.GetContainer(_destinationContainerId);

               ContainerProperties leaseContainerProperties = new ContainerProperties("consoleLeases", "/id");
               Container leaseContainer = await  db.CreateContainerIfNotExistsAsync(leaseContainerProperties, throughput: 400);

               var builder = container.GetChangeFeedProcessorBuilder("migrationProcessor",
                  (IReadOnlyCollection<object> input, CancellationToken cancellationToken) =>
                  {
                     Console.WriteLine(input.Count + " Changes Received");
                     //todo: Add processor code here
                  });

               var processor = builder
                               .WithInstanceName("changeFeedConsole")
                               .WithLeaseContainer(leaseContainer)
                               .Build();

               await processor.StartAsync();
               Console.WriteLine("Started Change Feed Processor");
               Console.WriteLine("Press any key to stop the processor...");

               Console.ReadKey();

               Console.WriteLine("Stopping Change Feed Processor");
               await processor.StopAsync();
           }
       }
   }
   ```

### ライブデータマイグレーションの完了

1. **ChangeFeedConsole**フォルダーの`Program.cs`内で、`//todo: Add processor code here`と書かれた行を見つけます。

1. `GetChangeFeedProcessorBuilder`内の`Func<T>`のシグネチャ`object`を`CartAction`に変更します。以下のようになります。

   ```csharp
   var builder = container.GetChangeFeedProcessorBuilder("migrationProcessor", 
      (IReadOnlyCollection<CartAction> input, CancellationToken cancellationToken) =>
      {
         Console.WriteLine(input.Count + " Changes Received");
         //todo: Add processor code here
      });
   ```

1. **input**は、変更された**CartAction**内のドキュメントのコレクションです。これらを移行するには、ループ処理で宛先のコンテナーに書き出すだけです。`//todo: Add processor code here`を、次のコード行に置き換えます。

   ```csharp
   var tasks = new List<Task>();

   foreach (var doc in input)
   {
      tasks.Add(destinationContainer.CreateItemAsync(doc, new PartitionKey(doc.BuyerState)));
   }

   return Task.WhenAll(tasks);
   ```

### 変更フィードのテスト

変更フィードを処理するアプリケーションができたので、テストを実行することができます。

1. **2番目の**ターミナルウィンドウを開き、**ChangeFeedConsole**フォルダーに移動します。

1. 変更フィードを処理するアプリケーションを起動するために、**2番目の**ターミナルウィンドウで次のコマンドを実行します。

   ```sh
   cd ChangeFeedConsole

   dotnet run
   ```

1. 関数の実行が開始されると、コンソールに次のメッセージが表示されます。

   ```sh
   Started Change Feed Processor
   Press any key to stop the processor...
   ```

   > 変更フィードを処理するのはこれが初めてであるため、この時点では処理するデータはありません。変更を受信するために、データ生成アプリケーションを使用します。

1. **最初の**ターミナルウィンドウで、**DataGenerator**フォルダに移動します。

1. 次のコマンドを**最初の**ターミナルウィンドウで実行して、**DataGenerator**アプリケーションを再度実行します。

   ```sh
   dotnet run
   ```

1. データが書き込まれると、アスタリスクが再び表示され始めるはずです。

1. データの書き込みが開始されるとすぐに、**2番目の**ターミナルウィンドウに次の出力が表示され始めます。

   ```sh
   100 Changes Received
   100 Changes Received
   3 Changes Received
   ...
   ```

1. 数分後、**cosmosdblab**データエクスプローラーに移動し、**StoreDatabase**、**CartContainerByState**の順に展開し、 **Items**を選択します。そこに項目が登録されていることが確認できます。コンテナーのパーティションキーが`/BuyerState`であることに注目してください。 

   ![The Cart Container By State is displayed](../media/08-cart-container-by-state.jpg "Open the CartContainerByState and review the items")

1. **最初の**ターミナルウィンドウで任意のキーを押して、データ生成を停止します。

1. **ChangeFeedConsole**の実行を終了させます。データ生成を終了させてしばらくすると、新しいログメッセージの書き込みが停止して完了したことがわかります。**2番目の**ターミナルウィンドウで任意のキーを押して機能を停止します。

> これで、ライブデータを新しいコレクションに書き込む最初のCosmos DB変更フィードアプリケーションが作成されました。次の手順では、Azure Functionsを使用して Cosmos DB変更フィードを使用する方法について、さらに2つのユース ケースについて説明します。

## 変更フィードを使用する Azure Functionsを作成する

### .NET CoreのAzure Functionsプロジェクトを作成する

この演習では、.NET SDKの変更フィードプロセッサライブラリを実装して、スケーラブルでフォールトトレラントな方法で Azure Cosmos DBの変更フィードを読み取ります。Azure Functionsでは、変更フィードプロセッサをネイティブにサポートすることで、Cosmos DBの変更フィードにすばやく簡単に接続できます。まずは、.NET CoreでAzure Functionsプロジェクトを作成します。

> 詳細については、[doc](https://docs.microsoft.com/azure/cosmos-db/sql/change-feed-processor)を参照してください。

1. ターミナル ウィンドウを開き、このラボで使用している Lab08 フォルダーに移動します。

1. Azure Functions Core Toolsを利用するために、`node.js`が必要です。

> node.jsがインストールされていない場合、[こちら](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-run-local?tabs=v4%2Cwindows%2Ccsharp%2Cportal%2Cbash#install-the-azure-functions-core-tools)を参考に、Core Toolsのパッケージを導入することもできます。

1. ターミナルウィンドウで次のコマンドを入力して実行し、nodeバージョンを確認します。

   ```sh
   node --version
   ```

   > node.jsがインストールされていない場合は、[こちら](https://docs.npmjs.com/getting-started/installing-node#osx-or-windows)からダウンロードしてください。`8.5`より古いバージョンをインストールしている場合には、次のコマンドを実行してください。
   
   ```sh
   npm i -g node@latest
   ```

1. 次のコマンドを入力して実行し、Azure Function Core Toolsをダウンロードします。

   ```sh
   npm install -g azure-functions-core-tools
   ```

   > このコマンドが失敗した場合は、node.jsのセットアップに関する前のステップを参照してください。これらの変更を有効にするには、ターミナルウィンドウを再起動する必要がある場合があります。

1. ターミナルウィンドウで、次のコマンドを入力して実行します。このコマンドにより、新しいAzure Functionsプロジェクトが作成されます。

   ```sh
   func init ChangeFeedFunctions
   ```

1. メッセージが表示されたら、**worker runtime**として**dotnet**を選択します。矢印キーを使用して上下にスクロールします。

1. メッセージが表示されたら、**language**として**C#**を選択します。矢印キーを使用して上下にスクロールします。

1. 前の手順で作成された`ChangeFeedFunctions`ディレクトリに変更します

   ```sh
   cd ChangeFeedFunctions
   ```

1. ターミナルウィンドウで、次のコマンドを入力して実行します。

   ```sh
   func new
   ```

1. メッセージが表示されたら、テンプレートの一覧から**CosmosDBTrigger**を選択します。矢印キーを使用して上下にスクロールします。

1. プロンプトが表示されたら、関数の名前に`MaterializedViewFunction`を入力します。

1. **ChangeFeedFunctions.csproj** ファイルを開き、ターゲット フレームワークを .NET Core 7.0に更新します。

    ```xml
   <TargetFramework>net7.0</TargetFramework>
    ```

1. ターミナルウィンドウで、次のコマンドを入力して実行します。

   ```sh
   dotnet add package Microsoft.Azure.Cosmos
   dotnet add package Microsoft.NET.Sdk.Functions --version 3.0.9
   dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB
   dotnet add ChangeFeedFunctions.csproj reference ..\\Shared\\Shared.csproj
   ```

1. これで**ChangeFeedFunctions**フォルダの下に新しいAzure Functionsプロジェクトが作成されました。

# 変更フィードを使用してマテリアライズドビューを作成する

マテリアライズドビューパターンは、ソースデータ形式がアプリケーションの要件にあまり適していない環境で、データの事前設定されたビューを生成するために使用されます。この例では、州別に集計された売上データのリアルタイムコレクションを作成し、別のアプリケーションが売上の概要データをすばやく取得できるようにします。

### マテリアライズドビューのAzure Functionを作成する

1. **local.settings.json**ファイルを見つけて選択し、エディターで開きます。

1. Add a new value `DBConnection` using the **Primary Connection String** parameter from your Cosmos DB account collected earlier in this lab. The **local.settings.json** file should like this:このラボの前半で収集した Cosmos DBアカウントの**プライマリ接続文字列**パラメーターを使用して、新しい値を追加します。**local.settings.json**ファイルは次のようになります。

   ```json
   {
     "IsEncrypted": false,
     "Values": {
       "AzureWebJobsStorage": "UseDevelopmentStorage=true",
       "FUNCTIONS_WORKER_RUNTIME": "dotnet",
       "DBConnection": "<your-db-connection-string>"
     }
   }
   ```

1. Select the new  file to open it in the editor.`MaterializedViewFunction.cs`ファイルを選択して、エディターで開きます。

   > The ,  and  refer to the source Cosmos DB account that the function is listening for changes on.**databaseName**、**collectionName**、および**ConnectionStringSetting**は、関数が変更をリッスンしているCosmos DBアカウントを参照します。

1. **databaseName**の値を`StoreDatabase`に変更します。

1.  **collectionName**の値を`CartContainerByState`に変更します。

   > Cosmos DB 変更フィードはパーティション内で順序付けられることが保証されているため、この場合、パーティションが既に状態に設定されている`CartContainerByState`コンテナーをソースとして使用します。

1. **ConnectionStringSetting**の値を、前の手順で設定した**DBConnection**に置き換えます。

   ```csharp
   ConnectionStringSetting = "DBConnection",
   ```

1. **ConnectionStringSetting**と**LeaseCollectionName**の間に次の行を追加します。

   ```csharp
   CreateLeaseCollectionIfNotExists = true,
   ```

1. **LeaseCollectionName**の値を`materializedViewLeases`に変更します。

   > リース コレクションは、Cosmos DB 変更フィードの重要な部分です。これにより、関数の複数のインスタンスがコレクションに対して動作し、関数が最後に中断した場所の仮想ブックマークとして機能します。

1. **Run**関数は次のようになります。

   ```csharp
   [FunctionName("MaterializedViewFunction")]
   public static void Run([CosmosDBTrigger(
      databaseName: "StoreDatabase",
      collectionName: "CartContainerByState",
      ConnectionStringSetting = "DBConnection",
      CreateLeaseCollectionIfNotExists = true,
      LeaseCollectionName = "materializedViewLeases")]IReadOnlyList<Document> input, ILogger log)
   {
      if (input != null && input.Count > 0)
      {
         log.LogInformation("Documents modified " + input.Count);
         log.LogInformation("First document Id " + input[0].Id);
      }
   }
   ```

> この関数は、コンテナーを一定の間隔でポーリングし、最後のリース時間以降の変更を確認することで機能します。関数のターンごとに複数のドキュメントが変更される可能性があるため、引数はDocumentのIDeadOnlyListです。

1. 次の using ステートメントを`MaterializedViewFunction.cs`ファイルの先頭に追加します。

   ```csharp
   using System.Threading.Tasks;
   using System.Linq;
   using Newtonsoft.Json;
   using Microsoft.Azure.Cosmos;
   using Shared;
   ```

1. **Run**関数のシグネチャを戻り値の型を`async Task`に変更します。関数は次のようになります。

   ```csharp
   [FunctionName("MaterializedViewFunction")]
      public static async Task Run([CosmosDBTrigger(
         databaseName: "StoreDatabase",
         collectionName: "CartContainerByState",
         ConnectionStringSetting = "DBConnection",
         CreateLeaseCollectionIfNotExists = true,
         LeaseCollectionName = "materializedViewLeases")]IReadOnlyList<Document> input, ILogger log)
      {
         if (input != null && input.Count > 0)
         {
            log.LogInformation("Documents modified " + input.Count);
            log.LogInformation("First document Id " + input[0].Id);
         }
      }
   ```

1. 今回のターゲットは、**StateSales**というコンテナーです。**MaterializedViewFunction**の先頭に次の行を追加して、宛先接続を設定します。エンドポイントURLとキーは必ず置き換えてください。

   ```csharp
    private static readonly string _endpointUrl = "<your-endpoint-url>";
    private static readonly string _primaryKey = "<your-primary-key>";
    private static readonly string _databaseId = "StoreDatabase";
    private static readonly string _containerId = "StateSales";
    private static CosmosClient _client = new CosmosClient(_endpointUrl, _primaryKey);
   ```

### StateSalesデータのクラスを作成する

1. **Shared**フォルダーの下にある`DataModel.cs`を開きます。

1. **CartAction**クラスの定義の下に、次のように新しいクラスを追加します。

   ```csharp
   public class StateCount
   {
      [JsonProperty("id")]
      public string Id { get; set; }
      public string State { get; set; }
      public int Count { get; set; }
      public double TotalSales { get; set; }

      public StateCount()
      {
         Id = Guid.NewGuid().ToString();
      }
   }
   ```

### MaterializedViewFunctionを実装してマテリアライズドビューを作成する

Azure Functionsは、変更されたドキュメントの一覧を受け取ります。このリストを各ドキュメントの状態をディクショナリ形式に整理し、購入したアイテムの合計価格と数を追跡したいと考えています。後でこのディクショナリを使用して、具体化されたビューコレクション**StateSales**にデータを書き込みます。

1. エディターで**MaterializedViewFunction.cs**ファイルに戻ります。

1. **MaterializedViewFunction.cs**のコードで次のセクションを見つけます。

   ```csharp
   if (input != null && input.Count > 0)
   {
      log.LogInformation("Documents modified " + input.Count);
      log.LogInformation("First document Id " + input[0].Id);
   }
   ```

1. 2つのログ出力行を次のコードに置き換えます。

   ```csharp
   var stateDict = new Dictionary<string, List<double>>();
   foreach (var doc in input)
   {
      var action = JsonConvert.DeserializeObject<CartAction>(doc.ToString());

      if (action.Action != ActionType.Purchased)
      {
         continue;
      }

      if (stateDict.ContainsKey(action.BuyerState))
      {
         stateDict[action.BuyerState].Add(action.Price);
      }
      else
      {
         stateDict.Add(action.BuyerState, new List<double> { action.Price });
      }
   }
   ```

1. このforeachループの終了後、次のコードを追加して宛先コンテナーに接続します。

   ```csharp
      var database = _client.GetDatabase(_databaseId);
      var container = database.GetContainer(_containerId);

      //todo - Next steps go here
   ```

1. 集計コレクションを扱っているため、ディクショナリの各エントリのドキュメントを作成または更新します。手始めに、処理が必要なドキュメントが存在するかどうかを確認する必要があります。上記の行の後に次のコードを追加します。

   ```csharp
   var tasks = new List<Task>();

   foreach (var key in stateDict.Keys)
   {
      var query = new QueryDefinition("select * from StateSales s where s.State = @state").WithParameter("@state", key);

      var resultSet = container.GetItemQueryIterator<StateCount>(query, requestOptions: new QueryRequestOptions() { PartitionKey = new Microsoft.Azure.Cosmos.PartitionKey(key), MaxItemCount = 1 });

      while (resultSet.HasMoreResults)
      {
         var stateCount = (await resultSet.ReadNextAsync()).FirstOrDefault();

         if (stateCount == null)
         {
            //todo: Add new doc code here
         }
         else
         {
            //todo: Add existing doc code here
         }

         //todo: Upsert document
      }
   }

   await Task.WhenAll(tasks);
   ```

   > **QueryRequestOptions**の _MaxItemCount_ に注意します。各州には最大で1つのドキュメントがあるため、最大で1つの結果しか期待できません。

1. stateCountオブジェクトが _null_ の場合は、新しいオブジェクトを作成します。`//todo: Add new doc code here`セクションを次のコードに置き換えます。

   ```csharp
   stateCount = new StateCount();
   stateCount.State = key;
   stateCount.TotalSales = stateDict[key].Sum();
   stateCount.Count = stateDict[key].Count;
   ```

1. stateCount オブジェクトが存在する場合は、それを更新します。`//todo: Add existing doc code here`セクションを次のコードに置き換えます。

   ```csharp
    stateCount.TotalSales += stateDict[key].Sum();
    stateCount.Count += stateDict[key].Count;
   ```

1. 最後に、宛先のCosmos DBアカウントでアップサート（更新または挿入）操作を実行します。`//todo: Upsert document`セクションを次のコードに置き換えます。

   ```csharp
   log.LogInformation("Upserting materialized view document");
   tasks.Add(container.UpsertItemAsync(stateCount, new Microsoft.Azure.Cosmos.PartitionKey(stateCount.State)));
   ```

   > ここでは、アップサートを並行して実行するため、タスクのリストを使用しています。

1. Your **MaterializedViewFunction** should now look like this:**MaterializedViewFunction**は次のようになります。

   ```csharp
   using System;
   using System.Collections.Generic;
   using Microsoft.Azure.Documents;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Host;
   using Microsoft.Extensions.Logging;
   using System.Threading.Tasks;
   using System.Linq;
   using Newtonsoft.Json;
   using Microsoft.Azure.Cosmos;
   using Shared;


   namespace ChangeFeedFunctions
   {
      public static class MaterializedViewFunction
      {
         private static readonly string _endpointUrl = "<your-endpoint-url>";
         private static readonly string _primaryKey = "<primary-key>";
         private static readonly string _databaseId = "StoreDatabase";
         private static readonly string _containerId = "StateSales";
         private static CosmosClient _client = new CosmosClient(_endpointUrl, _primaryKey);

         [FunctionName("MaterializedViewFunction")]
         public static async Task Run([CosmosDBTrigger(
            databaseName: "StoreDatabase",
            collectionName: "CartContainerByState",
            ConnectionStringSetting = "DBConnection",
            CreateLeaseCollectionIfNotExists = true,
            LeaseCollectionName = "materializedViewLeases")]IReadOnlyList<Document> input, ILogger log)
         {
            if (input != null && input.Count > 0)
            {
               var stateDict = new Dictionary<string, List<double>>();

               foreach (var doc in input)
               {
                  var action = JsonConvert.DeserializeObject<CartAction>(doc.ToString());

                  if (action.Action != ActionType.Purchased)
                  {
                     continue;
                  }

                  if (stateDict.ContainsKey(action.BuyerState))
                  {
                     stateDict[action.BuyerState].Add(action.Price);
                  }
                  else
                  {
                     stateDict.Add(action.BuyerState, new List<double> { action.Price });
                  }
               }

               var database = _client.GetDatabase(_databaseId);
               var container = database.GetContainer(_containerId);

               var tasks = new List<Task>();

               foreach (var key in stateDict.Keys)
               {
                  var query = new QueryDefinition("select * from StateSales s where s.State = @state").WithParameter("@state", key);

                  var resultSet = container.GetItemQueryIterator<StateCount>(query, requestOptions: new QueryRequestOptions() { PartitionKey = new Microsoft.Azure.Cosmos.PartitionKey(key), MaxItemCount = 1 });

                  while (resultSet.HasMoreResults)
                  {
                     var stateCount = (await resultSet.ReadNextAsync()).FirstOrDefault();

                     if (stateCount == null)
                     {
                        stateCount = new StateCount();
                        stateCount.State = key;
                        stateCount.TotalSales = stateDict[key].Sum();
                        stateCount.Count = stateDict[key].Count;
                     }
                     else
                     {
                        stateCount.TotalSales += stateDict[key].Sum();
                        stateCount.Count += stateDict[key].Count;
                     }

                     log.LogInformation("Upserting materialized view document");
                     tasks.Add(container.UpsertItemAsync(stateCount, new Microsoft.Azure.Cosmos.PartitionKey(stateCount.State)));
                  }
               }

               await Task.WhenAll(tasks);
            }
         }
      }
   }
   ```

### マテリアライズドビュー関数が機能することを確認する

1. Open three terminal windows.3つのターミナル ウィンドウを開きます。

1. **最初の**ターミナルウィンドウで、**DataGenerator**フォルダに移動します。

1. **最初の**ターミナルウィンドウで以下を入力して実行することにより、**DataGenerator**を起動します。

   ```sh
   dotnet run
   ```

1. **2番目**のターミナルウィンドウで、**ChangeFeedConsole**フォルダーに移動します。

1. **2番目の**ターミナルウィンドウで、次のように入力して実行して**ChangeFeedConsole**を起動します。

   ```sh
   dotnet run
   ```

1. **3番目の**ターミナル ウィンドウで、**ChangeFeedFunctions**フォルダーに移動します。

1. **3番目の**ターミナル ウィンドウで、次のように入力して実行してAzure Functionsを起動します。

   ```sh
   func host start
   ```

   > If prompted, select **Allow access**メッセージが表示されたら、**Allow access**を選択します。

   > データは、DataGenerator > CartContainer > ChangeFeedConsole > CartContainerByState > MaterializedViewFunction > StateSalesの順で連携されます。

1. データが生成されているときは**最初の**ウィンドウにアスタリスクが表示され、**2番目**と **3番目**のウィンドウには関数が実行されていることを示すコンソールメッセージが表示されます。

1. ブラウザーウィンドウを開き、Cosmos DBのデータエクスプローラーに移動します。

1. **StoreDatabase**、**StateSales**の順に展開し、**Items**を選択します。

1. 州別にコンテナーにデータが入力されているのを確認し、項目を選択してデータの内容を表示します。

   ![The Cosmos DB StateSales container is displayed](../media/08-cosmos-state-sales.jpg "Browse the StateSales container items")

1. **最初の**ターミナルウィンドウで、任意のキーを押してデータ生成を停止します。

1. **2番目の**ターミナルウィンドウで、任意のキーを押してデータ移行を停止します

1. **3番目の**ターミナルウィンドウで、コンソールログメッセージが停止するのを待って、関数がデータの処理を終了できるようにします。（数秒）次に、`Ctrl+C`を押して関数の実行を終了します。

## 変更フィードを使用して、Azure Functions経由でEvenet Hubにデータを書き込む

このラボの変更フィードのユースケースの最後の例では、変更データをAzure Event Hubに書き出す簡単な Azure Functionsを記述します。ストリープロセッサを使用して、Power BIで使用できるリアルタイムのデータ出力を作成し、eコマースダッシュボードを構築します。

### Power BI アカウントを作成する (省略可能)

This step is optional, if you do not wish to follow the lab to creating the dashboard you can skip it
この手順は省略可能です。ラボに従ってダッシュボードを作成しない場合は、スキップできます。

> Power BI アカウントにサインアップするには、[Power BIサイト](https://powerbi.microsoft.com/)にアクセスし、 **無料で試す**を選択します。

1. **CosmosDB**ログインしたら、**CosmosDB**という名前の新しいワークスペースを作成します。

### Azure Event Hubsの接続情報を取得する

1. [Azure Portal](https://portal.azure.com)に切り替えます。

1. ポータルの左側で**リソースグループ**リンクを選択します。

   ![Resource Groups is highlighted](../media/08-select-resource-groups.jpg "Browse to resource groups")

1. **リソースグループ**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

   ![The lab resource group is highlighted](../media/08-cosmos-in-resources.jpg "Select your lab resource group")

1. **cosmoslabs**リソースブレードで、**Event Hubs名前空間**を選択します。

   ![The lab Event Hub is highlighted](../media/08-cosmos-select-hub.jpg "Select the lab Event Hub resource")

1. **Event Hubs名前空間**ブレードで、**設定**の下の**共有アクセスポリシー**を選択します。

1. **共有アクセスポリシー**ブレードで、**RootManageSharedAccessKey**ポリシーを選択します。

1. 表示されるパネルで、**接続文字列 – 主キー**の値をコピーし、このラボの後半で使用するために保存します。

   ![The Event Hub Keys are highlighted](../media/08-event-hub-keys.jpg "Copy and save the connection string for later use")

## Azure Stream Analyticsジョブの出力を作成する

この手順は省略可能です。Power BIに接続してEvent Hubを視覚化しない場合には、スキップできます。

1. ブラウザで**cosmoslabs**ブレードに戻ります。

1. **cosmoslabs**リソースで**CartStreanProcessor**を見つけて選択します。

   ![Stream Analytics is highlighted](../media/08-select-stream-processor.jpg "Select the stream analytics resource")

1. **CartStreanProcessor**の概要画面で、左側のメニューから**出力**を選択します。

   ![The Stream Analytics resource overview blade is displayed](../media/08-stream-processor-output.jpg "Review the overview blade")

1. **出力**ページの上部にある **+出力を追加する**を選択し、**Power BI**を選択します。

   ![Power BI is highlighted](../media/08-add-power-bi.jpg "Choose Power BI")

1. 表示されるウィンドウで、次のデータを入力します。

   - _出力エイリアス_ : `averagePriceOutput`

   - `サブスクリプションからPower BIを選択する`を選択します。

   - _グループワークスペース_ : `CosmosDB`（もしくは作成済みの任意のPower BIのワークスペース）

   - _認証モード_ : `ユーザートークン`

   - _データセット名_ : `averagePrice`

   - _テーブル名_ : `averagePrice`

　 - _接続を承認する_ で`承認`ボタンが表示された場合は、承認します。

   - **保存**を選択します。

   ![The New output dialog is displayed](../media/08-adding-output.jpg "Set the values and select Save")

1. 前の手順を繰り返して、2番目の出力を追加します。

   - _出力エイリアス_ : `incomingRevenueOutput`

   - `サブスクリプションからPower BIを選択する`を選択します。

   - _グループワークスペース_ : `CosmosDB`（もしくは作成済みの任意のPower BIのワークスペース）

   - _認証モード_ : `ユーザートークン`

   - _データセット名_ : `incomingRevenue`

   - _テーブル名_ : `incomingRevenue`

　 - _接続を承認する_ で`承認`ボタンが表示された場合は、承認します。

   - **保存**を選択します。

1. Repeat the previous step to add a third output前の手順を繰り返して3番目の出力を追加します。

   - _出力エイリアス_ : `top5Output`

   - `サブスクリプションからPower BIを選択する`を選択します。

   - _グループワークスペース_ : `CosmosDB`（もしくは作成済みの任意のPower BIのワークスペース）

   - _認証モード_ : `ユーザートークン`

   - _データセット名_ : `top5`

   - _テーブル名_ : `top5`

　 - _接続を承認する_ で`承認`ボタンが表示された場合は、承認します。

   - **保存**を選択します。

1. 前の手順を繰り返して4番目の出力を追加します。

   - _出力エイリアス_ : `uniqueVisitorCountOutput`

   - `サブスクリプションからPower BIを選択する`を選択します。

   - _グループワークスペース_ : `CosmosDB`（もしくは作成済みの任意のPower BIのワークスペース）

   - _認証モード_ : `ユーザートークン`

   - _データセット名_ : `uniqueVisitorCount`

   - _テーブル名_ : `uniqueVisitorCount`

　 - _接続を承認する_ で`承認`ボタンが表示された場合は、承認します。

   - **保存**を選択します。

1. これらの手順を完了すると**出力**ブレードは次のようになります。

   ![The Outputs Blade is displayed with four outputs](../media/08-outputs-blade.jpg "You should see four outputs now")

### Event Hubsにデータを書き込む関数を作成する

変更データを新しいEvent Hubにリアルタイムで書き込む関数を作成します。簡単に作成できることができます。

1. ターミナルウィンドウを開き、**ChangeFeedFunctions**フォルダーに移動します。

1. 次のコマンドを入力して実行し、新しい関数を作成します。

   ```sh
   func new
   ```

   1. メッセージが表示されたら、_template_ として**CosmosDBTrigger**を選択します。

   1. メッセージが表示されたら、_name_ として`AnalyticsFunction`を入力します。

1. 次のコマンドを入力して実行し、[Microsoft Azure Event Hubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) NuGetパッケージを追加します。

   ```sh
   dotnet add package Microsoft.Azure.EventHubs --version 4.3.0
   ```

1. **AnalyticsFunction.cs**ファイルをエディターで開きます。

1. 次の using ステートメントを**AnalyticsFunction.cs**ファイルの先頭に追加します。

   ```csharp
   using Microsoft.Azure.EventHubs;
   using System.Threading.Tasks;
   using System.Text;
   ```

1. **Run**関数のシグネチャを以下のように設定します。

   - **databaseName** to `StoreDatabase`
   - **collectionName** to `CartContainer`
   - **ConnectionStringSetting** to `DBConnection`
   - **LeaseCollectionName** to `analyticsLeases`.

1.  **ConnectionStringSetting**と**LeaseCollectionName**の間に次の行を追加します。

   ```csharp
   CreateLeaseCollectionIfNotExists = true,
   ```

1. **Run**関数を`async Task`に変更します。コードは次のようになります。

   ```csharp
   using System;
   using System.Collections.Generic;
   using Microsoft.Azure.Documents;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Host;
   using Microsoft.Extensions.Logging;
   using Microsoft.Azure.EventHubs;
   using System.Threading.Tasks;
   using System.Text;

   namespace ChangeFeedFunctions
   {
       public static class AnalyticsFunction
       {
           [FunctionName("AnalyticsFunction")]
           public static async Task Run([CosmosDBTrigger(
               databaseName: "StoreDatabase",
               collectionName: "CartContainer",
               ConnectionStringSetting = "DBConnection",
               CreateLeaseCollectionIfNotExists = true,
               LeaseCollectionName = "analyticsLeases")]IReadOnlyList<Document> input, ILogger log)
           {
               if (input != null && input.Count > 0)
               {
                   log.LogInformation("Documents modified " + input.Count);
                   log.LogInformation("First document Id " + input[0].Id);
               }
           }
       }
   }
   ```

1. クラスの先頭に、次の構成パラメーターを追加します。

   ```csharp
   private static readonly string _eventHubConnection = "<event-hub-connection>";
   private static readonly string _eventHubName = "carteventhub";
   ```

1. **\_eventHubConnection**の値を前の手順で確認した**C接続文字列 – 主キー**に置き換えます。

1. 2つのログ出力行を次のコード行に置き換えて、**EventHubClient**を作成します。

   ```csharp
   var sbEventHubConnection = new EventHubsConnectionStringBuilder(_eventHubConnection){
       EntityPath = _eventHubName
   };

   var eventHubClient = EventHubClient.CreateFromConnectionString(sbEventHubConnection.ToString());

   //todo: Next steps here
   ```

1. 変更されたドキュメントごとに、データをEvent Hubに書き込みます。JSONデータを想定するようにイベントハブを構成しているので、ここで行う処理はほとんどありません。`//todo: Next steps here`を次のコード行で置き換えます。

   ```csharp
   var tasks = new List<Task>();

   foreach (var doc in input)
   {
       var json = doc.ToString();

       var eventData = new EventData(Encoding.UTF8.GetBytes(json));

       log.LogInformation("Writing to Event Hub");
       tasks.Add(eventHubClient.SendAsync(eventData));
   }

   await Task.WhenAll(tasks);
   ```

1. 最終的に**AnalyticsFunction**は次のようになります。

   ```csharp
   using System.Collections.Generic;
   using Microsoft.Azure.Documents;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Host;
   using Microsoft.Extensions.Logging;
   using Microsoft.Azure.EventHubs;
   using System.Threading.Tasks;
   using System.Text;

   namespace ChangeFeedFunctions
   {
       public static class AnalyticsFunction
       {
           private static readonly string _eventHubConnection = "<your-connection-string>";
           private static readonly string _eventHubName = "carteventhub";

           [FunctionName("AnalyticsFunction")]
           public static async Task Run([CosmosDBTrigger(
               databaseName: "StoreDatabase",
               collectionName: "CartContainer",
               ConnectionStringSetting = "DBConnection",
               CreateLeaseCollectionIfNotExists = true,
               LeaseCollectionName = "analyticsLeases")]IReadOnlyList<Document> input, ILogger log)
           {
               if (input != null && input.Count > 0)
               {
                   var sbEventHubConnection = new EventHubsConnectionStringBuilder(_eventHubConnection)
                   {
                       EntityPath = _eventHubName
                   };

                   var eventHubClient = EventHubClient.CreateFromConnectionString(sbEventHubConnection.ToString());

                   var tasks = new List<Task>();

                   foreach (var doc in input)
                   {
                       var json = doc.ToString();

                       var eventData = new EventData(Encoding.UTF8.GetBytes(json));

                       log.LogInformation("Writing to Event Hub");
                       tasks.Add(eventHubClient.SendAsync(eventData));
                   }

                   await Task.WhenAll(tasks);
               }
           }
       }
   }
   ```

### AnalyticsFunctionをテストするためにPower BIダッシュボードを作成する

1. もう一度、3つのターミナル ウィンドウを開きます。

1. **最初の**ターミナルウィンドウで、**DataGenerator**フォルダに移動します。


1. **最初の**ターミナルウィンドウで以下を入力して実行することにより、**DataGenerator**を起動します。

   ```sh
   dotnet run
   ```

1. **2番目**のターミナルウィンドウで、**ChangeFeedConsole**フォルダーに移動します。

1.**2番目の**ターミナルウィンドウで、次のように入力して実行して**ChangeFeedConsole**を起動します。

   ```sh
   dotnet run
   ```

1. **3番目の**ターミナル ウィンドウで、**ChangeFeedFunctions**フォルダーに移動します。

1. **3番目の**ターミナル ウィンドウで、次のように入力して実行してAzure Functionsを起動します。

   ```sh
   func host start
   ```

### _残りの手順はPower BIでEvent Hubの出力データを視覚化しない場合にはスキップできます_

1. 次の手順に進む前に、データ ジェネレーターが実行されていること、および Azure 関数とコンソール変更プロセッサが起動していることを確認します。

1. **CartStreamProcessor**の概要画面に戻り、上部にある**開始**ボタンを選択してプロセッサを起動します。プロンプトが表示されたら、今すぐ出力を開始することを選択します。プロセッサの起動には数分かかる場合があります。

> [!TIP]
> Stream Analyticsジョブの開始に失敗した場合は、Event Hubsへの接続不良が原因である可能性があります。これを修正するには、Stream Analyticジョブの入力に移動し、Service Bus名前空間とイベンハブ名をメモしてから、Event Hubsへの`cartInput`接続を削除して再作成します。

> Stream Analyticsジョブの起動に失敗した際にクエリのエラーが出力された場合は、左側の**クエリ**を選択し、表示された内容を以下に置き換えて保存して、再度Stream Analyticsジョブを開始してください。
   ```
   /*TOP 5*/ WITH Counter AS ( SELECT Item, Price, Action, COUNT(*) AS
      countEvents FROM cartinput WHERE Action = 'Purchased' GROUP BY Item, Price, Action,
      TumblingWindow(second,300) ), top5 AS ( SELECT DISTINCT CollectTop(5)  OVER
      (ORDER BY countEvents) AS topEvent FROM Counter GROUP BY TumblingWindow(second,300) ),
      arrayselect AS  ( SELECT arrayElement.ArrayValue FROM top5 CROSS APPLY
      GetArrayElements(top5.topevent) AS arrayElement )  SELECT arrayvalue.value.item,
      arrayvalue.value.price, arrayvalue.value.countEvents INTO top5Output FROM
      arrayselect /*REVENUE*/ SELECT System.TimeStamp AS Time, SUM(Price) INTO
      incomingRevenueOutput FROM cartinput WHERE Action = 'Purchased' GROUP BY
      TumblingWindow(minute, 5) /*UNIQUE VISITORS*/ SELECT System.TimeStamp AS Time,
      COUNT(DISTINCT CartID) as uniqueVisitors INTO uniqueVisitorCountOutput FROM cartinput
      GROUP BY TumblingWindow(second, 30)  /*AVERAGE PRICE*/      SELECT System.TimeStamp
      AS Time, Action, AVG(Price)   INTO averagePriceOutput   FROM cartinput   GROUP BY
      Action, TumblingWindow(second,30) 
   ```

   ![The start link is highlighted](../media/08-start-processor.jpg "Start the stream analytics job")

   > プロセッサが起動するのを待ってから続行します。

1. ブラウザーを開き、[Power BIサイト](https://powerbi.microsoft.com/)に移動します。

1. サインインし、左側のセクションから**CosmosDB**を選択します。

   ![The Power BI portal is displayed](../media/08-power-bi.jpg "Open the PowerBI website")

1. 画面の左上にある **+新規**を選択し、**ダッシュボード**を選択してダッシュボードに任意の _名前_ を付けます。

1. **ダッシュボード**画面で、上部から**編集**を選択し、**タイルの追加**を選択します。

   ![Add Tile link is highlighted](../media/08-power-bi-add-title.jpg "Add a new tile")

1. **カスタムストリーミングデータ**を選択し、**次へ**をクリックします

   ![The real-time data streaming tile is highlighted.](../media/08-pbi-custom-streaming-data.jpg "Add a new stream data item")

1. **カスタムストリーミングデータの追加**ウィンドウから **averagePrice**を選択して**次へ**を選択します。

   ![averagePrice is highlighted](../media/08-add-averageprice-pbi.jpg "Select averagePrice")

1. _ビジュアラゼーションの種類_ から **集横棒グラフ**を選択します。

   - _軸_ で**値の追加**を選択し、**Action**を選択します。

   - _値_ で**値の追加**を選択し、**AVG**を選択します。

   - **次へ**を選択します。

   ![The settings of the tile are highlighted](../media/08-power-bi-first-tile.jpg "Configure the tile")

   - 名前を`Average Price`として、**適用**を選択します。

1. 同様に、残りの 3 つの入力のタイルを追加します。

   - **incomingRevenue**では**軸**は`Time`に、**値**は`SUM`に設定した**折れ線グラフ**を選択します。**表示する時間枠**を30分以上に設定します。

   - **uniqueVisitors**では**フィールド**に`uniqueVisitors`が設定された**カード**を選択します。

   - **top5**では、**軸**を`item`に、**値**を`countEvents`に設定した**集合縦棒グラフ**を選択します。

1. 完了すると、下の画像のようなダッシュボードがリアルタイムで更新されます。

   ![The Final Power BI Dashboard is displayed with real-time data flowing](../media/08-power-bi-dashboard.jpg "Review the new dashboard")

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。
