---
title: Windows IaaS VM で Azure Disk Encryption を有効にする| Microsoft Docs
description: この記事では、Windows IaaS VM で Microsoft Azure Disk Encryption を有効にする手順について説明します。
author: mestew
ms.service: security
ms.subservice: Azure Disk Encryption
ms.topic: article
ms.author: mstewart
ms.date: 10/12/2018
ms.openlocfilehash: 545723a020609766b9556746e6547eb8b93e5de9
ms.sourcegitcommit: db2cb1c4add355074c384f403c8d9fcd03d12b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/15/2018
ms.locfileid: "51687522"
---
# <a name="enable-azure-disk-encryption-for-windows-iaas-vms"></a>Windows IaaS VM で Azure Disk Encryption を有効にする 

有効にできるディスク暗号化シナリオは多数あり、手順はシナリオによって異なる場合があります。 以下のセクションでは、Windows IaaS VM 用のシナリオについて詳しく説明します。 ディスク暗号化を使用する前に、[Azure Disk Encryption の前提条件](../security/azure-security-disk-encryption-prerequisites.md)を満たす必要があります。 

ディスクを暗号化する前に、[スナップショット](../virtual-machines/windows/snapshot-copy-managed-disk.md)またはバックアップを作成します。 バックアップがあると、暗号化中に予期しないエラーが発生した場合に、回復オプションを使用できるようになります。 マネージド ディスクを含む VM では、暗号化する前にバックアップが必要になります。 バックアップを作成すると、Set-AzureRmVMDiskEncryptionExtension コマンドレットを使用し、-skipVmBackup パラメーターを指定してマネージド ディスクを暗号化できます。 暗号化された VM のバックアップと復元方法の詳細については、[Azure Backup](../backup/backup-azure-vms-encryption.md) に関する記事を参照してください。 

>[!WARNING]
> Azure Disk Encryption では、Key Vault と VM が同じリージョンに配置されている必要があります。 暗号化する VM と同じリージョン内に Key Vault を作成し、使用します。 


## <a name="bkmk_RunningWinVM"></a> 既存または実行中の IaaS Windows VM で暗号化を有効にする
このシナリオでは、テンプレート、PowerShell コマンドレット、または CLI コマンドを使用して、暗号化を有効にすることができます。 次のセクションでは、Azure Disk Encryption を有効にする方法を詳しく説明します。 仮想マシンを拡張するためのスキーマの情報が必要な場合は、[Windows 用 Azure Disk Encryption 拡張機能](../virtual-machines/extensions/azure-disk-enc-windows.md)に関する記事を参照してください。

>[!IMPORTANT]
 >Azure Disk Encryption を有効にする前に、Azure Disk Encryption 以外を使用して、マネージド ディスク ベースの VM インスタンスのスナップショットまたはバックアップ (またはその両方) を作成する必要があります。 マネージド ディスクのスナップショットは、ポータルから作成できます。[Azure Backup](../backup/backup-azure-vms-encryption.md) を使用することもできます。 バックアップがあると、暗号化中に予期しないエラーが発生した場合に、回復オプションを使用できるようになります。 バックアップを作成すると、Set-AzureRmVMDiskEncryptionExtension コマンドレットを使用し、-skipVmBackup パラメーターを指定してマネージド ディスクを暗号化できます。 バックアップが作成されておらず、このパラメーターを指定していない場合、マネージド ディスク ベースの VM に対して、Set-AzureRmVMDiskEncryptionExtension コマンドを実行するとエラーが発生します。 
>
>暗号化を有効または無効にすると、VM が再起動する場合があります。 
>

### <a name="bkmk_RunningWinVMPSH"></a> Azure PowerShell を使用して既存または実行中の VM で暗号化を有効にする 
[Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) コマンドレットを使用して、Azure で実行中の IaaS 仮想マシンで暗号化を有効にします。 

-  **実行中の VM を暗号化する:** 次のスクリプトでは変数を初期化し、Set-AzureRmVMDiskEncryptionExtension コマンドレットを実行します。 前提条件として、リソース グループ、VM、およびキー コンテナーが既に作成されている必要があります。 MySecureRg、MySecureVM、および MySecureVault を実際の値に置き換えます。

     ```azurepowershell-interactive
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **KEK を使用して実行中の VM を暗号化する:** 

     ```azurepowershell-interactive
     $rgName = 'MySecureRg';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > disk-encryption-keyvault パラメーターの値の構文は、/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] という完全な識別子の文字列です。</br> key-encryption-key パラメーターの値の構文は、 https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] という KEK への完全な URI です。 

- **ディスクが暗号化されていることを確認する:** IaaS VM の暗号化の状態を確認するには、[Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) コマンドレットを使用します。 
     ```azurepowershell-interactive
     Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName 'MySecureRg' -VMName 'MySecureVM'
     ```
    
- **ディスク暗号化を無効にする:** 暗号化を無効にするには、[Disable-AzureRmVMDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) コマンドレットを使用します。 OS とデータ ディスクの両方が暗号化されている場合は、Windows VM におけるデータ ディスクの暗号化の無効化が想定どおりに機能しません。 代わりに、すべてのディスクで暗号化を無効にしてください。

     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM'
     ```

### <a name="bkmk_RunningWinVMCLI"></a> Azure CLI を使用して既存または実行中の VM で暗号化を有効にする
[az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) コマンドを使用して、Azure で実行中の IaaS 仮想マシンで暗号化を有効にします。

-  **実行中の VM を暗号化する:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **KEK を使用して実行中の VM を暗号化する:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > disk-encryption-keyvault パラメーターの値の構文は、/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] という完全な識別子の文字列です。 </br> key-encryption-key パラメーターの値の構文は、 https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] という KEK への完全な URI です。 

- **ディスクが暗号化されていることを確認する:** IaaS VM の暗号化の状態を確認するには、[az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) コマンドを使用します。 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
     ```

- **暗号化を無効にする:** 暗号化を無効にするには、[az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) コマンドを使用します。 OS とデータ ディスクの両方が暗号化されている場合は、Windows VM におけるデータ ディスクの暗号化の無効化が想定どおりに機能しません。 代わりに、すべてのディスクで暗号化を無効にしてください。

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
     ```
 
 > [!NOTE]
 >Azure Disk Encryption を有効にする前に、Azure Disk Encryption 以外を使用して、マネージド ディスク ベースの VM インスタンスのスナップショットまたはバックアップ (またはその両方) を作成する必要があります。 マネージド ディスクのスナップショットは、ポータルから作成できます。[Azure Backup](../backup/backup-azure-vms-encryption.md) を使用することもできます。 バックアップがあると、暗号化中に予期しないエラーが発生した場合に、回復オプションを使用できるようになります。 バックアップを作成すると、Set-AzureRmVMDiskEncryptionExtension コマンドレットを使用し、-skipVmBackup パラメーターを指定してマネージド ディスクを暗号化できます。 バックアップが作成されておらず、このパラメーターを指定していない場合、マネージド ディスク ベースの VM に対してこのコマンドを実行するとエラーが発生します。 
>
>暗号化を有効または無効にすると、VM が再起動する場合があります。 

### <a name="bkmk_RunningWinVMwRM"> </a>Resource Manager テンプレートの使用
[実行中の Windows VM を暗号化するための Resource Manager テンプレート](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad)を使用して、Azure で既存または実行中の IaaS Windows VM に対してディスク暗号化を有効にすることができます。


1. Azure クイックスタート テンプレートで、**[Azure に配置する]** をクリックします。

2. サブスクリプション、リソース グループ、場所、設定、法律条項、および契約を選択します。 **[購入]** をクリックして、既存または実行中の IaaS VM で暗号化を有効にします。

次の表は、既存または実行中の VM に対する、Resource Manager テンプレートのパラメーターをまとめたものです。

| パラメーター | 説明 |
| --- | --- |
| vmName | 暗号化操作を実行する VM の名前。 |
| KeyVaultName | BitLocker キーのアップロード先となる Key Vault の名前。 これは、コマンドレット `(Get-AzureRmKeyVault -ResourceGroupName <MyResourceGroupName>). Vaultname` または Azure CLI コマンド `az keyvault list --resource-group "MySecureGroup"  |ConvertFrom-JSON` を使用して取得できます。|
| keyVaultResourceGroup | キー コンテナーが含まれているリソース グループの名前|
|  keyEncryptionKeyURL | 生成された BitLocker キーの暗号化に使用されるキー暗号化キーの URL。 このパラメーターは、UseExistingKek ドロップダウン リストで **nokek** を選択した場合には省略可能です。 UseExistingKek ドロップダウン リストで **kek** を選択した場合は、_keyEncryptionKeyURL_ 値を入力する必要があります。 |
| volumeType | 暗号化操作が実行されるボリュームの種類。 有効な値は _OS_、_Data_、および _All_ です。 
| forceUpdateTag | 操作が強制実行である必要があるたびに、GUID のような一意の値を渡します。 |
| resizeOSDisk | OS パーティションは、システム ボリュームを分割する前に、完全な OS VHD が占有できるようにサイズ変更する必要があります。 |
| location | すべてのリソースの場所。 |

## <a name="encrypt-virtual-machine-scale-sets"></a>仮想マシン スケール セットを暗号化する
[Azure Virtual Machine Scale Sets](../virtual-machine-scale-sets/overview.md) では、負荷分散が行われる同一の VM のグループを作成して管理することができます。 需要または定義されたスケジュールに応じて、VM インスタンスの数を自動的に増減させることができます。 CLI または Azure PowerShell を使用して仮想マシン スケール セットを暗号化します。


### <a name="register-for-disk-encryption-preview-using-azure-powershell"></a>Azure PowerShell を使用してディスク暗号化プレビューの登録をする

仮想マシン スケール セットのプレビューのための Azure ディスク暗号化では、[Register-AzureRmProviderFeature](/powershell/module/azurerm.resources/register-azurermproviderfeature) を使用してサブスクリプションを自己登録する必要があります。 ディスク暗号化のプレビュー機能を初めて利用する場合は、以下の手順を行えばよいだけです｡

```azurepowershell-interactive
Register-AzureRmProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName "UnifiedDiskEncryption"
```

登録要求が伝達されるのに最大 10 分時間がかかることがあります｡ 登録状態は、[Get-AzureRmProviderFeature](/powershell/module/AzureRM.Resources/Get-AzureRmProviderFeature) を使用して確認できます。 `RegistrationState` から *Registered* と報告されたら、[Register-AzureRmResourceProvider](/powershell/module/AzureRM.Resources/Register-AzureRmResourceProvider) を使用して *Microsoft.Compute* プロバイダーを登録し直します。

```azurepowershell-interactive
Get-AzureRmProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Compute
```

### <a name="encrypt-virtual-machine-scale-sets-with-azure-powershell"></a>Azure PowerShell を使用して仮想マシン スケール セットを暗号化する

[Set-AzureRmVmssDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmssdiskencryptionextension) コマンドレットを使用して、Windows 仮想マシン スケール セットで暗号化を有効にします。 前提条件として、リソース グループ、VM、およびキー コンテナーが既に作成されている必要があります。

-  **実行中の仮想マシン スケール セットを暗号化する**:
    ```azurepowershell-interactive
     $rgName= "MySecureRg";
     $VmssName = "MySecureVmss";
     $KeyVaultName= "MySecureVault";
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgName;
     $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     Set-AzureRmVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;


-  **Encrypt a running virtual machine scale set using KEK to wrap the key**:

    ```azurepowershell-interactive
     $rgName= "MySecureRg";
     $VmssName = "MySecureVmss";
     $KeyVaultName= "MySecureVault";
     $keyEncryptionKeyName = "MyKeyEncryptionKey";
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgName;
     $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     Set-AzureRmVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;
    ```

- **仮想マシン スケール セットの暗号化状態を取得する:** [Get-AzureRmVmssVMDiskEncryption](/powershell/module/azurerm.compute/get-azurermvmssvmdiskencryption) コマンドレットを使用します。
    
    ```azurepowershell-interactive
    get-AzureRmVmssVMDiskEncryption -ResourceGroupName "MySecureRG" -VMScaleSetName "MySecureVmss"
    ```

- **仮想マシン スケール セットの暗号化を無効にする:** [Disable-AzureRmVmssDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmssdiskencryption) コマンドレットを使用します。 

    ```azurepowershell-interactive
    Disable-AzureRmVmssDiskEncryption -ResourceGroupName "MySecureRG" -VMScaleSetName "MySecureVmss"
    ```

### <a name="register-for-disk-encryption-preview-using-azure-cli"></a>Azure CLI を使用してディスク暗号化プレビューの登録をする

仮想マシン スケール セットのプレビューのための Azure ディスク暗号化では、 [az feature register](/cli/azure/feature#az_feature_register) を使用してサブスクリプションを自己登録する必要があります。 ディスク暗号化のプレビュー機能を初めて利用する場合は､以下の手順を行えばよいだけです｡

```azurecli-interactive
az feature register --name UnifiedDiskEncryption --namespace Microsoft.Compute
```

登録要求が伝達されるのに最大 10 分時間がかかることがあります｡ 登録状態は､[az feature show](/cli/azure/feature#az_feature_show) で確認できます｡ `State` から *Registered* と報告されたら､[az provider register](/cli/azure/provider#az_provider_register) を使って *Microsoft.Compute* プロバイダーを登録し直します。

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```



###  <a name="encrypt-virtual-machine-scale-sets-with-azure-cli"></a>Azure CLI を使用して仮想マシン スケール セットを暗号化する

[az vmss encryption enable](/cli/azure/vmss/encryption#az-vmss-encryption-enable) を使用して、Windows 仮想マシン スケール セットで暗号化を有効にします。 スケール セットのアップグレード ポリシーが manual に設定されている場合は、[az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances) を使用して暗号化を開始します。 前提条件として、リソース グループ、VM、およびキー コンテナーが既に作成されている必要があります。

-  **実行中の仮想マシン スケール セットを暗号化する**
    ```azurecli-interactive
     az vmss encryption enable --resource-group "MySecureRG" --name "MySecureVmss" --disk-encryption-keyvault "MySecureVault" 
    ```

-  **キーをラップするために KEK を使用して、実行中の仮想マシン スケール セットを暗号化する**
    ```azurecli-interactive
     az vmss encryption enable --resource-group "MySecureRG" --name "MySecureVmss" --disk-encryption-keyvault "MySecureVault" --key-encryption-key "MyKEK" --key-encryption-keyvault "MySecureVault" 

     ```
- **仮想マシン スケール セットの暗号化状態を取得する:** [az vmss encryption show](/cli/azure/vmss/encryption#az-vmss-encryption-show) を使用します

    ```azurecli-interactive
     az vmss encryption show --resource-group "MySecureRG" --name "MySecureVmss"
    ```

- **仮想マシン スケール セットで暗号化を無効化する**: [az vmss encryption disable](/cli/azure/vmss/encryption#az-vmss-encryption-disable) を使用します
    ```azurecli-interactive
     az vmss encryption disable --resource-group "MySecureRG" --name "MySecureVmss"
    ```

## <a name="bkmk_VHDpre"> </a>顧客が暗号化した VHD と暗号化キーから作成された新しい IaaS VM
このシナリオでは、PowerShell コマンドレットまたは CLI コマンドを使用して、暗号化を有効にすることができます。 

付録の手順を使用して、Azure で使用できる事前に暗号化されたイメージを準備します。 イメージを作成したら、次のセクションの手順に従って、暗号化された Azure VM を作成できます。

* [事前に暗号化された Windows VHD を準備する](azure-security-disk-encryption-appendix.md#bkmk_preWin)
* [事前に暗号化された Linux VHD を準備する](azure-security-disk-encryption-appendix.md#bkmk_preLinux)

>[!IMPORTANT]
 >Azure Disk Encryption を有効にする前に、Azure Disk Encryption 以外を使用して、マネージド ディスク ベースの VM インスタンスのスナップショットまたはバックアップ (またはその両方) を作成する必要があります。 マネージド ディスクのスナップショットは、ポータルから作成できます。[Azure Backup](../backup/backup-azure-vms-encryption.md) を使用することもできます。 バックアップがあると、暗号化中に予期しないエラーが発生した場合に、回復オプションを使用できるようになります。 バックアップを作成すると、Set-AzureRmVMDiskEncryptionExtension コマンドレットを使用し、-skipVmBackup パラメーターを指定してマネージド ディスクを暗号化できます。 バックアップが作成されておらず、このパラメーターを指定していない場合、マネージド ディスク ベースの VM に対して、Set-AzureRmVMDiskEncryptionExtension コマンドを実行するとエラーが発生します。 
>
>暗号化を有効または無効にすると、VM が再起動する場合があります。 


### <a name="bkmk_VHDprePSH"> </a> Azure PowerShell を使用して事前に暗号化された VHD を含む VM を暗号化する
PowerShell コマンドレット [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk#examples) を使用して、暗号化された VHD でディスク暗号化を有効にすることができます。 次の例では、一般的ないくつかのパラメーターを示します。 

```azurepowershell-interactive
$VirtualMachine = New-AzureRmVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzureRmVM -VM $VirtualMachine -ResourceGroupName "MySecureRG"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>新しく追加されたデータ ディスクで暗号化を有効にする
[PowerShell を使用して Windows VM に新しいディスクを追加する](../virtual-machines/windows/attach-disk-ps.md)ことができます。また、[Azure Portal を使用する](../virtual-machines/windows/attach-managed-disk-portal.md)こともできます。 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Azure PowerShell を使用して新しく追加されたディスクで暗号化を有効にする
 PowerShell を使用して Windows VM 用の新しいディスクを暗号化する場合は、新しいシーケンス バージョンを指定する必要があります。 シーケンス バージョンは一意である必要があります。 次のスクリプトでは、シーケンス バージョン用の GUID が生成されます。 場合によっては、Azure Disk Encryption 拡張機能により、新しく追加されたデータ ディスクが自動的に暗号化される可能性があります。 新しいディスクがオンラインになった後に VM を再起動すると、自動的に暗号化が行われることが多いです。 通常、これは以前に VM でディスクの暗号化を実行したときに、ボリュームの種類に "All" を指定していることが原因です。 自動の暗号化が新しく追加されたデータ ディスク上で発生する場合は、新しいシーケンス バージョンで Set-AzureRmVmDiskEncryptionExtension コマンドレットをもう一度実行することをお勧めします。 新しいデータ ディスクが自動的に暗号化されており、暗号化したくない場合は、まずすべてのドライブの暗号化を解除して、ボリュームの種類に OS を指定して新しいシーケンス バージョンで再暗号化します。 
  
 

-  **実行中の VM を暗号化する:** 次のスクリプトでは変数を初期化し、Set-AzureRmVMDiskEncryptionExtension コマンドレットを実行します。 前提条件として、リソース グループ、VM、およびキー コンテナーが既に作成されている必要があります。 MySecureRg、MySecureVM、および MySecureVault を実際の値に置き換えます。 この例では -VolumeType パラメーターに "All" が使用されているため、OS ボリュームとデータ ボリュームの両方が含まれます。 OS ボリュームのみを暗号化する場合は、-VolumeType パラメーターに "OS" を使用します。 

     ```azurepowershell-interactive
      $sequenceVersion = [Guid]::NewGuid();
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;
    ```
- **KEK を使用して実行中の VM を暗号化する:** この例では、OS と Data の両方のボリュームが含まれる -VolumeType パラメーターには "All" が使用されます。 OS ボリュームのみを暗号化する場合は、-VolumeType パラメーターに "OS" を使用します。

     ```azurepowershell-interactive
     $sequenceVersion = [Guid]::NewGuid();
     $rgName = 'MySecureRg';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > disk-encryption-keyvault パラメーターの値の構文は、/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] という完全な識別子の文字列です。</br> key-encryption-key パラメーターの値の構文は、 https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] という KEK への完全な URI です。 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Azure CLI を使用して新しく追加されたディスクで暗号化を有効にする
 暗号化を有効にするコマンドを実行する際に、Azure CLI コマンドによって、新しいシーケンス バージョンが自動的に提供されます。 この例では、"All" が volume-type パラメーターに使用されています。 OS ディスクのみ暗号化する場合は、volume-type パラメーターを OS に変更する必要があります。 PowerShell 構文とは異なり、CLI では暗号化を有効にする際にユーザーが一意のシーケンス バージョンを指定する必要はありません。 CLI では独自の一意のシーケンス バージョン値が自動的に生成され、使用されます。   

-  **実行中の VM を暗号化する:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **KEK を使用して実行中の VM を暗号化する:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "All"
     ```


## <a name="disable-encryption"></a>暗号化を無効にする
Azure PowerShell、Azure CLI、または Resource Manager テンプレートを使用して暗号化を無効にすることができます。 OS とデータ ディスクの両方が暗号化されている場合は、Windows VM におけるデータ ディスクの暗号化の無効化が想定どおりに機能しません。 代わりに、すべてのディスクで暗号化を無効にしてください。

- **Azure PowerShell を使用してディスク暗号化を無効にする:** 暗号化を無効にするには、[Disable-AzureRmVMDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) コマンドレットを使用します。 
     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM' -VolumeType "all"
     ```

- **Azure CLI を使用して暗号化を無効にする:** 暗号化を無効にするには、[az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) コマンドを使用します。 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type "all"
     ```
- **Resource Manager テンプレートを使用して暗号化を無効にする:** 

    1. [実行中の Windows VM でディスク暗号化を無効にする](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm-without-aad)ためのテンプレートで **[Azure に配置する]** をクリックします。
    2. サブスクリプション、リソース グループ、場所、VM、ボリュームの種類、法律条項、および契約を選択します。
    3.  **[購入]** をクリックして、実行中の Windows VM でディスク暗号化を無効にします。 

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [Linux で Azure Disk Encryption を有効にする](azure-security-disk-encryption-linux.md)
