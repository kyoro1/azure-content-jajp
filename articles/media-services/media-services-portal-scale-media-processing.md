<properties
	pageTitle=" Azure ポータルを使用したメディア処理のスケール設定 | Microsoft Azure"
	description="このチュートリアルでは、Azure ポータルを使用したメディア処理のスケール設定の手順について説明します。"
	services="media-services"
	documentationCenter=""
	authors="Juliako"
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

# 予約ユニットの種類の変更

> [AZURE.SELECTOR]
- [.NET](media-services-dotnet-encoding-units.md)
- [ポータル](media-services-portal-scale-media-processing.md)
- [REST ()](https://msdn.microsoft.com/library/azure/dn859236.aspx)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

## Overview

メディア処理のスケール設定についての詳細は、[概要](media-services-scale-media-processing-overview.md)に関するページを参照してください。

## メディア処理のスケール設定

予約ユニットの種類と予約ユニットの数を変更するには、以下の手順に従います。

1. [Azure ポータル](https://portal.azure.com/)にログインします。

2. **[設定]** ウィンドウで **[Media reserved units (メディア予約ユニット)]** をクリックします。

	選択した予約ユニットの種類の予約ユニット数を変更するには、**[Media Serverd Units (メディア予約ユニット)]** スライダーを使用します。

	**[占有ユニットの種類]** を変更するには、[S1]、[S2]、または [S3] をクリックします。

	![[プロセッサ] ページ](./media/media-services-portal-scale-media-processing/media-services-scale-media-processing.png)

3. [保存] ボタンを押して、変更を保存します。

	[保存] をクリックすると、新しい予約ユニットが割り当てられます。

##次のステップ

Media Services のラーニング パスを確認します。

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0831_2016-->