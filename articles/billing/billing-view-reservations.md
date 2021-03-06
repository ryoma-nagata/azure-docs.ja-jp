---
title: Azure の予約を表示する | Microsoft Docs
description: Azure portal で Azure の予約を表示する方法について説明します。
services: billing
documentationcenter: ''
author: yashesvi
manager: yashar
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/03/2018
ms.author: cwatson
ms.openlocfilehash: 2ae30ca55f3ca03a64438025960ddd807e288216
ms.sourcegitcommit: f58fc4748053a50c34a56314cf99ec56f33fd616
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/04/2018
ms.locfileid: "48272618"
---
# <a name="view-azure-reservations-in-the-azure-portal"></a>Azure portal で Azure の予約を表示する

ご利用のサブスクリプションの種類とアクセス許可に応じて、Azure の予約を表示する方法がいくつかあります。

## <a name="view-reservations-as-owner-or-reader"></a>所有者または閲覧者として予約を表示する

既定では、予約を購入すると、お客様とアカウント管理者は予約を表示できます。 お客様とアカウント管理者は、予約の所有者ロールを自動的に取得します。 その他のユーザーが予約を表示できるようにするには、予約でそのユーザーを**所有者**または**閲覧者**として追加する必要があります。 詳細については、「[予約を管理できるユーザーを追加または変更する](billing-manage-reserved-vm-instance.md#add-or-change-users-who-can-manage-a-reservation)」を参照してください。
 
所有者または閲覧者として予約を表示するには

1. [Azure Portal]( http://portal.azure.com) にサインインします。
1. **[予約]** で検索します。

    ![Azure portal の検索を表示するスクリーンショット](./media/billing-view-reservation/portal-reservation-search.png)

1. ご自分が所有者または閲覧者ロールになっている予約の一覧を確認します。

予約の範囲を変更する必要がある場合は、予約を分割するか、予約を管理できるユーザーを変更します。[Azure の予約の管理](billing-manage-reserved-vm-instance.md)に関するページを参照してください。

## <a name="view-reservation-transactions-for-enterprise-enrollments"></a>エンタープライズ登録に対する予約トランザクションを表示する

 エンタープライズ登録を先導するパートナーがいる場合は、EA ポータルの **[レポート]** に移動することで予約を表示します。 その他のエンタープライズ登録の場合は、EA ポータルおよび Azure portal で予約を表示できます。 予約トランザクションを表示するには、EA 管理者である必要があります。

Azure portal で予約トランザクションを表示するには

1. [Azure Portal]( http://portal.azure.com) にサインインします。
1. **[コストの管理と請求]** で検索します。

    ![Azure portal の検索を表示するスクリーンショット](./media/billing-view-reservation/portal-cm-billing-search.png)

1. **[予約トランザクション]** を選択します。
1. 結果をフィルター処理するには、**[期間]**、**[種類]** または **[説明]** を選択します。
1. **[適用]** を選択します。

    ![予約トランザクションの結果を示すスクリーンショット](./media/billing-view-reservation/portal-billing-reservation-transaction-results.png)

API を使用してデータを取得するには、「[Get Reserved Instance transaction charges for enterprise customers](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges)」 (エンタープライズ顧客向けの予約インスタンス トランザクションを取得する) を参照してください。

## <a name="next-steps"></a>次の手順

Azure の予約の詳細については、次の記事を参照してください。

- [Azure の予約とは](billing-save-compute-costs-reservations.md)
- [Cosmos DB 予約容量の前払い](../cosmos-db/cosmos-db-reserved-capacity.md)
- [Azure SQL Database の容量が予約された SQL Database 計算リソースの前払い](../sql-database/sql-database-reserved-capacity.md)
- [Azure Reserved VM Instances による仮想マシンの前払い](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Azure の予約の管理](billing-manage-reserved-vm-instance.md)
- [従量課金制サブスクリプションの予約使用量について](billing-understand-reserved-instance-usage.md)
- [エンタープライズ加入契約の予約使用量について](billing-understand-reserved-instance-usage-ea.md)
- [CSP サブスクリプションの予約の使用状況について](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-support"></a>お困りの際は、 サポートにお問い合せください

まだ他に質問がある場合は、問題を迅速に解決できるよう [サポートにお問い合わせ](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) ください。
