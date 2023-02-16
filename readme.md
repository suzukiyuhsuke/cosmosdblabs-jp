# Azure Cosmos DB Workshop

## Two Day Suggested Schedule

- [Sample Schedule](./decks/CosmosDBWorkshopSchedule2019.docx)

## Deep-Dive Powerpoint Decks(English)

- [Overview, Value Proposition & Use Cases](./decks/Overview-Value-Proposition-Use-Cases.pptx)
- [Resource Model](./decks/Resource-Model.pptx)
- [Request Units & Billing](./decks/Request-Units-Billing.pptx)
- [Data Modeling](./decks/Data-Modeling.pptx)
- [Partitioning](./decks/Partitioning.pptx)
- [SQL API Query](./decks/SQL-API-Query.pptx)
- [Server Side Programming](./decks/Server-Side-Programming.pptx)
- [Troubleshooting](./decks/Troubleshooting.pptx)
- [Concurrency](./decks/Concurrency.pptx)
- [Change Feed](./decks/Change-Feed.pptx)
- [Global Distribution](./decks/Global-Distribution.pptx)
- [Security](./decks/Security.pptx)

## Deep-Dive パワーポイント資料(日本語)
- [Azure Cosmos DB概要：Overview, Value Proposition & Use Cases](./decks/Overview-Value-Proposition-Use-Cases_JP.pptx)
- [リソースモデル：Resource Model](./decks/Resource-Model_JP.pptx)
- [要求ユニットと請求：Request Units & Billing](./decks/Request-Units-Billing_JP.pptx)
- [データモデリング：Data Modeling](./decks/Data-Modeling_JP.pptx)
- [パーティショニング：Partitioning](./decks/Partitioning.pptx)*未訳*
- [SQL API クエリ：SQL API Query](./decks/SQL-API-Query.pptx)*未訳*
- [サーバーサイドプログラミング：Server Side Programming](./decks/Server-Side-Programming.pptx)*未訳*
- [トラブルシューティング：Troubleshooting](./decks/Troubleshooting.pptx)*未訳*
- [整合性：Concurrency](./decks/Concurrency.pptx)*未訳*
- [変更フィード：Change Feed](./decks/Change-Feed.pptx)*未訳*
- [グローバル分散：Global Distribution](./decks/Global-Distribution.pptx)*未訳*
- [セキュリティ：Security](./decks/Security.pptx)*未訳*

## References

- [Use-Case cheat sheet (1-pager)](./decks/1Pager-Use-Cases.pptx)

In addition to the above workshop decks, we have hands-on labs. We have labs available for our .NET sdk and Java sdk below:

---

## Core (SQL) API

### .NET (V3) Labs

#### .NET Lab 前提条件

このラボを開始する前に、以下の条件を満たすローカルPCを準備してください。

##### オペレーションシステム

- 64 ビット の Windows 10 もしくは windows11 

##### Software

| Software                                    | Download Link                                                |
| ------------------------------------------- | ------------------------------------------------------------ |
| Git                                         | [/git-scm.com/downloads](https://git-scm.com/downloads)      |
| .NET Core 3.1(以上の) SDK <sup>1</sup> | [/download.microsoft.com/dotnet-sdk-3.1](https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-3.1.401-windows-x64-installer) |
| Visual Studio Code                          | [/code.visualstudio.com/download](https://go.microsoft.com/fwlink/?Linkid=852157) |

#### .NET Lab Guides

*It is recommended to complete the labs in the order specified below:*

- [事前準備：アカウントのセットアップ](dotnet/labs/00-account_setup.md)
- [ラボ1：.NET SDK を使用したパーティション分割されたコンテナーの作成](dotnet/labs/01-creating_partitioned_collection.md)
- [ラボ2：ADFを使用してCosmos DBにデータをインポートする](dotnet/labs/02-load_data_with_adf.md)
- [ラボ3：Azure Cosmos DBでのクエリ](dotnet/labs/03-querying_in_azure_cosmosdb.md)
- [ラボ4：Azure Cosmos DBでのインデックス作成](dotnet/labs/04-indexing_in_cosmosdb.md)
- [ラボ5：.NETコンソールアプリの作成](dotnet/labs/05-build_net_app.md)*未訳*
- [ラボ6：複数のドキュメントにまたがったトランザクション](dotnet/labs/06-multi-document-transactions.md)
- [ラボ7：トランザクションの継続](dotnet/labs/07-transactions-with-continuation.md)
- [ラボ8：変更フィードの概要](dotnet/labs/08-change_feed_with_azure_functions.md)
- [ラボ9：パフォーマンスのトラブルシューティング](dotnet/labs/09-troubleshooting-performance.md)
- [ラボ10：オプティミスティックな同時実行制御](dotnet/labs/10-concurrency-control.md)
- [事後作業：クリーンアップ](dotnet/labs/11-cleaning_up.md)

#### Notes

1. If you already have .NET Core installed on your local machine, you should check the version of your .NET Core installation using the ``dotnet --version`` command.

---

### Java Labs

#### Java Lab Prerequisites

Prior to starting these labs, you must have the following operating system and software configured on your local machine:

##### Operating System

- 64-bit Windows 10 Operating System
  - [download](https://www.microsoft.com/windows/get-windows-10)

##### Software

| Software | Download Link |
| --- | --- |
| Git | [/git-scm.com/downloads](https://git-scm.com/downloads)
Java 8 JDK (or greater) | [/jdk8-downloads](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) |
Java 8 JRE (or greater) | [/jre8-downloads](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) |
| Visual Studio Code | [/code.visualstudio.com/download](https://go.microsoft.com/fwlink/?Linkid=852157) |
| Java Extension Pack (if using VS Code) | [/vscode-java-pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) |
| Maven | [/maven.apache.org/](https://maven.apache.org/) |

#### Java Lab Guides

*It is recommended to complete the labs in the order specified below:*

- [Pre-lab: Creating an Azure Cosmos DB account](java/labs/00-account_setup.md)
- [Lab 1: Creating a container in Azure Cosmos DB](java/labs/01-creating_partitioned_collection.md)
- [Lab 2: Importing Data into Azure Cosmos DB with Azure Data Factory](java/labs/02-load_data_with_adf.md)
- [Lab 3: Querying in Azure Cosmos DB](java/labs/03-querying_in_azure_cosmosdb.md)
- [Lab 4: Indexing in Azure Cosmos DB](java/labs/04-indexing_in_cosmosdb.md)
- [Lab 5: Building a Java Console App on Azure Cosmos DB](java/labs/05-build_java_app.md)
- [Lab 6: Multi-Document Transactions in Azure Cosmos DB](java/labs/06-multi-document-transactions.md)
- [Lab 7: Transactional Continuation in Azure Cosmos DB](java/labs/07-transactions-with-continuation.md)
- [Lab 8: Intro to Azure Cosmos DB Change Feed](java/labs/08-change_feed_with_azure_functions.md)
- [Lab 9: Troubleshooting Performance in Azure Cosmos DB](java/labs/09-troubleshooting-performance.md)
- [Lab 10: Optimistic Concurrency Control in Azure Cosmos DB](java/labs/10-concurrency-control.md)
- [Post-lab: Cleaning Up](java/labs/11-cleaning_up.md)

#### Notes

1. When installing the Java 11 SDK or higher, this is bundled with a Java Runtime Environment (JRE). Make sure the JRE path (e.g: C:\Program Files\Java\jdk-11.0.2\bin\) is present at the top of your Path variable in System variables.
1. If you already have Java installed on your local machine, you should check the version of your Java Runtime Environment (JRE) installation using the ``java -version`` command.
1. If using a version of Java greater than version 8, some projects may not compile (for example the benchmarking application).

---

## Gremlin API

### Workshop Decks

- [Introduction](./decks/Gremlin/GraphWorkshop_1_Introduction.pptx)
- [Graph Modeling](./decks/Gremlin/GraphWorkshop_2_GraphModeling.pptx)
- [Design Principles](./decks/Gremlin/GraphWorkshop_3_GraphDesignPrinciples.pptx)

---

## Cassandra API

### Workshop Decks

- [Introduction](./decks/Cassandra/Cassandra_Workshop_Introduction.pptx)

### Cassandra Labs

*It is recommended to complete the labs in the order specified below:*

- [Pre-lab: Creating an Azure Cosmos DB Cassandra API Account](cassandra/labs/00-account_setup.md)
- [Lab 1: Load Data with Databricks](cassandra/labs/01-load_data_with_databricks.md)
- [Lab 2: Query Data with CQLSH](cassandra/labs/02-querying_with_cqlsh.md)
- [Lab 3: Implementing Retry and Failover](cassandra/labs/03-implementing_retry_and_failover.md)
- [Lab 4: Change Feed with Spring Data](cassandra/labs/04-change_feed_with_spring_data.md)
- [Lab 5: Cleaning Up](cassandra/labs/07-cleaning_up.md)

---

## Appendix: Stickers

Adobe Illustrator files for printing cosmic stickers (e.g. stickermule):

- [2x2 inch black circle](./stickers/2x2-circle-template-CosmosBlack.ai)
- [2x2 inch clear circle](./stickers/2x2-clear-sticker-template-CosmosClear.ai)
- [Die-cut color logo](./stickers/cosmos-die-cut-sticker-template-v2.ai)
