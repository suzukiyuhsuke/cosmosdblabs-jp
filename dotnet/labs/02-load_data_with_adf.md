 # ADFを使用してCosmos DBにデータをインポートする

このラボでは、Azure に組み込まれているツールを使用して、既存のデータ セットから Azure Cosmos DB コンテナーを設定します。インポート後、Azure portal を使用してインポートしたデータを表示します。

> ラボ コンテンツのセットアップをまだ完了していない場合は、このラボを開始する前に、 [アカウントのセットアップ](00-account_setup.md) を実施してください。これにより、ラボ全体で使用する Azure Cosmos DB データベースとコンテナーが作成されます。また、**Azure データ ファクトリ**(ADF) リソースを使用して、既存のデータをコンテナーにインポートすることもできます。

## Azure Cosmos DB データベースとコンテナーを作成する

Azure Cosmos DB アカウント内にデータベースとコンテナーを作成します。

1. [Azure Portal](https://portal.azure.com)に移動します。

1. ポータルの左側で**Resource groups**リンクを選択します。

    ![Resource groups is highlighted](../media/03-resource_groups.jpg "Select the Resource Groups")

1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

    ![The cosmoslabs resource group is highlighted](../media/03-lab_resource_group.jpg "Select the cosmoslabs resource group")
　
    > [アカウントのセットアップ](00-account_setup.md)で異なるリソースグループ名を指定した場合は、そのリソースグループ名を使用してください。

1. **cosmoslabs**ブレードで、作成されている**Azure Cosmos DB**を選択します。

    ![The Cosmos DB resource is highlighted](../media/03-cosmos_resource.jpg "Select the cosmoslabs resource")

1. **Azure Cosmos DB**ブレードで、左側にある**概要**リンクを見つけて選択します。右側のペインで上部にある**コンテナーの追加**ボタンを選択します。

    ![Add container link is highlighted](../media/03-add_collection.jpg "Add a new container")

1. **コンテナーの追加**ポップアップで、以下の操作を実行します。　

    1. **Database id**フィールドで**Create new**オプションを選択し、**ImportDatabase**を`Database id`に指定します。

    2. **Share throughput across containers**オプションをオフにします。

        > **Share throughput across containers**をオンにすると、そのデータベースに属するすべてのコンテナー間でスループットが共有されます。**Azure Cosmos DB**データベース内には、スループットを共有するコンテナーのセットと、専用のスループットを持つコンテナーを含めることができます。

    3. **Container Id**フィールドには、**FoodCollection**を入力します。

    4. **Partition key**フィールドには``/foodGroup``を入力します。

    5. **Throughput**フィールドには、``11000``を入力します。 *註: データのインポート後に、この値は400RU/sに変更します。*

    6. **OK**ボタンを選択します。

1. 新しい**データベース**と**コンテナー**の作成が完了するのを待って、次の手順に進みます。

## ラボデータをコンテナーにインポートする

**Azure Data Factory (ADF)**を使用して、**nutrition.json**ファイルに格納されているJSONデータをAzure Blob Storageからインポートします。

このセクションの手順１から４を実行しなくても、セットアップで作成済みのimportNutirationDataから始まるData Factoryリソースを使用して、手順４以降を実施することができます。

1. Azureポータルの左側で、**Resource groups**を選択します。

    > ADFを利用してCosmos DBにデータをコピーする方法の詳細については、[ADFのドキュメント](https://docs.microsoft.com/azure/data-factory/connector-azure-cosmos-db)を参照してください。

    ![Resource groups link is highlighted](../media/03-resource_groups.jpg "Select Resource Groups")

1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

1. すでにAzure Data Factoryリソースが存在する場合は、手順４に進んでください。そうでない場合は、**追加**を選択して新しいリソースを作成します。

    ![A data factory resource is highlighted](../media/03-adf-isntance.png "Review if you have data factory already")

    ![Select Add in the nav bar](../media/03-add_adf.jpg "Add a new resource")

   - **データファクトリー**を検索して選択します。 
   - 新しい**データファクトリー**を作成します。データファクトリの名称は、**importnutritiondata**で始まるユニークな番号を付与したものとして、このラボを実行するサブスクリプションに紐づけます。既存の**cosmoslabs**リソースグループと**V2**バージョンを選択するようにします。 
   - **East US**リージョンを選択します。（任意のリージョンでも問題ありません）**Gitを有効にする**は選択しないでください。(デフォルトでチェックされている場合があります) 
   - **確認と作成**を選択します。

        ![The new data factory dialog is displayed](../media/03-adf_selections.jpg "Add a new Data Factory resource")

1. 作成後、新しく作成したデータ ファクトリを開きます。**スタジオの起動**を選択すると、ADF が起動します。

    ![The overview blade is displayed for ADF](../media/03-adf_author&monitor.jpg "Select Author and Monitor link")

1. スタジオで**取り込み**を選択します。

   - Azure BLOB ストレージ上のソース JSON ファイルから Cosmos DB の SQL API 内のデータベースへのデータの 1 回限りのコピーには、ADF を使用します。ADF は、Cosmos DB から他のデータ ストアへのより頻繁なデータ転送にも使用できます。

    ![The main workspace page is displayed for ADF](../media/03-adf_copydata.jpg "Select the Copy Data activity")

1. プロパティで、以下の操作をします。
    1. **タスクの種類**で``組み込みコピータスク``を選択します。
    1. **タスクの実行またはタスクのスケジュール**で**今すぐ1回実行する**を選択します。
    1. **次へ**ボタンを選択します。

   ![The copy data activity properties dialog is displayed](../media/03-adf_properties.jpg "Enter a task name and the schedule")

1. ソースデータストアで、以下の操作をします。
    1. **+新しい接続**ボタンを選択します。
    1. ポップアップで**Azure BLOBストレージ**を選択し、**続行**を選択します。

    ![Create new connection link is highlighted](../media/03-adf_blob.jpg "Create a new connection")

    ![Azure Blog Storage is highlighted](../media/03-adf_blob2.jpg "Select the Azure Blob Storage connection type")

1. 新しい接続の名前に**NutritionJson**を入力し、認証の種類で**SAS URI**を選択します。**SAS URL**には以下の値を使用してください。

     `https://cosmoslabsstorageaccount.blob.core.windows.net/nutrition-data?si=container-list-read-policy&spr=https&sv=2021-06-08&sr=c&sig=jGrmrokYikbgbuW9we2am%2BwAq%2BC%2BxfZcPYswOeSQpAU%3D`

    ![The New linked service dialog is displayed](../media/03-adf_connecttoblob.jpg "Enter the SAS url in the dialog")

1. **作成**を選択します。
1. **次へ**を選択します。
1. **ファイルまたはフォルダー**テキストボックスにフォルダー名``nutirion-data``を入力し、**参照**を選択します。続いて、開いたダイアログで**NutritionData.json** ファイルを選択します。

    ![The nutritiiondata folder is displayed](../media/03-adf_choosestudents.jpg "Select the NutritionData.json file")

1. **再帰的に実行** と **バイナリコピー** のチェックを外します。 他のフィールドは空欄して、**次へ**を選択します。

    ![The input file or folder dialog is displayed](../media/03-adf_source_next.jpg "Ensure all other fields are empty, select next")

1. ファイル形式は**JSON**を選択します。他は何も変更せずに**次へ**を選択します。

    !["The file format settings dialog is displayed"](../media/03-adf_source_dataset_format.jpg "Ensure JSON format is selected, then select Next")

1. これで、NutirationData.jsonをソースとするBLOBストレージコンテナーへの接続が正常に作成されました。
1. **コピー先データストア**で**新しい接続の作成**を選択し、ポップアップ上で**Azure Cosmos DB for NoSQL**を選択してCosmos DBへの接続を追加します。

    !["The New Linked Service dialog is displayed"](../media/03-adf_selecttarget.jpg "Select the Azure Cosmos DB service type")

1. 名前を**targetcosmosdb**と入力し、アカウントの選択でこのラボで使用しているサブスクリプションとCosmos DBアカウントを選択します。データベース名は、**ImportData**を選択します。

    !["The linked service configuration dialog is displayed"](../media/03-adf_selecttargetdb.jpg "Select the ImportDatabase database")

1. **作成**を選択します。

1. 作成した**targetcosmosdb**を接続に指定します。

    !["The destination data source dialog is displayed"](../media/03-adf_destconnectionnext.jpg "Select your recently created data source")

1. ターゲットのドロップダウンリストで**FoodCollection**を選択します。**次へ**を選択します。

    !["The table mapping dialog is displayed"](../media/03-adf_correcttable.jpg "Select the FoodCollection container")

1. `設定`は何も変更をせずに、**次へ**を選択します。

    !["The settings dialog is displayed"](../media/03-adf_settings.jpg "Review the dialog, select next")

1. 概要を確認して**次へ**を選択します。パイプラインのデプロイが完了したら、**モニター**を選択します。

    !["The pipeline runs are displayed"](../media/03-adf_progress.jpg "Notice the pipeline is In progress")

1. パイプラインの実行には数分かかります。数分待って、**最新の情報に更新**を選択し、完了後に状態が**成功**となっていることを確認します。

    !["The pipeline runs are displayed"](../media/03-adf_progress_complete.jpg "The pipeline has succeeded")

1. インポートが完了したら、ADFを閉じることができます。次に、インポートしたデータの検証に進みます。

## インポートしたデータの検証

Azure Cosmos DB データ エクスプローラーを使用すると、ドキュメントを表示し、Azure ポータル内で直接クエリを実行できます。この演習では、データ エクスプローラーを使用して、コンテナーに格納されているデータを表示します。

**データエクスプローラー**の**Items**ビューを使用して、データがコンテナーに正常にインポートされたことを検証します。

1. **Azure Portal** (<http://portal.azure.com>)に戻ります。

1. ポータルの左側で**Resource groups**リンクを選択します。

    ![Resource groups link is highlighted](../media/03-resource_groups.jpg "Select your resource group")

1. **Resource groups**ブレードで、**cosmoslabs**リソースグループを見つけて選択します。

    ![The Lab resource group is highlighted](../media/03-lab_resource_group.jpg "Select the resource group")

1. **cosmoslabs**ブレードで、作成されている**Azure Cosmos DB**を選択します。

    ![The Cosmos DB resource is highlighted](../media/03-cosmos_resource.jpg "Select the Cosmos DB resource")

1. **Azure Cosmos DB**ブレードで、左側にある**データエクスプローラー**リンクを見つけて選択します。

    ![The Data Explorer link was selected and is blade is displayed](../media/03-data_explorer_pane.jpg "Select Data Explorer")

1. 右側に表示された**データエクスプローラー**内で、**ImportDatabase**データベースのノードを展開し、**FoodCollection**コンテナーのノードを展開します。

    ![The Container node is displayed](../media/03-collection_node.jpg "Expand the ImportDatabase node")

1. **FoodCollection**ノード内で、**Scale and Settings**リンクを選択してコンテナーのスループットを表示します。スループットの値を**400 RU/s**に減らします。

    ![Scale and Settings](../media/03-collection-settings.png "Reduce throughput")

1. **FoodCollection**ノード内で、**Items**リンクを選択して、コンテナー内のさまざまなドキュメントのサブセットを表示します。いくつかのドキュメントを選択し、ドキュメントのプロパティと構造を確認します。

    ![Items is highlighted](../media/03-documents.jpg "Select Items")

    ![An Example document is displayed](../media/03-example_document.jpg "Select a document")

> 以降のラボを実施しない場合は、[Removing Lab Assets](11-cleaning_up.md) の手順に従ってすべてのラボリソースを削除します。