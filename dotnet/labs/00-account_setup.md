# アカウントの設定

このラボでは、Cosmos DB ラボの実行に必要なリソースを使用して Azure サブスクリプションを設定します。これらのラボを一度に実行した場合の推定コストは、~$100 USD です。

## 前提条件

- Azure有料サブスクリプション
- [Azure PowerShell Module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps)

## ラボコンテンツのセットアップ

1. セットアップを開始するには、Git の複製を行うか、これらの手順を含むリポジトリを [Github](https://github.com/AzureCosmosDB/labs) からダウンロードします。

2. Windows PowerShellを起動します。
3. ダウンロードしたリポジトリのコピーを含むフォルダに移動します。
4. リポジトリ内で **dotnet\setup** フォルダに移動します。

   ```powershell
   cd .\dotnet\setup\
   ```

5. 現在のセッションでPowerShellスクリプトを実行できるようにするには、次のように入力します。

   ```powershell
   Set-ExecutionPolicy Unrestricted -Scope Process
   ```

   > この設定は現在のPowerShellウィンドウ内でのみ適用されます。

6. ラボの多くは、準備されたスターターコードを利用します。スターターコードを使用するために、labCodeSetup.ps1を実行すると **Documents** フォルダ内の **CosmosLabs** フォルダにコードが自動的にコピーされます。

   ```powershell
   .\labCodeSetup.ps1
   ```

   > 各ラボのスターターコードは **templates** フォルダにあります。**Documents\CosmosLabs** フォルダ以外のフォルダを使用するためには、これらのコードを任意のフォルダに手動でコピーしてください。

7. Azureのリソースをセットアップするために、Azureアカウントに接続します。

   ```powershell
   Connect-AzAccount
   ```

   or

   ```powershell
   Connect-AzAccount -subscription <subscription id>
   ```

8. labSetup.ps1スクリプトを実行して、ラボ用のAzureリソースを作成します。

   ```powershell
   .\labSetup.ps1 -resourceGroupName 'name'
   ```

   `name` はユニークである可能性が高いものを指定してください。例として、`cosmoslabsXXXXX` など `XXXXX` の部分が乱数となるような形式を推奨します。

   - このスクリプトでは既定で米国西部リージョンが使用されますが、**-location** オプションを指定することで他のリージョンを指定することも可能です。例えば、東日本リージョンを利用する場合には以下のように指定します。
   ```powershell
   .\labSetup.ps1 -resourceGroupName 'name' -location "Japan East"
   ```

   - 指定したリソースグループが既に存在する場合、このスクリプトは失敗します。別の値を選択することもできます。または、このエラーを回避してリソースを作成するには、上記のコマンドに **-orverwriteGroup** オプションを追加します。

9. 一部のAzureリソースでは、セットアップが完了するまでに10分以上かかる場合があるため、スクリプトの実行完了には時間を要します。スクリプトが完了した後、 作成したリソースグループは以下のリソースが自動的にデプロイされた状態になります。

   - Azure CosmosDB Account
   - Stream Analytics Job
   - Azure Data Factory
   - Event Hubs Namespace

## Azureポータルにログイン

1. 新しいブラウザのウィンドウで、 **Azure Portal** (<https://portal.azure.com>)にログインします。

1. ログイン後にAzureポータルのツアーを開始するように求められる場合がありますが、この手順はスキップしても問題ありません。

### アカウント資格情報の取得

.NET SDKでは、Azure Cosmos DB アカウントに接続するために資格情報が必要です。ラボ全体で使用するため、これらの資格情報を収集してメモします。

1. ポータルの左側で **リソースグループ** リンクを選択します。

   ![Resource groups is highlighted](../media/02-resource_groups.jpg "Select resource groups")

1. **リソースグループ** ブレードで、**cosmoslabs** リソースグループを見つけて選択します。

   ![The recently cosmosdb resource group is highlighted](../media/02-lab_resource_group.jpg "Select the CosmosDB resource group")

1. **cosmoslabs** ブレードで作成した **Azure Cosmos DB** アカウントを選択します。

   ![The Cosmos DB resource is highlighted](../media/02-cosmos_resource.jpg "Select the Cosmos DB resource")

1. **Azure Cosmos DB** ブレードで **設定**のセクションから **キー** リンクを選択します。

   ![The Keys pane is highlighted](../media/02-keys_pane.jpg "Select the Keys Pane")

1. **Keys** ウィンドウに表示された **接続文字列**、 **URI** と **プライマリーキー** をメモります。これらの値は、以後のラボで使用します。

   ![The URI, Primary Key and Connection string credentials are highlighted](../media/02-keys.jpg "Copy the URI, primary key and the connection string")
