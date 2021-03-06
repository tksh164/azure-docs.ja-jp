
ディスクの詳細については、[About Disks and VHDs for Virtual Machines (Virtual Machines 用のディスクと VHD について)](../articles/virtual-machines/virtual-machines-linux-about-disks-vhds.md) を参照してください。

<a id="attachempty"></a>

## Attach an empty disk
1. Azure CLI を開いて、[Azure サブスクリプションに接続](../articles/xplat-cli-connect.md)します。Azure サービス管理モード (`azure config mode asm`) であることを確認します。
2. 次のように「`azure vm disk attach-new`」と入力し、新しいディスクを作成して接続します。*TestVM* は Linux 仮想マシンの名前で置き換え、ディスクのサイズを GB 単位で指定します (この例では 100 GB)。
   
       azure vm disk attach-new TestVM 100
3. データ ディスクが作成されて接続されると、次のように `azure vm disk list <virtual-machine-name>` の出力に表示されます。
   
     $ azure vm disk list TestVM
     info:    Executing command vm disk list
   
   * Fetching disk images
   * Getting virtual machines
   * Getting VM disks
     data:    Lun  Size(GB)  Blob-Name                         OS
     data:    ---  --------  --------------------------------  -----
     data:         30        TestVM-2645b8030676c8f8.vhd  Linux
     data:    0    100       TestVM-76f7ee1ef0f6dddc.vhd
     info:    vm disk list command OK

<a id="attachexisting"></a>

## 既存のディスクの接続
既存のディスクを接続する場合は、ストレージ アカウントで利用できる .vhd を持っている必要があります。

1. Azure CLI を開いて、[Azure サブスクリプションに接続](../articles/xplat-cli-connect.md)します。Azure サービス管理モード (`azure config mode asm`) であることを確認します。
2. 接続する VHD が Azure サブスクリプションに既にアップロードされているかどうかを確認します。
   
     $azure vm disk list
     info:    Executing command vm disk list
   
   * Fetching disk images
     data:    Name                                          OS
     data:    --------------------------------------------  -----
     data:    myTestVhd                                     Linux
     data:    TestVM-ubuntuVMasm-0-201508060029150744  Linux
     data:    TestVM-ubuntuVMasm-0-201508060040530369
     info:    vm disk list command OK
3. 使用するディスクが見つからない場合は、`azure vm disk create` または `azure vm disk upload` を使用してローカル VHD をサブスクリプションにアップロードできます。`disk create` の例は次のようになります。
   
       $azure vm disk create myTestVhd2 .\TempDisk\test.VHD -l "East US" -o Linux
       info:    Executing command vm disk create
       + Retrieving storage accounts
       info:    VHD size : 10 GB
       info:    Uploading 10485760.5 KB
       Requested:100.0% Completed:100.0% Running:   0 Time:   25s Speed:    82 KB/s
       info:    Finishing computing MD5 hash, 16% is complete.
       info:    https://mystorageaccount.blob.core.windows.net/disks/test.VHD was
       uploaded successfully
       info:    vm disk create command OK
   
   `azure vm disk upload` を使用して、特定のストレージ アカウントに VHD をアップロードすることもできます。Azure Virtual Machine データ ディスクの管理用コマンドの詳細については、[こちら](../articles/virtual-machines-command-line-tools.md#commands-to-manage-your-azure-virtual-machine-data-disks)を参照してください。
4. ここで、必要な VHD を仮想マシンに接続します。
   
       $azure vm disk attach TestVM myTestVhd
       info:    Executing command vm disk attach
       + Getting virtual machines
       + Adding Data-Disk
       info:    vm disk attach command OK
   
   *TestVM* は仮想マシンの名前に、*myTestVhd* は目的の VHD に置き換えてください。
5. ディスクが仮想マシンに接続されたことを確認するには、`azure vm disk list <virtual-machine-name>` を使用します。
   
     $azure vm disk list TestVM
     info:    Executing command vm disk list
   
   * Fetching disk images
   * Getting virtual machines
   * Getting VM disks
     data:    Lun  Size(GB)  Blob-Name                         OS
     data:    ---  --------  --------------------------------  -----
     data:         30        TestVM-2645b8030676c8f8.vhd  Linux
     data:    1    10        test.VHD
     data:    0    100        TestVM-76f7ee1ef0f6dddc.vhd
     info:    vm disk list command OK

> [!NOTE]
> データ ディスクを追加した後、仮想マシンがストレージとしてそのディスクを使用できるように、仮想マシンにログオンしてディスクを初期化する必要があります (手順については下記を参照)。
> 
> 

