<properties
	pageTitle="メディア処理のスケール設定の概要 | Microsoft Azure"
	description="このトピックでは、Azure Media Services を使用したメディア処理のスケール設定の概要を示します。"
	services="media-services"
	documentationCenter=""
	authors="juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/29/2016"
	ms.author="juliako"/>


# メディア処理のスケール設定の概要

このページでは、メディア処理のスケールを設定する方法と、スケール設定を行う理由の概要について説明します。

## Overview

Media Services アカウントは、メディア処理タスクを処理する速度を決定する予約ユニットの種類に関連付けられます。予約ユニットの種類は、**S1**、**S2**、**S3** から選択できます。たとえば、同じエンコード ジョブの場合に、予約ユニットの種類として **S1** よりも **S2** を使用する方が、ジョブの実行が高速になります。詳細については、[予約ユニットの種類](https://azure.microsoft.com/blog/high-speed-encoding-with-azure-media-services/)に関するページを参照してください。

予約ユニットの種類を指定するだけでなく、予約ユニットを使用したアカウントのプロビジョニングを指定することもできます。プロビジョニングされた予約ユニットの数によって、所定のアカウントで並列処理できるメディア タスクの数が決まります。たとえば、アカウントの予約ユニットの数が 5 である場合、処理するタスクがある限り、5 個のメディア タスクが並列実行されます。残りのタスクはキューで待機して、実行中のタスクが完了するとキューから取り出されて順番に処理されます。アカウントに予約ユニットが用意されていない場合、タスクは逐次処理されます。この場合、あるタスクが終了した後、次のタスクが開始するまでの待機時間は、システムのリソースが利用できるかどうかに左右されます。

## さまざまな予約ユニットの種類の選択

次の表は、さまざまなエンコーディングの速度を選択して決定するときに役立ちます。ベンチマーク ケースや SAS URL も記載されています。これらの SAS URL を使用してビデオをダウンロードして、独自のテストを実行できます。

シナリオ|**S1**|**S2**|**S3**|
----------|------------|----------|------------
目的のユース ケース| シングル ビットレート エンコード。<br/>解像度が SD 以下であり、時間的制約がなく、低コストなファイル。|シングル ビットレート エンコードと複数ビットレート エンコード。<br/>通常は SD と HD 両方のエンコードに使用されます。 |シングル ビットレート エンコードと複数ビットレート エンコード。<br/>解像度がフル HD および 4 K であるビデオ。時間に依存しない高速ターンアラウンド エンコード。 
ベンチマーク|[入力ファイル: 640x360p を 29.97 フレーム/秒で 5 分間](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_360p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>同じ解像度でシングル ビットレート MP4 ファイルにエンコードするには約 11 分かかります。|[入力ファイル: 1280x720p を 29.97 フレーム/秒で 5 分間](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_720p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>[H264 Single Bitrate 720p] プリセットでのエンコードは約 5 分かかります。<br/><br/>[H264 Multiple Bitrate 720p] プリセットでのエンコードは約 11 分半かかります。|[入力ファイル: 1920x1080p を 29.97 フレーム/秒で 5 分間](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_1080p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>[H264 Single Bitrate 1080p] プリセットでのエンコードは約 2.7 分かかります。<br/><br/>[H264 Multiple Bitrate 1080p] プリセットでのエンコードは約 5.7 分かかります。

##考慮事項

>[AZURE.IMPORTANT] このセクションで説明する考慮事項を確認してください。

- 予約ユニットは、Azure Media Indexer を使用するインデックス作成ジョブを含む、すべてのメディア処理を並列化するために動作します。ただし、エンコードとは異なり、インデックス作成ジョブでは高速予約ユニットを使用した高速処理は行われません。

- 共有プールを使用する (すなわち、予約ユニットを使用しない) 場合のエンコード タスクのパフォーマンスは S1 予約ユニットを使用したときと同等になります。ただし、タスクがキューに登録された状態である時間の上限はなく、同時に実行されるタスクの数は最大で 1 つです。

- ブラジル南部、インド西部、インド中部、およびインド南部のデータ センターでは、占有ユニットの種類 **S2** は提供されません。

- ブラジル南部、インド西部、およびインド中部のデータ センターでは、占有ユニットの種類 **S3** は提供されません。

- コストの計算時には、24 時間の期間内に指定されたユニットの最大数が使用されます。


##クォータと制限

クォータと制限の詳細、サポート チケットを開く方法については、「[Quotas and limitations (クォータと制限)](media-services-quotas-and-limitations.md)」をご覧ください

##次のステップ

[Azure ポータルを使用してメディア処理のスケールを設定する](media-services-portal-scale-media-processing.md)

##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0831_2016-->