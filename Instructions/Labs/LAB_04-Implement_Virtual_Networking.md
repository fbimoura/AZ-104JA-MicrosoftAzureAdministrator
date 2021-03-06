---
lab:
    title: '04 - 仮想ネットワークの実装'
    module: 'モジュール 04 - 仮想ネットワーク'
---

# ラボ 04 - 仮想ネットワークの実装

# 受講者用ラボ マニュアル

## ラボ シナリオ

Azure 仮想ネットワークの機能を詳しく調べる必要があります。まず、いくつかの Azure 仮想マシンをホストする仮想ネットワークを Azure に作成することを検討します。ネットワークベースのセグメンテーションを実装しして、それらの仮想マシンを異なるサブネットの仮想ネットワークにデプロイします。また、プライベート IP アドレスとパブリック IP アドレスの使用が進むにつれて変更されないようにする必要もあります。Contoso のセキュリティ要件に準拠するには、インターネットからアクセスできる Azure 仮想マシンのパブリック エンドポイントを保護する必要があります。最後に、仮想ネットワーク内およびインターネットからの Azure 仮想マシンの DNS 名前解決を実装する必要があります。

## 目標

このラボでは、次の内容を学習します。

+ タスク 1: 仮想ネットワークを作成および構成する
+ タスク 2: 仮想マシンを仮想ネットワークにデプロイする
+ タスク 3: Azure VM のプライベート IP アドレスとパブリック IP アドレスを構成する
+ タスク 4: ネットワーク セキュリティ グループを構成する
+ タスク 5: 内部の名前解決用に Azure DNS を構成する
+ タスク 6: 外部の名前解決用に Azure DNS を構成する

## 想定時間: 40分間

## 手順

### エクササイズ 1

#### タスク 1: 仮想ネットワークを作成および構成する

このタスクでは、Azure portal を使用して、複数のサブネットを持つ仮想ネットワークを作成します。

1. [Azure portal](https://portal.azure.com) にログインします。

1. Azure portal で、「**仮想ネットワーク**」 を検索して選択し、「**仮想ネットワーク**」 ブレードで 「**+ 追加**」 をクリック します。

1. 仮想ネットワークを次の設定で作成します (その他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | **az104-04-rg1** という名前の **新しい** リソース グループ |
    | 名前 | **az104-04-vnet1** |
    | リージョン | このラボで使用するサブスクリプションで使用できる Azure リージョンの名前 |
    | IPv4 アドレス空間 | **10.40.0.0/20** |
    | サブネット名 | **subnet0** |
    | サブネット アドレス範囲 | **10.40.0.0/24** |

    >**注意:** 仮想ネットワークがプロビジョニングされるのを待ちます (1 分未満)。

1. 「**仮想ネットワーク**」 ブレードで、「**更新**」 をクリックし、「**az104-04-vnet1**」 をクリックします。

1. 「**az104-04-vnet1** Virtual Network」 ブレードで、「**サブネット**」 をクリックし、「**+ サブネット**」 をクリックします。 

1. サブネットを次の設定で作成します (その他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **subnet1** |
    | アドレス範囲 (CIDR ブロック) | **10.40.1.0/24** |
    | ネットワーク セキュリティ グループ | **なし** |
    | ルート テーブル | **なし** |

#### タスク 2: 仮想マシンを仮想ネットワークにデプロイする

このタスクでは、ARM テンプレートを使用して、Azure 仮想マシンを仮想ネットワークの異なるサブネットにデプロイします。

1. Azure portal で右上にあるアイコンをクリックして **Azure Cloud Shell** を開きます。

1. **Bash** または **PowerShell** のいずれかを選択するように求められた場合、**PowerShell**を選択します。 

    >**注**: **Cloud Shell** を初めて起動し、「 **ストレージがマウントされていません**」というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 をクリックします。 

1. 「クラウド シェル」 ペインのツール バーで 「**ファイルのアップロード/ダウンロード**」 アイコンをクリックし、ドロップダウン メニューで 「**アップロード**」 をクリックして、ファイル **\Allfiles\\Labs\\04\az104-04-vms-template.json** とファイル **\Allfiles\\Labs\\04\az104-04-vms-parameters.json** を Cloud Shell のホーム ディレクトリにアップロードします。

    >**注**: 各ファイルを別々にアップロードすることが必要な場合があります。

1. 「Cloud Shell」 ペインで次のコマンドを実行して、アップロードしたテンプレートとパラメーター ファイルを使用して 2 つの仮想マシンをデプロイします。

   ```pwsh
   $rgName = 'az104-04-rg1'

   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-04-vms-template.json `
      -TemplateParameterFile $HOME/az104-04-vms-parameters.json
   ```

    >**注意**: ARM テンプレートをデプロイするこの方法では、Azure PowerShell を使用します。同等の Azure CLI コマンド **az deployment create** を実行して、同じタスクを実行することもできます (詳細については、[Resource Manager テンプレートおよび Azure CLI を使用したリソースのデプロイ](https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deploy-cli)』を参照してください)。

    >**注**: 次のタスクに進む前に、デプロイが完了するのを待ちます。これには約 2 分かかります。

1. Cloud Shell ペインを閉じます。

#### タスク 3: Azure VM のプライベート IP アドレスとパブリック IP アドレスを構成する

このタスクでは、Azure 仮想マシンのネットワーク インターフェイスに割り当てられたパブリック IP アドレスとプライベート IP アドレスの静的割り当てを構成します。

   >**注**: プライベート IP アドレスとパブリック IP アドレスは、実際にネットワーク インターフェイスに割り当てられ、次に Azure Virtual Machines に接続されますが、Azure VM に割り当てられた IP アドレスを代わりに参照することもよくあります。

1. Azure portal で 「**リソース グループ**」 を検索して選択し、「**リソース グループ**」 ブレードで 「**az104-04-rg1**」 をクリックします。

1. 「**az104-04-rg1** リソース グループ」 ブレードのリソースの一覧で、「**az104-04-vnet1**」 をクリックします。

1. 「**az104-04-vnet1** 仮想ネットワーク」 ブレードで、「**接続デバイス**」 セクションを確認し、仮想ネットワークに接続されている 2 つのネットワーク インターフェイス **az104-04-nic0** と **az104-04-nic1**があることを確認します。

1. 「**az104-04-nic0** をクリックし、「**az104-04-nic0**」 ブレードで 「**IP 構成**」 をクリックします。 

    >**注**: **ipconfig1** が動的プライベート IP アドレスで現在設定されていることを確認します。

1. IP 構成の一覧で、「**ipconfig1**」 をクリックします。

1. 「**ipconfig1**」 ブレードで 「**割り当て**」 を 「**静的**」 に設定し、**IP アドレス**の既定値を **10.40.0.4** のままにします。

1. **Ipconfig1** ブレードの**パブリック IP アドレス設定**セクションで、**関連付け**をクリックしてから、**IP address - Configure required settings** (IP アドレス - 必要な設定を構成してください) をクリックします。 

1. 「**パブリック IP アドレスの選択**」 ブレードで 「**+ 新規作成**」 をクリックし、次の設定で新しいパブリック IP アドレスを作成します。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **az104-04-pip0** |
    | SKU | **Standard** |

1. 「**ipconfig1**」 ブレードに戻り、変更を保存します。

1. 「**az104-04-vnet1**」 ブレードに戻り、前の 6 つの手順を繰り返して、「**az104-04-nic1**」 の 「**ipconfig1**」 の IP アドレス割り当てを 「**静的**」 に変更し、「**az104-04-pip1**」 という名前の新しい標準 SKU パブリック IP アドレスに 「**az104-04-nic1**」 を関連付けます。

1. 「**az104-04-rg1** リソース グループ」 ブレードに戻り、そのリソースの一覧で 「**az104-04-vm0**」 をクリックし、「**az104-04-vm0** 仮想マシン」 ブレードでパブリック IP アドレスのエントリをメモします。

1. 「**az104-04-rg1** リソース グループ」 ブレードに戻り、そのリソースの一覧で 「**az104-04-vm1**」 をクリックし、「**az104-04-vm1** 仮想マシン」 ブレードでパブリック IP アドレスのエントリをメモします。

    >**注**: このラボの最後のタスクで、両方の IP アドレスが必要になります。 


#### タスク 4: ネットワーク セキュリティ グループを構成する

このタスクでは、Azure 仮想マシンへの接続を制限できるように、ネットワーク セキュリティ グループを構成します。

1. Azure portal で 「**az104-04-rg1** リソース グループ」 ブレードに戻り、そのリソースの一覧で 「**az104-04-vm0**」 をクリックします。

1. 「**az104-04-vm0**」 ブレードで 「**接続**」 をクリックし、ドロップダウン メニューで 「**RDP**」 をクリックし、「**RDP で接続**」 ブレードで 「**RDP ファイルのダウンロード**」 をクリックし、プロンプトに従ってリモート デスクトップ セッションを開始します。

1. 接続の試行が失敗します。

    >**注**: これは正常な動作です。既定では、Standard SKU のパブリック IP アドレスが割り当てられたネットワーク インターフェイスは、ネットワーク セキュリティ グループで保護されている必要があるためです。リモート デスクトップ接続を許可するには、インターネットからの受信 RDP トラフィックを明示的に許可するネットワーク セキュリティ グループを作成し、両方の仮想マシンのネットワーク インターフェイスに割り当てます。

1. Azure portal で 「**ネットワーク セキュリティ グループ**」 を検索して選択し、「**ネットワーク セキュリティ グループ**」 ブレードで 「**+ 追加**」 をクリックします。

1. ネットワーク セキュリティ グループを次の設定で作成します。その他の設定は既定値のままにします。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | **az104-04-rg1** |
    | 名前 | **az104-04-nsg01** |
    | リージョン | このラボで他のすべてのリソースをデプロイする Azure リージョンの名前 |

    >**注**: デプロイが完了するまで待ちます。これには約 2 分かかります。

1. 「デプロイ」 ブレードで 「**リソースに移動**」 をクリックして、「**az104-04-nsg01**」 ネットワーク セキュリティ グループ ブレードを開きます。 

1. 「**az104-04-nsg01**」 ネットワーク セキュリティ グループ」 ブレードの 「**設定**」 セクションで、「**受信セキュリティ ルール**」 をクリックします。 

1. 次の設定を使用して受信ルールを追加します。その他の設定は既定値のままにします。

    | 設定 | 値 |
    | --- | --- |
    | ソース | **Any** |
    | ソース ポート範囲 | * |
    | 宛先 | **Any** |
    | 宛先 ポート範囲 | **3389** |
    | プロトコル | **TCP** |
    | アクション | **許可** |
    | 優先度 | **300** |
    | 名前 | **AllowRDPInBound** |

1. 「**az104-04-nsg01** ネットワーク セキュリティ グループ」 ブレードの 「**設定**」 セクションで、「**ネットワーク インターフェイス**」 をクリックし、「**+ 関連付け**」 をクリックします。

1. 「**az104-04-nsg01** ネットワーク セキュリティ グループ」 を 「**az104-04-nic0**」 および 「**az104-04-nic1**」 ネットワーク インターフェイスに関連付けます。

    >**注**: 新しく作成されたネットワーク セキュリティ グループのルールがネットワーク インターフェイス カードに適用されるまで、最大 5 分かかる場合があります。

1. 「**az104-04-vm0** 仮想マシン」 ブレードに戻ります。

    >**注**: これで、ターゲット仮想マシンに正常に接続し、**受講生** ユーザー名と **Pa55w.rd1234** のパスワードを使用してサインインできることを確認します。

1. 「**az104-04-vm0**」 ブレードで、「**接続**」 をクリックして 「**接続**」 をクリックし、ドロップダウン メニューで 「**RDP**」 をクリックし、「**RDP で接続**」 ブレードで 「**RDP ファイルをダウンロード**」 をクリックして、プロンプトに従ってリモート デスクトップ セッションを開始します。

    >**注**: この手順は、Windows コンピューターからリモート デスクトップ経由で接続することを指します。Mac では、Mac App Store からリモート デスクトップ クライアントを使用でき、Linux コンピューターでは、オープンソースの RDP クライアント ソフトウェアを使用できます。

    >**注**: ターゲットの仮想マシンに接続する際は、警告メッセージを無視できます。

1. プロンプトが表示されたら、ユーザー名 「**Student**」 とパスワード 「**Pa55w.rd1234**」 を使用してログインします。

    >**注**: リモート デスクトップ セッションを開いたままにします。これは、次のタスクで必要になります。

#### タスク 5: 内部の名前解決用に Azure DNS を構成する

このタスクでは、Azure プライベート DNS ゾーンを使用して、仮想ネットワーク内で DNS 名前解決を構成します。

1. Azure portal で、「**プライベート DNS ゾーン**」 を検索して選択し、「**プライベート DNS ゾーン**」 ブレードで 「**+ 追加**」 をクリックします。

1. プライベート DNS ゾーンを次の設定で作成します (その他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | **az104-04-rg1** |
    | 名前 | **contoso.org** |

    >**注**: プライベート DNS ゾーンが作成されるのを待ちます。これには約 2 分かかります。

1. 「**リソースに移動**」 をクリックして 「**contoso.org** DNS プライベート ゾーン」 ブレードを開きます。 

1. 「**contoso.org** プライベート DNS ゾーン」 ブレードの 「**設定**」 セクションで、「**仮想ネットワーク リンク**」 をクリックします。

1. 仮想ネットワーク リンクを次の設定で追加します (他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | リンク名 | **az104-04-vnet1-link** |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | 仮想ネットワーク | **az104-04-vnet1** |
    | 自動登録を有効にする | enabled |

    >**注意:** 仮想ネットワーク リンクが作成されるまで待ちます (1 分未満)。

1. **「contoso.org」** プライベート DNS ゾーンブレードのサイド バーで、**「概要」** をクリックします。

1. 「**az104-04-vm0**」 および 「**az104-04-vm1**」 の DNS レコードが、レコード セットの一覧に「**自動登録**」として表示されることを確認 します。

    >**注意:** レコード セットが表示されない場合は、数分待ってからページを更新する必要があります。

1. 「**az104-04-vm0** へのリモート デスクトップ」 セッションに切り替えて、**スタート** ボタンを右クリックし、「**スタート**」 ボタンを右クリックし、表示されたメニューから 「**Windows PowerShell (Admin)**」 をクリックします。

1. Windows PowerShell コンソール ウィンドウで次のコマンドを実行して、新しく作成したプライベート DNS ゾーン内の DNS レコード セット **az104-04-vm1** の内部名前解決をテストします。

   ```pwsh
   nslookup az104-04-vm1.contoso.org
   ```
1. コマンドの出力に、**az104-04-vm1** (**10.40.1.4**) のプライベート IP アドレスが含まれていることを確認します。

#### タスク 6: 外部の名前解決用に Azure DNS を構成する

このタスクでは、Azure パブリック DNS ゾーンを使用して外部 DNS 名前解決を構成します。

1. Web ブラウザーで、https://www.godaddy.com/domains/domain-name-search を新しいタブで開きます。

1. 適当なドメイン名を検索して使用されていないドメイン名を探します。 

1. Azure portal で 「**DNS ゾーン**」を検索して選択し、「**DNS ゾーン**」 ブレードで 「**+ 追加**」 をクリックします。

1. DNS ゾーンを次の設定で作成します (その他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | **az104-04-rg1** |
    | 名前 | このタスクの前半で確認した DNS ドメイン名 |

    >**注**: DNS ゾーンが作成されるのを待ちます。これには約 2 分かかります。 

1. **リソースに移動**をクリックし、新しく作成した DNS ゾーンのブレードを開きます。 

1. DNS ゾーン ブレードで、**+ Record set** (+ レコード セット) をクリックします。

1. レコード セットを次の設定で追加します (その他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **az104-04-vm0** |
    | タイプ | **A** |
    | エイリアス レコード セット | **No** |
    | TTL | **1** |
    | TTL の単位 | **時間** |
    | IP アドレス | このラボの 3 番目の演習で特定した **az104-04-vm0** のパブリック IP アドレス |

1. レコード セットを次の設定で追加します (その他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **az104-04-vm1** |
    | タイプ | **A** |
    | エイリアス レコード セット | **No** |
    | TTL | **1** |
    | TTL の単位 | **時間** |
    | IP アドレス | この課題の 3 番目の実習で特定した**az104-04-vm1** のパブリック IP アドレス |

1. **DNS ゾーン**ブレードで、**Name server 1** (ネーム サーバー 1) という名前の値をメモします。

1. Azure portal で、右上にあるアイコンをクリックして、「**Cloud Shell**」 の **PowerShell** セッションを開きます。

1. 「Cloud Shell」ウィンドウで、次を実行して、新しく作成した DNS ゾーンで **az104-04-vm0** DNS レコード セットの外部名の解決をテストします (プレースホルダ `[Name server 1]` を、[] ブラケットを含めて本タスクの前半でメモした **「Name server 1」** に差し替え、`[domain name]` プレースホルダを本タスクの前半で作成した DNS ドメイン名に差し替えます)。

   ```pwsh
   nslookup az104-04-vm0.[domain name] [Name server 1]
   ```
1. コマンドの出力に、**az104-04-vm0** のパブリック IP アドレスが含まれていることを確認します。

1. Cloud Shell ペインで、新しく作成した DNS ゾーン内の **az104-04-vm1** DNS レコード セットの外部名前解決について、次を実行してテストします (`[Name server 1]`というプレースホルダーを、このタスクで先にメモした **「Name server 1」** に置き換え、`[domain name]` プレースホルダーをこのタスクで作成した DNS 名に置き換えます)。

   ```pwsh
   nslookup az104-04-vm1.[domain name] [Name server 1]
   ```
1. コマンドの出力に、**az104-04-vm1** のパブリック IP アドレスが含まれていることを確認します。

#### リソースのクリーンアップ

   >**注**: 新しく作成された Azure リソースのうち、使用しないリソースは必ず削除してください。使用していないリソースを削除することで、予期しないコストが発生しなくなります。

1. Azure portalの 「**Cloud Shell**」 ウインドウで、**PowerShell** セッションを開始します。

1. 次のコマンドを実行して、このモジュールのラボ全体で作成されたすべてのリソース グループを一覧表示します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-04*'
   ```

1. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループを削除します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-04*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**注意**: コマンドは非同期に実行されるため (-AsJob パラメーターによって決まります)、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボで実行した内容は以下のとおりです。

- 仮想ネットワークを作成して構成しました
- 仮想マシンを仮想ネットワークにデプロイしました
- Azure VM のプライベート IP アドレスとパブリック IP アドレスを構成しました
- ネットワーク セキュリティ グループを構成しました
- Azure DNS を内部の名前解決用に構成しました
- Azure DNS を外部の名前解決用に構成しました 
