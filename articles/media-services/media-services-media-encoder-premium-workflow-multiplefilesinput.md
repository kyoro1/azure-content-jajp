<properties
	pageTitle="Premium Encoder での複数の入力ファイルとコンポーネント プロパティの使用 | Microsoft Azure"
	description="このトピックでは、setRuntimeProperties を使って複数の入力ファイルを使用し、カスタム データを、メディア エンコーダー プレミアム ワークフロー メディア プロセッサに渡す方法について説明します。"
	services="media-services"
	documentationCenter=""
	authors="xpouyat"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/12/2016"  
	ms.author="xpouyat;anilmur;juliako"/>

# Premium Encoder での複数の入力ファイルとコンポーネント プロパティの使用

## 概要

**メディア エンコーダー プレミアム ワークフロー** メディア プロセッサを使用してタスクを送信するときに、コンポーネント プロパティのカスタマイズ、クリップ リスト XML コンテンツの指定、または複数の入力ファイルの送信が必要になる場合があります。次に例をいくつか示します。

- 各入力ビデオの実行時に、テキストをビデオに重ね合わせて、テキスト値 (たとえば、現在の日付) を設定する。
- クリップ リスト XML をカスタマイズする (トリミングあり/なしの 1 つまたは複数のソース ファイルを指定)。
- エンコード中に、入力ビデオにロゴ イメージを重ね合わせる。

タスクを作成するとき、または複数の入力ファイルを送信するときに、ワークフローの一部のプロパティを変更することを**メディア エンコーダー プレミアム ワークフロー**に通知するには、**setRuntimeProperties** や **transcodeSource** を含む構成文字列を使用する必要があります。このトピックでは、その使用方法について説明します。


## 構成文字列の構文

エンコード タスクで設定する構成文字列では、次のような XML ドキュメントを使用します。

    <?xml version="1.0" encoding="utf-8"?>
    <transcodeRequest>
      <transcodeSource>
      </transcodeSource>
      <setRuntimeProperties>
        <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
      </setRuntimeProperties>
    </transcodeRequest>


ファイルから XML 構成を読み取り、ジョブ内のタスクに渡す C# コードを次に示します。

    XDocument configurationXml = XDocument.Load(xmlFileName);
    IJob job = _context.Jobs.CreateWithSingleTask(
                                                  "Media Encoder Premium Workflow",
                                                  configurationXml.ToString(),
                                                  myAsset,
                                                  "Output asset",
                                                  AssetCreationOptions.None);


## コンポーネント プロパティのカスタマイズ  

### 単純値が含まれるプロパティ
場合によっては、メディア エンコーダー プレミアム ワークフローで実行されるワークフロー ファイルと共に、コンポーネントのプロパティをカスタマイズすると便利です。

ビデオにテキストを重ね合わせるワークフローを設計したとします。そのテキスト (たとえば、現在の日付) は実行時に設定することが想定されています。これを行うには、オーバーレイ コンポーネントの text プロパティの新しい値として設定するテキストをエンコード タスクから送信します。このメカニズムを使用して、ワークフロー内のコンポーネントの他のプロパティ (オーバーレイの位置や色、AVC エンコーダーのビットレートなど) を変更することができます。

**setRuntimeProperties** は、ワークフローのコンポーネントのプロパティを上書きするときに使用されます。

例:

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
          <property propertyPath="/primarySourceFile" value="MyInputVideo.mp4" />
          <property propertyPath="Optional Overlay/Overlay/filename" value="MyLogo.png"/>
          <property propertyPath="Optional Text Overlay/Text To Image Converter/text" value="Today is Friday the 13th of May, 2016"/>
      </setRuntimeProperties>
    </transcodeRequest>


### XML 値が含まれるプロパティ

XML 値が想定されるプロパティを設定するには、`<![CDATA[ and ]]>` を使用してカプセル化します。

例:

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="/primarySourceFile" value="start.mxf" />
          <property propertyPath="/inactiveTimeout" value="65" />
          <property propertyPath="clipListXml" value="xxx">
          <extendedValue><![CDATA[<clipList>
            <clip>
              <videoSource>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </videoSource>
              <audioSource>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
            </clipList>]]>
          </extendedValue>
          </property>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

>[AZURE.NOTE]`<![CDATA[` の直後には復帰 (CR) を配置しないでください


### propertyPath 値

前の例では、propertyPath は、"/Media File Input/filename"、"/inactiveTimeout"、"clipListXml" のいずれかでした。これは、一般的に、コンポーネントの名前、つまりプロパティの名前です。パスのレベルは、"/primarySourceFile" (プロパティがワークフローのルートにある)、"/Video Processing/Graphic Overlay/Opacity" (オーバーレイがグループ内にある) のように、多かったり少なかったりします。

パスとプロパティ名を確認するには、各プロパティのすぐ横にあるアクション ボタンを使用します。このアクション ボタンをクリックし、**[編集]** を選択すると、プロパティの実際の名前と、その名前のすぐに上に名前空間が表示されます。

![アクション/編集](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture6_actionedit.png)

![プロパティ](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture7_viewproperty.png)

## 複数の入力ファイル

**メディア エンコーダー プレミアム ワークフロー**に送信するタスクごとに 2 つの資産が必要です。

- 1 つ目は、ワークフロー ファイルを含む "ワークフロー資産" です。ワークフロー ファイルは、[ワークフロー デザイナー](media-services-workflow-designer.md)を使用して設計できます。
- 2 つ目は、エンコードするメディア ファイルを含む "メディア資産" です。

複数のメディア ファイルを**メディア エンコーダー プレミアム ワークフロー** エンコーダーに送信する場合は、次の制約が適用されます。

- すべてのメディア ファイルが、同じ "メディア資産" に含まれている必要があります。複数のメディア アセットを使用することはできません。
- このメディア アセット (理想的には、エンコーダーによる処理が求められているメイン ビデオ ファイル) で、プライマリ ファイルを設定する必要があります。
- **setRuntimeProperties**、**transcodeSource** などの要素を含む構成データを、プロセッサに渡す必要があります。
  - **setRuntimeProperties** は、ワークフローのコンポーネントでファイル名などのプロパティを上書きするときに使用されます。
  - **transcodeSource** は、クリップ リスト XML コンテンツを指定するときに使用されます。

ワークフロー内での接続は次のようになります。

 - 1 つまたは複数のメディア ファイル入力コンポーネントを使用している場合、**setRuntimeProperties** を使ってファイル名を指定するときは、プライマリ ファイル コンポーネント ピンを、そのコンポーネントに接続しないでください。また、プライマリ ファイル オブジェクトとメディア ファイル入力が接続されていないことを確認してください。
 - クリップ リスト XML と 1 つのメディア ソース コンポーネントを使用する必要がある場合は、両方をまとめて接続できます。

![プライマリ ソース ファイルからメディア ファイル入力への接続なし](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture0_nopin.png)

"setRuntimeProperties を使用してファイル名のプロパティを設定する場合、プライマリ ファイルとメディア ファイル入力コンポーネントは接続されていません"

![クリップ リスト XML からクリック リスト ソースへの接続](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture1_pincliplist.png)

"クリップ リスト XML をメディア ソースに接続して transcodeSource を使用できます"


### クリップ リスト XML のカスタマイズ
実行時にワークフローでクリップ リスト XML を指定するには、構成文字列 XML で **transcodeSource** を使用します。それには、ワークフローでクリップ リスト XML ピンをメディア ソース コンポーネントに接続する必要があります。

    <?xml version="1.0" encoding="utf-16"?>
      <transcodeRequest>
        <transcodeSource>
          <clipList>
            <clip>
              <videoSource>
                <mediaFile>
                  <file>video-part1.mp4</file>
                </mediaFile>
              </videoSource>
              <audioSource>
                <mediaFile>
                  <file>video-part1.mp4</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
          </clipList>
        </transcodeSource>
        <setRuntimeProperties>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

このプロパティを使用して "式" で出力ファイルに名前を付けるために、/primarySourceFile プロパティを指定する必要がある場合は、/primarySourceFile の設定によってクリップ リストが上書きされないように、/primarySourceFile プロパティの "後" に、クリップ リスト XML をプロパティとして渡すことをお勧めします。

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="/primarySourceFile" value="c:\temp\start.mxf" />
          <property propertyPath="/inactiveTimeout" value="65" />
          <property propertyPath="clipListXml" value="xxx">
          <extendedValue><![CDATA[<clipList>
            <clip>
              <videoSource>
                <mediaFile>
                  <file>c:\temp\start.mxf</file>
                </mediaFile>
              </videoSource>
              <audioSource>
                <mediaFile>
                  <file>c:\temp\start.mxf</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
            </clipList>]]>
          </extendedValue>
          </property>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

追加の正確なフレーム トリミングを使用します。

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="/primarySourceFile" value="start.mxf" />
          <property propertyPath="/inactiveTimeout" value="65" />
          <property propertyPath="clipListXml" value="xxx">
          <extendedValue><![CDATA[<clipList>
            <clip>
              <videoSource>
                <trim>
                  <inPoint fps="25">00:00:05:24</inPoint>
                  <outPoint fps="25">00:00:10:24</outPoint>
                </trim>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </videoSource>
              <audioSource>
               <trim>
                  <inPoint fps="25">00:00:05:24</inPoint>
                  <outPoint fps="25">00:00:10:24</outPoint>
                </trim>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
            </clipList>]]>
          </extendedValue>
          </property>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>


## 例

ビデオのエンコード中、入力ビデオにロゴ イメージを重ね合わせる例を見てみましょう。この例では、入力ビデオの名前は "MyInputVideo.mp4"、ロゴの名前は "MyLogo.png" です。以下の手順を実行する必要があります。

- ワークフロー ファイルを使用してワークフロー資産を作成します (以下の例を参照してください)。
- メディア アセットを作成します。このアセットには、MyInputVideo.mp4 (プライマリ ファイル) と MyLogo.png の 2 つのファイルが含まれます。
- 上記の入力資産と共に、タスクをメディア エンコーダー プレミアム ワークフロー メディア プロセッサに送信し、次の構成文字列を指定します。

Configuration:

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
          <property propertyPath="/primarySourceFile" value="MyInputVideo.mp4" />
          <property propertyPath="Media File Input Logo/filename" value="MyLogo.png" />
        </setRuntimeProperties>
      </transcodeRequest>


上記の例では、ビデオ ファイルの名前は、メディア ファイル入力コンポーネントと primarySourceFile プロパティに送信されます。ロゴ ファイルの名前は、グラフィック オーバーレイ コンポーネントに接続されているもう 1 つのメディア ファイル入力に送信されます。

>[AZURE.NOTE]ビデオ ファイル名は、primarySourceFile プロパティに送信されます。その理由は、ワークフローでは、たとえば、式を使用して正しい出力ファイル名を構築するために、このプロパティが使用されるためです。


### ビデオにロゴを重ね合わせるワークフローの作成手順     

ここでは、ビデオとイメージの 2 つのファイルを入力として受け取るワークフローの作成手順について説明します。このワークフローは、ビデオにイメージを重ね合わせます。

**ワークフロー デザイナー**を開いて、**[ファイル]**、**[新しいワークスペース]**、**[Transcode Blueprint (トランスコード設計)]** の順に選択します

新しいワークフローには 3 つの要素があります。

- プライマリ ソース ファイル
- クリップ リスト XML
- 出力ファイル/資産

![New Encoding Workflow](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture9_empty.png)

*新しいエンコード ワークフロー*


入力メディア ファイルを受け取るには、まず、メディア ファイル入力コンポーネントを追加します。ワークフローにコンポーネントを追加するには、[リポジトリの検索] ボックスでコンポーネントを探し、必要な項目をデザイナー ウィンドウにドラッグします。

次に、ワークフローの設計に使用するビデオ ファイルを追加します。これを行うには、ワークフロー デザイナーの背景ウィンドウをクリックし、右側のプロパティ ウィンドウでプライマリ ソース ファイル プロパティを探します。フォルダー アイコンをクリックし、適切なビデオ ファイルを選択します。

![プライマリ ファイル ソース](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture10_primaryfile.png)

*"プライマリ ファイル ソース"*


次に、メディア ファイル入力コンポーネントで、ビデオ ファイルを指定します。

![メディア ファイル入力ソース](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture11_mediafileinput.png)

*"メディア ファイル入力ソース"*


その直後に、ファイルはメディア ファイル入力コンポーネントによって調査され、調査されたファイルを反映するように出力ピンに設定されます。

次に、"ビデオ データ タイプ アップデーター" を追加して、色空間を Rec.709 に指定します。"データのレイアウト/レイアウトの種類 = 構成可能な平面" に設定された "ビデオ形式コンバーター" を追加します。これにより、ビデオ ストリームが、オーバーレイ コンポーネントのソースとして取得できる形式に変換されます。

![ビデオ データ タイプ アップデーターと形式コンバーター](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture12_formatconverter.png)

*"ビデオ データ タイプ アップデーターと形式コンバーター"*

![レイアウトの種類 = 構成可能な平面](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture12_formatconverter2.png)

"レイアウトの種類 = 構成可能な平面"

次に、ビデオ オーバーレイ コンポーネントを追加し、(圧縮されていない) ビデオ ピンを、メディア ファイル入力の (圧縮されていない) ビデオ ピンに接続します。

(ロゴ ファイルを読み込むために) メディア ファイル入力をもう 1 つ追加し、このコンポーネントをクリックします。次に、その名前を "メディア ファイル入力ロゴ" に変更し、ファイル プロパティでイメージ (png ファイルなど) を選択します。圧縮されていないイメージ ピンを、オーバーレイの圧縮されていないイメージ ピンに接続します。

![オーバーレイ コンポーネントとイメージ ファイル ソース](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture13_overlay.png)

*"オーバーレイ コンポーネントとイメージ ファイル ソース"*


ビデオのロゴの位置を変更する (たとえば、ビデオの左上隅から 10% 離して配置する) 必要がある場合は、[手操作入力] チェック ボックスをオフにします。このように操作できるのは、メディア ファイル入力を使用して、ロゴ ファイルをオーバーレイ コンポーネントに配置しているためです。

![オーバーレイの位置](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture14_overlay_position.png)

*"オーバーレイの位置"*


ビデオ ストリームを H.264 にエンコードするには、デザイナー画面に AVC ビデオ エンコーダー コンポーネントと AAC エンコーダー コンポーネントを追加します。ピンを接続します。AAC エンコーダーを設定し、オーディオ形式の変換/プリセット: 2.0 (L, R) を選択します。

![オーディオ エンコーダーとビデオ エンコーダー](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture15_encoders.png)

*"オーディオ エンコーダーとビデオ エンコーダー"*


ここで、次に示すように **ISO Mpeg-4 マルチプレクサ―** コンポーネントと**ファイル出力**コンポーネントを追加して、ピンを接続します。

![MP4 マルチプレクサ―とファイル出力](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture16_mp4output.png)

*"MP4 マルチプレクサ―とファイル出力"*


出力ファイルの名前を設定する必要があります。**ファイル出力** コンポーネントをクリックし、ファイルの式を編集します。

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_withoverlay.mp4

![ファイル出力名](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture17_filenameoutput.png)

*"ファイル出力名"*

ワークフローをローカルで実行して、適切に実行されていることを確認します。

完了したら、Azure Media Services で実行できます。

まず、Azure Media Services で、ビデオ ファイルとロゴの、2 つのファイルが含まれる資産を準備します。これは、.NET または REST API を使用して行うことができます。Azure ポータルまたは [Azure Media Services Explorer](https://github.com/Azure/Azure-Media-Services-Explorer) (AMSE) を使用して実行することもできます。

このチュートリアルでは、AMSE で資産を管理する方法について説明します。ファイルを資産に追加する方法は 2 種類あります。

- ローカル フォルダーを作成し、そのフォルダーに 2 つのファイルをコピーします。次に、フォルダーを **[資産]** タブにドラッグ アンド ドロップします
- ビデオ ファイルを資産としてアップロードし、資産情報を表示します。次に [ファイル] タブに移動し、その他のファイル (ロゴ) をアップロードします。

>[AZURE.NOTE]アセット (メイン ビデオ ファイル) では必ずプライマリ ファイルを設定してください。

![AMSE のアセット ファイル](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture18_assetinamse.png)

*"AMSE のアセット ファイル"*


アセットを選択し、Premium Encoder でエンコードすることを選択します。ワークフローをアップロードし、それを選択します。

ボタンをクリックしてデータをプロセッサに渡し、次の XML を追加して、ランタイム プロパティを設定します。

![AMSE の Premium Encoder](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture19_amsepremium.png)

*"AMSE の Premium Encoder"*


次に、以下の XML データを貼り付けます。メディア ファイル入力と primarySourceFile の両方について、ビデオ ファイルの名前を指定する必要があります。ロゴのファイル名も指定します。

    <?xml version="1.0" encoding="utf-16"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="Media File Input/filename" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
          <property propertyPath="/primarySourceFile" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

![setRuntimeProperties](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture20_amsexmldata.png)

*setRuntimeProperties*


.NET SDK を使用してタスクを作成および実行する場合、この XML データは、構成文字列として渡す必要があります。

    public ITask AddNew(string taskName, IMediaProcessor mediaProcessor, string configuration, TaskOptions options);

ジョブが完了すると、出力資産の MP4 ファイルにオーバーレイが表示されます。

![ビデオでのオーバーレイ](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture21_resultoverlay.png)

*"ビデオでのオーバーレイ"*


サンプル ワークフローは [GitHub](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows/) からダウンロードできします。


## 関連項目

- [Azure Media Services の Premium Encoding の紹介 (ブログの投稿)](http://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services)

- [Azure Media Services で Premium Encoding を使用する方法 (ブログの投稿)](http://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services)

- [Azure Media Services を使用してオンデマンド コンテンツをエンコードする](media-services-encode-asset.md#media_encoder_premium_workflow)

- [メディア エンコーダー プレミアム ワークフローの形式とコーデック](media-services-premium-workflow-encoder-formats.md)

- [サンプルのワークフロー ファイル](https://github.com/AzureMediaServicesSamples/Encoding-Presets/tree/master/VoD/MediaEncoderPremiumWorkfows)

- [Azure Media Services Explorer ツール](http://aka.ms/amse)

## Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0824_2016-->