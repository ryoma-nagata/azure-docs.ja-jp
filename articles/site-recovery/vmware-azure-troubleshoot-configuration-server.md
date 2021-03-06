---
title: Azure Site Recovery を使用して VMware VM と物理サーバーを Azure にディザスター リカバリーする間の構成サーバーでの問題のトラブルシューティング | Microsoft Docs
description: この記事では、Azure Site Recovery による VMware VM と物理サーバーの Azure へのディザスター リカバリーのために構成サーバーをデプロイする場合のトラブルシューティング情報を提供します。
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/11/2018
ms.author: ramamill
ms.openlocfilehash: b819783d127c51c0d5f33b2273a37a4180cb13a6
ms.sourcegitcommit: 6b7c8b44361e87d18dba8af2da306666c41b9396
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/12/2018
ms.locfileid: "51569745"
---
# <a name="troubleshoot-configuration-server-issues"></a>構成サーバーの問題のトラブルシューティング

この記事は、[Azure Site Recovery](site-recovery-overview.md) 構成サーバーをデプロイおよび管理するときの問題のトラブルシューティングに役立ちます。 構成サーバーは、Site Recovery を使用してオンプレミスの VMware VM と物理サーバーを Azure にディザスター リカバリーするように設定するときに使用されます。 

## <a name="installation-failures"></a>インストール エラー

| **エラー メッセージ** | **推奨される操作** |
|--------------------------|------------------------|
|エラー   アカウントを読み込めませんでした。 エラー: System.IO.IOException: Unable to read data from the transport connection when installing and registering the CS server (CS サーバーをインストールおよび登録するときに転送接続からデータを読み取れません)| コンピューターで TLS 1.0 が有効になっていることを確認します。 |

## <a name="registration-failures"></a>登録エラー

登録エラーは、%ProgramData%\ASRLogs フォルダーのログを使用してデバッグできます。

登録エラーは **%ProgramData%\ASRLogs** フォルダーのログを確認してデバッグできます。

| **エラー メッセージ** | **推奨される操作** |
|--------------------------|------------------------|
|**09:20:06**:InnerException.Type: SrsRestApiClientLib.AcsException,InnerException.<br>Message: ACS50008: SAML token is invalid.<br>Trace ID: 1921ea5b-4723-4be7-8087-a75d3f9e1072<br>Correlation ID: 62fea7e6-2197-4be4-a2c0-71ceb7aa2d97><br>Timestamp: **2016-12-12 14:50:08Z<br>** | システム クロックの時刻と現地時刻に 15 分以上の差がないことを確認します。 インストーラーを再実行して登録を完了します。|
|**09:35:27** :DRRegistrationException while trying to get all disaster recovery vault for the selected certificate: : Threw Exception.Type:Microsoft.DisasterRecovery.Registration.DRRegistrationException, Exception.Message: ACS50008: SAML token is invalid.<br>Trace ID: e5ad1af1-2d39-4970-8eef-096e325c9950<br>Correlation ID: abe9deb8-3e64-464d-8375-36db9816427a<br>Timestamp: **2016-05-19 01:35:39Z**<br> | システム クロックの時刻と現地時刻に 15 分以上の差がないことを確認します。 インストーラーを再実行して登録を完了します。|
|06:28:45:Failed to create certificate<br>06:28:45:Setup cannot proceed. A certificate required to authenticate to Site Recovery cannot be created. Rerun Setup | ローカル管理者としてセットアップを実行していることを確認します。 |
