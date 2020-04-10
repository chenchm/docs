## Engine接口整理

EngineApi接口的根路径为**IP:PORT/acloudageks/api**

## 1.服务接口

## 1.1.虚拟机

### 1.1.1. 添加虚拟机

 请求参数

|          参数名          |         类型          | 位置 |                             说明                             |
| :----------------------: | :-------------------: | :--: | :----------------------------------------------------------: |
|          clone           |        Boolean        | url  | 可选参数，指定新建的虚拟机是否独立于模板。从模板创建虚拟机时，虚拟机的磁盘取决于模板的磁盘，它们使用写时复制机制，因此只有与模板之间的差异才占用实际存储空间。 如果指定了此参数且值为true，则将克隆创建的虚拟机的磁盘，并且与模板无关。 |
|    clone_permissions     |        Boolean        | url  |         可选参数，指定模板权限是否复制到新建的虚拟机         |
|           name           |        String         | body |                      必要参数，虚拟机名                      |
|         cluster          |     ClusterModel      | body | 必要参数，指定虚拟机所属的集群，需要指定集群的id或者name字段 |
|         template         |     TemplateModel     | body | 可选参数，代表通过模板创建虚拟机时选择的虚拟机模板，需要指定id字段或者name字段。 |
|        snapshots         |       Snapshots       | body | 可选参数，代表通过快照创建虚拟机时，指定的虚拟机快照。Snapshots是Snapshot集合的映射，只持有一个snapshots字段是Snapshot集合的引用，只需要指定Snapshot对象的id字段。Snapshot集合中只要一个Snapshot对象即可，即使传入多个也只会使用第一Snapshot对象。 |
|      initialization      |  InitializationModel  | body | 可选参数，代表虚拟机几个方面：1.通过配置创建虚拟机时，指定配置的内容。需要指定InitializationModel的configuration字段，configuration字段是ConfigurationModel的引用，它由2个字段组成：type字段代表配置的类型，只有唯一的值OVF；data字段代表配置的内容，即虚拟机的xml信息。另外可以指定InitializationModel的regenerateIds字段，表示是否为创建的虚拟机生成新的id。2.虚拟机初始化相关的内容，这一部分内容太多了，暂时先不展开。 |
|          memory          |         Long          | body | 可选参数，表示虚拟机的内存大小，单位为b。当该值为空或者为0时，使用默认大小10240MB即1G。 |
|      NumOfIoThreads      |        Integer        | body |               可选参数，表示虚拟机的IO线程数量               |
|       description        |        String         | body |                     可选参数，虚拟机描述                     |
|           cpu            |       CpuModel        | body | 可选参数，CpuModel有以下几个字段：1.topology字段是CpuTopologyModel的引用，代表了cpu的拓扑结构。CpuTopologyModel的几个字段：cores字段代表了每个虚拟机插槽的内核数；sockets字段代表了虚拟机cpu插槽数量；threads字段代表每个内核的cpu内核的线程数量，可选值只有1和2。2.architecture字段，代表集群cpu架构。可选的cpu架构有x86_64和ppc64。3.mode字段，代表虚拟机cpu模式，可选值有CUSTOM、HOST_MODEL、HOST_PASSTHROUGH。mode字段为HOST_PASSTHROUGH时，表示启用PASSTHROUGH主机CPU。4.cpuTune字段，代表cpu固定拓扑结构参数，是CpuTuneModel对象的引用。CpuTuneModel只持有vcpuPins字段，是VcpuPins对象的引用。VcpuPins对象是VcpuPin集合的映射，其vcpuPins字段是VcpuPin对象列表的引用。VcpuPin有以下2个字段：cpuSet字段，为String格式，作用未知；vcpu字段，为Integer格式，作用未知。 |
|     highAvailability     | HighAvailabilityModel | body | 可选参数，代表虚拟机高可用相关的配置，其中enabled字段表示是否开启虚拟机高可用；priority字段表示虚拟机迁移的优先级，为Interge类型。 |
|         display          |     DisplayModel      | body | 可选参数，DisplayModel代表虚拟机的显示模型，有以下几个字段：1.type字段，代表虚拟机使用的图形协议，可选的图形协议选项有VNC和SPICE，当设置了该字段虚拟的视频类型（defaultDisplayType）会被置为空，由后台决定。2.monitors字段，代表监视器数量。3.singleQxlPci字段，表示是否为单个PCI接口。4.smartcardEnabled字段，表示是否启用智能卡。5.allowOverride字段，代表是否允许控制台重连（不知道啥意思）。在界面中没有对应的选项，与虚拟机是服务器还是桌面相关，当虚拟机为服务器时该会被设为true。6.keyboardLayout字段，代表vnc键盘布局（作用未知），在界面中该选项被隐藏，可选内容见`vdc_options`表的`VncKeyboardLayoutValidValues`选项。7.fileTransferEnabled字段，表示是否启用SPICE文件转换（作用未知），在界面中该选项被隐藏。8.copyPasteEnabled字段，表示是否启用SPICE剪贴板复制和粘贴（作用未知），在界面中该选项被隐藏。9.disconnectAction字段，代表控制台连接中断以后虚拟机的行为，可选值有：NONE（无操作）、LOCK_SCREEN（锁定屏幕）、LOGOUT（用户登出）、REBOOT（重启虚拟机）、SHUTDOWN（关闭虚拟机）。 |
|    migrationDowntime     |        Integer        | body |  可选参数，代表虚拟机自定义的迁移下线时间，为Interge类型。   |
|        migration         | MigrationOptionsModel | body | 可选参数，MigrationOptionsModel的几个属性映射了集群虚拟机迁移策略的3个方面。1.**policy**字段是MigrationPolicyModel的引用，代表集群的迁移策略，集群可选的迁移策略一共有3种：Legacy（id为00000000-0000-0000-0000-000000000000）、Minimal downtime（id为80554327-0569-496b-bdeb-fcbbf52b827b）和Suspend workload if needed（id为80554327-0569-496b-bdeb-fcbbf52b827c），其中Legacy只设置了最大的迁移数量为2，而另外2种选项的详细配置见**vdc_optins**表的`MigrationPolicies`选项，迁移策略需要设置MigrationPolicyModel对象的id字段，默认值为null；2.**autoConverge**字段和**compressed**字段，分别表示是否自动聚合迁移以及是否启用迁移压缩。只有在迁移策略为`Legacy`时，这两个字段在虚拟机迁移过程中才会生效，它们的可选值有三种：TRUE、FALSE、INHERIT（表示从全局设置继承），这两个字段的默认值为null。 |
|      customCpuModel      |        String         | body | 可选参数，代表虚拟机的cpu类型，需要注意的是虚拟机cpu类型，必须与虚拟机所在集群的cpu类型为同一产商，且代数必须等于或低于集群cpu类型。 |
|  customEmulatedMachine   |        String         | body |                可选参数，代表虚拟机的仿真机。                |
|       memoryPolicy       |   MemoryPolicyModel   | body | 可选参数，MemoryPolicyModel对象代表虚拟机的内存策略，其guaranteed字段代表虚拟机保证的物理内存，单位为b。 |
|            os            | OperatingSystemModel  | body | 可选参数，OperatingSystemModel对象代表与系统引导有关的参数。以下几个字段会影响模板的创建：1.boot字段是BootModel对象的引用，代表引导设备序列。BootModel只持有devices字段，是BootDeviceValues对象的引用。BootDeviceValues对象是BootDevice的集合映射，只持有BootDevice集合的引用bootDevice字段。BootDevice代表虚拟机的引导设备序列，可选值有：CDROM（光驱）、HD（硬盘）和NETWORK（网络）。列表中BootDevice各个值的排列顺序，代表虚拟机引导设备顺序，如果出现重复的值，会先去重。2.type字段，代表操作系统类型，例如other_linux，windows_7x64等，可选值参考`share/ovirt-engine/conf/osinfo-defaults.properties`文件中的配置。3.kernel字段，代表Linux引导选项中的内核路径，因此只有在操作系统类型是linux时才会生效，该字段填写的是linux内核在iso域的路径。4.initrd字段，代表Linux引导选项中的initrd路径，因此只有在操作系统类型是linux时才会生效，该字段填写的是initrd在iso域的路径。5.cmdline字段，代表虚拟机自定义的内核参数，格式目前不知道。 |
|     bootMenuEnabled      |        Boolean        | body |               可选参数，表示是否启用引导菜单。               |
|         timeZone         |     TimeZoneModel     | body | 可选参数，TimeZoneModel对象代表了虚拟机的时区设置。其name字段，代表虚拟机时区，可选值详见`TimeZoneType`枚举类的`initializeTimeZoneList`方法返回的map的key值。 |
|          origin          |        String         | body | 可选参数，代表虚拟机来源（不会手动设置吧？），可选值有VMWARE、XEN、OVIRT、EXTERNAL、KVM。 |
|        stateless         |        Boolean        | body |              可选参数，表示虚拟机是否为无状态。              |
|     deleteProtected      |        Boolean        | body |            可选参数，表示虚拟机是否启用删除保护。            |
|        ssoEnabled        |        Boolean        | body | 可选参数，表示虚拟机是否启用单点登入（即是否使用GuestAgent）。 |
|           type           |        String         | body | 可选参数，代表虚拟机类型，可选值有：DESKTOP（桌面）、SERVER（服务器）。 |
|     tunnelMigration      |        Boolean        | body | 可选参数，代表虚拟机是否启用通道迁移（作用未知），界面没找到与之对应的选项。 |
|       startPaused        |        Boolean        | body |           可选参数，代表虚拟机是否以暂停模式启动。           |
|     customProperties     |   CustomProperties    | body | 可选参数，CustomProperties是虚拟机自定义参数集合的映射，持有唯一的customProperties字段是CustomPropertyModel列表的引用。CustomPropertyModel代表了每一个自定义参数，其中name字段代表参数名，value字段代表参数值，regexp代表正则表达式。 |
|          quota           |      QuotaModel       | body | 可选参数，QuotaModel代表了虚拟机的配额，配额具体的作用不清楚，界面上也没找到对应的内容，这里只需要指定id字段即可。 |
| useLatestTemplateVersion |        Boolean        | body | 可选参数，表示通过模板创建虚拟机时是否使用模板的最新子版本。 |
|     placementPolicy      |   VmPlacementPolicy   | body | 可选参数，代表虚拟机的安置策略（即虚拟机与主机之间的关系），VmPlacementPolicy有以下几个字段：1.affinity字段，代表主机的迁移模式，可选值有MIGRATABLE（允许手动迁移和自动迁移）、USER_MIGRATABLE（允许手动迁移）、PINNED（不运行迁移）。2.hosts字段，是Hosts的引用，代表虚拟机可用的主机列表。Hosts对象是的HostModel集合映射，只持有一个HostModel集合的引用hosts字段。这里只需要指定HostModel的id字段或者name字段，host必须为虚拟机所在集群的主机。当HostModel集合为空（或者null）时，表示虚拟机能运行在任一主机上。 |
|           usb            |       UsbModel        | body | 可选参数，代表虚拟机模板的usb策略，UsbModel具有2个字段：1.enabled字段，表示是否启用usb策略，只有该字段为true时，其他字段才能生效。2.type字段，代表usb策略类型，可选值有LEGACY和NATIVE。在创建方式为虚拟机模板时有效。 |
|       numaTuneMode       |     NumaTuneMode      | body | 可选参数，代表Numa的Tune模式，可选值有：STRICT（严格）、INTERLEAVE（交错）、PREFERRED（首选）。 |
|     diskAttachments      |    DiskAttachments    | body | 可选参数，代表对指定的模板或者快照中原有的磁盘信息进行修改。DiskAttachments对象代表虚拟机附加磁盘列表，DiskAttachments只拥有diskAttachments一个字段，是DiskAttachment集合的引用，DiskAttachment对象的disk字段所指向的DiskModel才是真正的磁盘信息。当需要修改磁盘信息时必须指定的字段与创建方式有关，当通过模板创建虚拟机时需要指定DiskModel的id字段来指定要修改的磁盘，但是如果通过快照创建虚拟机时需要指定DiskModel的imageId字段。可修改磁盘信息，例如DiskModel的name字段表示要修改的磁盘名；DiskModel的format字段表示要修改的磁盘格式，可选值包括COW和RAW；DiskModel的storageDomains字段是StorageDomains对象的引用，表示指定磁盘的存储域，StorageDomains对象只持有一个StorageDomainModel集合的引用storageDomains字段，指定磁盘存储域时只需要指定集合中一个StorageDomainModel的id字段即可。 |

说明：创建虚拟机接口，可以通过模板、快照或者xml信息创建虚拟机，同时也可以指定其他参数对相应模板或者快照信息进行修改。这三者的优先级为：`xml > 快照 > 模板`。

- 通过模板创建虚拟机，这种方式必须提供模板的`id`或者名字：

请求：

```http
POST /acloudageks/api/vms
Body: 
{
    "name": "myvm",
    "template": {
        "name": "Blank"
    },
    "cluster": {
        "name": "Default"
    }
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "VmModel [cluster.id|name] required for add",
    "reason": "Incomplete parameters"
}
```

- 通过快照创建虚拟机，这种方式必须提供快照id

请求：

```http
POST /acloudageks/api/vms
Body: 
{
    "name": "myvm",
    "snapshots": {
    	"snapshots": [
        	{
            	"id": "bc6ccc8b-278c-4d87-a945-5728b45690d5"
        	}
    	]
    },
    "cluster": {
        "name": "Default"
    }
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "VmModel [cluster.id|name] required for add",
    "reason": "Incomplete parameters"
}
```

从模板或快照创建虚拟机时，通常通常要明确指出要在哪个存储域中创建虚拟机磁盘。 如果虚拟机是从模板创建的，则可以通过添加diskAttachment参数来说明映射关系：

```json
{
    ...
    "diskAttachments": {
        "diskAttachments": [
            "disk": {
            	"id": "bc6ccc8b-1234-4d87-a945-5728b45690d1",
            	"storageDomains": {
            		"storageDomains": [
            			{
            				"id": "9cb6cb0a-cf1d-41c2-92ca-5a6d665649c9"
           				 }
       				 ]
            	}
            }
        ]
    }
}
```

从快照创建虚拟机时磁盘配置略微有些不同，这时需要使用imageId属性来替换id属性

```json
{
    ...
    "diskAttachments": {
        "diskAttachments": [
            "disk": {
            	"imageId": "bc6ccc8b-1234-4d87-a945-5728b45690d1",
            	"storageDomains": {
            		"storageDomains": [
            			{
            				"id": "9cb6cb0a-cf1d-41c2-92ca-5a6d665649c9"
           				 }
       				 ]
            	}
            }
        ]
    }
}
```

可以在请求参数中指定其他虚拟机参数，例如可以添加具有2 GiB RAM和附加说明的桌面型虚拟机，可以添加以下参数：

```json
{
    "name": "myvm",
    "description": "My Desktop Virtual Machine",
    "type": "desktop"
    "memory": 2147483648
    ...
}
```

### 1.1.2.获得虚拟机列表

 请求参数

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
|  all_content   | Boolean | url  | 可选参数，指定是否在response是否应该包括所有的虚拟机属性（貌似没作用） |
| case_sensitive | Boolean | url  | 可选参数，与search参数一起使用，指定在执行search参数时是否考虑大小写 |
|     filter     | Boolean | url  |     可选参数，指定返回的虚拟机信息是否根据用户权限被过滤     |
|      max       | Integer | url  |              可选参数，指定返回的最大虚拟机数量              |
|     search     | String  | url  |    可选参数，查询约束条件约束返回信息，例如name="myname"     |

请求：

```http
GET /acloudageks/api/vms
```

请求结果：成功

### 1.1.3.取消迁移虚拟机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/cancelmigration
Body: 
{
	
}
```

请求结果：待测试

### 1.1.4.克隆虚拟机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |
|   vm   | VmModel | body |   必要参数，需要指定克隆的新虚拟机的名字   |

请求：

```http
POST /acloudageks/api/vms/{id}/clone
Body: 
{
	"vm": {
        "name": "myClone"
	}
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "ActionModel [vm.name] required for doClone",
    "reason": "Incomplete parameters"
}
```

### 1.1.5.提交预览镜像

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/commitsnapshot
Body: 
{
	
}
```

请求结果：待测试

### 1.1.6.从虚拟机池中分离虚拟机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/detach
Body: 
{
	
}
```

请求结果：成功

### 1.1.7.导出虚拟机到存储域

请求参数

|      参数名      |        类型        | 位置 |                             说明                             |
| :--------------: | :----------------: | :--: | :----------------------------------------------------------: |
|      async       |      Boolean       | body |          可选参数，表示是否通过异步的方式调用该请求          |
| discardSnapshots |      Boolean       | body | 可选参数，表示导出虚拟机到导出域时是否需要折叠虚拟机所有的快照 |
|    exclusive     |      Boolean       | body | 可选参数，表示导出虚拟机到导出域时如果该虚拟机已经在导出域存在，是否覆盖已经存在导出域的虚拟机 |
|  storageDomain   | StorageDomainModel | body |          必要参数，需要指定存储域的id字段或者id字段          |

请求：

```http
POST /acloudageks/api/vms/{id}/export
Body: 
{
	"storageDomain": {
        "name": "myexport"
	},
	"exclusive": true,
	"discardSnapshots": true
}
```

请求结果：待测试

### 1.1.8.冻结虚拟机文件系统

当为正在运行的虚拟机拍摄活动快照时，`QEMU guest agent`会冻结虚拟机的文件系统。通常，这个是由管理系统自动完成的，但虚拟机在使用`OpenStack Volume(Cinder)`磁盘时，这个操作必须进行手动调用。

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/freezefilesystems
Body: 
{
	
}
```

请求结果：待测试

### 1.1.9.获得指定的虚拟机信息

请求参数

|   参数名    |  类型   | 位置 |                             说明                             |
| :---------: | :-----: | :--: | :----------------------------------------------------------: |
| all_content | Boolean | url  | 可选参数，指定是否在response是否应该包括所有的虚拟机属性（貌似没作用） |
|   filter    | Boolean | url  |     可选参数，指定返回的虚拟机信息是否根据用户权限被过滤     |
|  next_run   | Boolean | url  | 可选参数，指定返回的结果是当前正在运行虚拟机的虚拟机信息，还是已经执行了修改，但是在虚拟机下次重启才生效的虚拟机信息 |

请求：

```http
GET /acloudageks/api/vms/{id}
```

请求结果：成功

### 1.1.10.启用用户自动登入虚拟机

启动自动用户登录以从外部控制台访问虚拟机。用户需要虚拟机的适当用户权限，才能从外部控制台访问虚拟机。

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/logon
Body: 
{
	
}
```

请求结果：成功

### 1.1.11.维护虚拟机

在ovirt中此接口用于将作为主机的虚拟机执行维护操作（嵌套虚拟化），在我们的系统中不支持，所以调用该接口会直接抛出异常。

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/maintenance
Body: 
{
	
}
```

请求结果：废弃的接口

### 1.1.12.虚拟机迁移

请求参数

| 参数名  |     类型     | 位置 |                             说明                             |
| :-----: | :----------: | :--: | :----------------------------------------------------------: |
|  async  |   Boolean    | body |          可选参数，表示是否通过异步的方式调用该请求          |
| cluster | ClusterModel | body | 可选参数，指定虚拟机要迁移到的集群，默认情况下系统会将虚拟机迁移到同一集群中的另一台主机上。需要通过id字段来指定集群。 |
|  force  |   Boolean    | body |   可选参数，强制迁移虚拟机，即使虚拟机可能被定义为不可迁移   |
|  host   |  HostModel   | body | 可选参数，指定虚拟机迁移到的主机，默认情况下系统会将虚拟机迁移到同一集群中的另一台主机上。需要通过name字段或者id字段来指定主机。 |

请求：

```http
POST /acloudageks/api/vms/{id}/migrate
Body: 
{
	"host": {
        "id": "2ab5e1da-b726-4274-bbf7-0a42b16a0fc3"
	}
}
```

请求结果：待测试

### 1.1.13.快照预览

请求参数：

|    参数名     |     类型      | 位置 |                             说明                             |
| :-----------: | :-----------: | :--: | :----------------------------------------------------------: |
|     async     |    Boolean    | body |          可选参数，表示是否通过异步的方式调用该请求          |
|     disks     |     Disks     | body | 可选参数，用于指定执行快照预览的磁盘，Disks对象为DiskModel集合的映射，Disks对象只持有DiskModel集合的引用disks字段，需要DiskModel的完整的信息，默认对所有磁盘进行快照预览。 |
| restoreMemory |    Boolean    | body |        可选参数，是否恢复快照的内存信息，默认为false         |
|   snapshot    | SnapshotModel | body |       必要参数，表示要进行预览的快照，需要指定快照id。       |

说明：参数中DiskModel的完整的信息不是很合理，这里应该改为只需要指定id字段和imageId字段。`ovirt4.3`中的逻辑也是如此。

请求：

```http
POST /acloudageks/api/vms/{id}/previewsnapshot
Body: 
{
	"snapshot": {
        "id": "2ab5e1da-b726-4274-bbf7-0a42b16a0fc3"
	}
}
```

请求结果：待测试

### 1.1.14.重启虚拟机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/reboot
Body: 
{
	
}
```

请求结果：成功

### 1.1.15.删除虚拟机

请求参数

|   参数名    |  类型   | 位置 |                             说明                             |
| :---------: | :-----: | :--: | :----------------------------------------------------------: |
| detach_only | Boolean | url  | 可选参数，表示删除虚拟机时，是否只分离虚拟机的附加磁盘，而不是删除磁盘。 |
|    force    | Boolean | url  |                 可选参数，是否强制删除虚拟机                 |

锁定的虚拟机和附加了锁定磁盘的虚拟机在没有附加fore参数时无法被删除。

请求：

```http
DELETE /acloudageks/api/vms/{id}
```

请求结果：成功

### 1.1.16.mac地址重排

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/reordermacaddresses
Body: 
{
	
}
```

请求结果：成功

### 1.1.17.关闭虚拟机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/shutdown
Body: 
{
	
}
```

请求结果：成功

### 1.1.18.启动虚拟机

请求参数

|    参数名    |  类型   | 位置 |                             说明                             |
| :----------: | :-----: | :--: | :----------------------------------------------------------: |
|    async     | Boolean | body |          可选参数，表示是否通过异步的方式调用该请求          |
|    pause     | Boolean | body |    可选参数，表示虚拟机是否以暂停模式启动，默认值为false     |
|  useSysprep  | Boolean | body | 可选参数，仅在RunVmOnce模式有效，表示虚拟机是否以sysprep方式启动（仅支持windows），与useCloudInit参数冲突，默认值为false |
| useCloudInit | Boolean | body | 可选参数，仅在RunVmOnce模式有效，表示虚拟机是否以clouldinit方式启动（仅支持linux），与useCloudInit参数冲突，默认值为false |
|      vm      | VmModel | body | 功能区分参数，用于启用RunVmOnce模式，当指定该参数时，表示以RunVmOnce模式启动虚拟机。VmModel对象中对RunVmOnce模式产生影响的字段有以下几个：1.stateless字段，表示虚拟机是否为无状态。2.display字段，是DisplayModel对象的引用代表虚拟机的显示模型，以下几个字段影响虚拟机启动：keyboardLayout字段，代表vnc键盘布局（作用未知），在界面中该选项被隐藏，可选内容见`vdc_options`表的`VncKeyboardLayoutValidValues`选项；type字段，代表虚拟机启动使用的图形协议，可选的图形协议选项有VNC和SPICE。3.os字段，是OperatingSystemModel对象的引用代表与系统引导有关的参数，以下几个字段影响虚拟机启动：**boot**字段是BootModel对象的引用，代表引导设备序列。BootModel只持有devices字段，是BootDeviceValues对象的引用。BootDeviceValues对象是BootDevice的集合映射，只持有BootDevice集合的引用bootDevice字段。BootDevice代表虚拟机的引导设备序列，可选值有：CDROM（光驱）、HD（硬盘）和NETWORK（网络）。列表中BootDevice各个值的排列顺序，代表虚拟机引导设备顺序，如果出现重复的值，会先去重；**kernel**字段，代表Linux引导选项中的内核路径，因此只有在操作系统类型是linux时才会生效，该字段填写的是linux内核在iso域的路径；**initrd**字段，代表Linux引导选项中的initrd路径，因此只有在操作系统类型是linux时才会生效，该字段填写的是initrd在iso域的路径；**cmdline**字段，代表虚拟机自定义的内核参数，格式目前不知道。4.cdroms字段，是Cdroms对象是CdromModel集合映射，只持有CdromModel集合的引用cdroms字段。CdromModel对象代表虚拟机光盘，只有一个file字段，file字段是FileModel的引用，需要指定FileModel对象的id字段来指定光盘文件的路径。5.floppies字段，是Floppies对象是FloppyModel集合映射，只持有FloppyModel集合的引用floppies字段。FloppyModel对象代表虚拟机软盘，只有一个file字段，file字段是FileModel的引用，需要指定FileModel对象的id字段来指定软盘文件的路径。6.customProperties字段，是CustomProperties对象的引用代表虚拟机自定义参数集合的映射，持有唯一的customProperties字段是CustomPropertyModel列表的引用。CustomPropertyModel代表了每一个自定义参数，其中name字段代表参数名，value字段代表参数值，regexp代表正则表达式。7.bootMenuEnabled字段，表示是否启用引导菜单。8.domain字段，DomainModel对象的引用，代表域相关的信息，只在SysPrep模式生效。DomainModel对象的name字段代表进行认证的域的域名；DomainModel对象的user字段，是UserModel的引用，代表域用户信息。UserModel的name字段代表域用户名，UserModel的password字段代表域用户密码。9.customCpuModel字段，代表虚拟机的cpu类型，需要注意的是虚拟机cpu类型，必须与虚拟机所在集群的cpu类型为同一产商，且代数必须等于或低于集群cpu类型。10.customEmulatedMachine字段，代表虚拟机的仿真机。11.placementPolicy字段，代表虚拟机的安置策略（即虚拟机与主机之间的关系），以下几个字段影响虚拟机启动：hosts字段，是Hosts的引用，代表虚拟机可用的主机列表。Hosts对象是的HostModel集合映射，只持有一个HostModel集合的引用hosts字段。这里只需要指定HostModel的id字段或者name字段，host必须为虚拟机所在集群的主机。当HostModel集合为空（或者null）时，表示虚拟机只能运行在固定主机上。12.initialization字段，代表虚拟机初始化相关的信息，参数太多了，暂时先不深入。 |

请求：

```http
POST /acloudageks/api/vms/{id}/start
Body: 
{
	
}
```

请求结果：成功

### 1.1.19.虚拟机断电

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/stop
Body: 
{
	
}
```

请求结果：成功

### 1.1.20.挂起虚拟机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/suspend
Body: 
{
	
}
```

请求结果：成功

### 1.1.21.解冻虚拟机文件系统

当为正在运行的虚拟机拍摄活动快照时，`QEMU guest agent`会冻结和解冻虚拟机的文件系统。通常，这个是由管理系统自动完成的，但虚拟机在使用`OpenStack Volume(Cinder)`磁盘时，这个操作必须进行手动调用。

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/thawfilesystems
Body: 
{
	
}
```

请求结果：待测试

### 1.1.22.生成显示令牌

用于生成有时效性的身份验证令牌来访问启动状态的虚拟机显示窗口。

请求参数

| 参数名 |    类型     | 位置 |                             说明                             |
| :----: | :---------: | :--: | :----------------------------------------------------------: |
| async  |   Boolean   | body |          可选参数，表示是否通过异步的方式调用该请求          |
| ticket | TicketModel | body | 可选参数，包括value值和有效时间两个字段，而两个字段都是可选的，其中过期时间的默认值为2个小时，value值默认值为随机字符串 |

请求：

```http
POST /acloudageks/api/vms/{id}/ticket
Body: 
{
	"ticket": {
        "value": "abcd12345",
        "expiry": 120
	}
}
```

请求结果：成功

### 1.1.23.撤销预览镜像

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{id}/undosnapshot
Body: 
{
	
}
```

请求结果：待测试

### 1.1.24.更新虚拟机

请求参数

|  参数名  |  类型   | 位置 |                             说明                             |
| :------: | :-----: | :--: | :----------------------------------------------------------: |
|  filter  | Boolean | url  |     可选参数，指定返回的虚拟机信息是否根据用户权限被过滤     |
| next_run | Boolean | url  | 可选参数，指定更新的信息是在虚拟机下次启动时再生效，还是对虚拟机立即生效，该参数的默认值为false，所以更新的信息会立即生效 |
|    ..    |   ...   | body |    需要更新的虚拟机信息，与1.1.1中新建虚拟机参数大致相同     |

请求：

```http
PUT /acloudageks/api/vms/{id}
Body: 
{
	"description": "测试更新虚拟机信息"
}
```

请求结果：待测试

## 1.2.虚拟机应用

### 1.2.1.获得虚拟机已安装应用程序列表

 请求参数

| 参数名 |  类型   | 位置 |                          说明                          |
| :----: | :-----: | :--: | :----------------------------------------------------: |
| filter | Boolean | url  | 可选参数，指定返回的应用程序信息是否根据用户权限被过滤 |
|  max   | Integer | url  |          可选参数，指定返回的最大应用程序数量          |

请求：

```http
GET /acloudageks/api/vms/{id}/applications
```

请求结果：成功

### 1.2.2.获得虚拟机指定的已安装应用程序信息

 请求参数

| 参数名 |  类型   | 位置 |                          说明                          |
| :----: | :-----: | :--: | :----------------------------------------------------: |
| filter | Boolean | url  | 可选参数，指定返回的应用程序信息是否根据用户权限被过滤 |

请求：

```http
GET /acloudageks/api/vms/{vmid}/applications/{appid}
```

请求结果：待测试

## 1.3.虚拟机光盘

### 1.3.1.获得虚拟机的光盘设备列表

请求参数

| 参数名 |  类型   | 位置 |                 说明                 |
| :----: | :-----: | :--: | :----------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大光盘设备数量 |

请求：

```http
GET /acloudageks/api/vms/{id}/cdroms
```

请求结果：成功

### 1.3.2.获得指定的光盘设备信息

请求参数

| 参数名  |  类型   | 位置 |                             说明                             |
| :-----: | :-----: | :--: | :----------------------------------------------------------: |
| current | Boolean | url  | 可选参数，由于对光盘信息的修改可以立即生效，也可以在虚拟机下次启动时生效，所以该值表示获得当前正在运行的虚拟机光盘信息，还是虚拟机下次启动时的光盘信息，默认值为false |

请求：

```http
GET /acloudageks/api/vms/{vmid}/cdroms/{cdid}
```

请求结果：成功

### 1.3.3.更新虚拟机光盘信息

更改和弹出光盘都使用该接口进行操作，通过设置的`FileModel`对象的id值来执行更改和弹出光盘操作。

请求参数：

| 参数名  |   类型    | 位置 |                             说明                             |
| :-----: | :-------: | :--: | :----------------------------------------------------------: |
|  file   | FileModel | body | 必要参数，通过设置file对象的id值，来更改或弹出磁盘。ISO域中必须存在与file对象id相对应的文件，当file对象id不为空时，挂载相应的iso文件；当id为空则表示弹出光盘。 |
| current |  Boolean  | url  | 可选参数，表示对光盘信息的修改是立即应用到当前运行的虚拟机上，还是在虚拟机下次启动时生效，默认值为false |

请求：

```http
PUT /acloudageks/api/vms/{vmid}/cdroms/{cdid}?current=true
Body: 
{
	"file": {
        "id": "boot2docker.iso"
	}
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "CdromModel [file] required for update",
    "reason": "Incomplete parameters"
}
```

## 1.4.虚拟机磁盘

与虚拟机磁盘相关的接口在整理时丢失了，所以需要在`VmResource`接口中添加以下接口方法才能使虚拟机磁盘接口生效。在`ovirt3.0`虚拟机磁盘相关的操作在此接口下执行，但是在`ovirt4.0`以后的虚拟机磁盘相关接口在磁盘附加（DiskAttachments）接口下执行，但是磁盘附加接口缺少部分操作（例如磁盘的激活和取消激活操作），并且接口参数以及返回对象的属性也发生了改变，将会在[1.5]()详细介绍。

```java
@Path("disks")
VmDisksResource getDisksResource();
```

### 1.4.1.添加虚拟机磁盘

添加磁盘接口实际上有2个功能，一个是附加现有磁盘到指定虚拟机，另一个是为虚拟机添加磁盘。两个功能通过是否传入`diskId`进行区分。

请求参数

|       参数名        |       类型       | 位置 |                             说明                             |
| :-----------------: | :--------------: | :--: | :----------------------------------------------------------: |
|         id          |      String      | body | 功能区分参数，当传入此参数时，表示调用磁盘附加功能，否则调用虚拟机添加磁盘功能。 |
|      bootable       |     Boolean      | body | 可选参数，表示是否作为将磁盘作为可引导磁盘，默认值为false。需要注意的是，每个虚拟机只能有一个可引导磁盘 |
|      interface      |      String      | body | 可选参数，表示磁盘接口类型，磁盘接口类型一共有三种：IDE、VIRTIO_SCSI、VIRTIO；默认的接口类型为VIRTIO |
|       active        |     Boolean      | body |          可选参数，表示是否激活磁盘，默认值为false           |
|      readOnly       |     Boolean      | body |         可选参数，表示是磁盘是否只读，默认值为false          |
|      snapshot       |  SnapshotModel   | body | 可选参数，仅在磁盘附加时有效，只需要指定SnapshotModel的id参数，来指定磁盘快照id。 |
|     lunStorage      | HostStorageModel | body | 仅在磁盘创建时有效，创建块设备（lun）的磁盘时，必须传入此参数，并且必须指定HostStorageModel的**logicalUnits**字段和**type**字段。logicalUnits字段的类型为LogicalUnits对象，LogicalUnits对象只有一个logicalUnits字段，是由LogicalUnitModel对象组成的列表；type是磁盘的存储类型，在创建lun时，可选的存储类型有ISCSI和FCP。当type为ISCSI时，LogicalUnitModel对象的address、target、port、id字段不能为空 |
|       format        |      String      | body | 仅在磁盘创建时有效，指定磁盘格式，创建非块设备磁盘时，必须指定此参数，可选的磁盘格式有RAW和COW。 |
|   provisionedSize   |       Long       | body | 仅在磁盘创建时有效，指定磁盘大小，单位为b，创建非块设备磁盘时，必须指定此参数。 |
|   storageDomains    |  StorageDomains  | body | 用于指定磁盘的存储域，在新建磁盘时必须指定此参数。StorageDomains对象只持有一个StorageDomainModel集合的引用storageDomains字段，指定磁盘存储域时只需要指定集合中一个StorageDomainModel的id字段即可，并且后台会根据id获得指定存储域并将磁盘的存储域类型设置为与指定存储域相同。 |
|        name         |      String      | body |  可选参数，磁盘别名，已废弃的字段，目前使用alias字段代替。   |
|        alias        |      String      | body |                     可选参数，磁盘别名。                     |
|   propagateErrors   |     Boolean      | body |        可选参数，表示是否传播错误到Guest，作用未知。         |
|   wipeAfterDelete   |     Boolean      | body |               可选参数，表示是否在删除后清理。               |
|     logicalName     |      String      | body |               可选参数，磁盘逻辑名，作用未知。               |
|     description     |      String      | body |                     可选参数，磁盘描述。                     |
|     lunStorage      | HostStorageModel | body |      可选参数，仅在磁盘类型为lun时有效，这里先不展开。       |
| usesScsiReservation |     Boolean      | body |        可选参数，仅在磁盘类型为lun时有效，作用未知。         |
|        sgio         |  ScsiGenericIO   | body | 可选参数，仅在磁盘类型为lun时有效，可选值有FILTERED和UNFILTERED，作用未知。 |
|       imageId       |      String      | body |                可选参数，代表磁盘所在卷的id。                |
|       sparse        |     Boolean      | body | 可选参数，表示磁盘是否为精简分配，true为精简分配，false为预分配。 |
|        quota        |    QuotaModel    | body |  可选参数，代表磁盘配额，只需要指定QuotaModel的id字段即可。  |
|     diskProfile     |   DiskProfile    | body | 可选参数，代表磁盘配置集，只需要指定DiskProfile的id字段即可。 |

请求：

- 附加磁盘

```http
POST /acloudageks/api/vms/{vmid}/disks/
Body: 
{
	"id": "3c0433b5-14b5-41ac-b2d5-3319b534b05e",
	"bootable": true,
	"interface": "IDE",
	"active": true,
	"readOnly": false,
	"snapshot": {
        "id": "d764f497-85a7-4585-b6f7-6fa3e4f8b085"
	}
}
```

请求结果：

未添加`snapshot`参数时请求成功，添加后请求失败。

失败原因：

```json
{
    "detail": "DiskModel [snapshot.id] required for attachDiskToVm",
    "reason": "Incomplete parameters"
}
```

- 新建磁盘并附加到虚拟机

请求：

```http
POST /acloudageks/api/vms/{vmid}/disks/
Body: 
{
	"bootable": true,
	"interface": "IDE",
	"active": false,
	"readOnly": true,
	"format": "RAW",
	"provisionedSize": 1073741824
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "DiskModel [provisionedSize|size, format] required for invoke0",
    "reason": "Incomplete parameters"
}
```

### 1.4.2.列出虚拟机所有磁盘

请求参数

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大虚拟机磁盘数量 |

请求：

```http
GET /acloudageks/api/vms/{vmid}/disks?max=1
```

请求结果：成功

### 1.4.3.激活虚拟机磁盘

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/disks/{diskId}/activate
Body: 
{
	
}
```

请求结果：成功

### 1.4.4.取消激活虚拟机磁盘

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/disks/{diskId}/deactivate
Body: 
{
	
}
```

请求结果：成功

### 1.4.5.获得虚拟机指定磁盘信息

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/disks/{diskId}
```

请求结果：成功

### 1.4.6.移动虚拟机磁盘

请求参数

|    参数名     |        类型        | 位置 |                           说明                           |
| :-----------: | :----------------: | :--: | :------------------------------------------------------: |
|     async     |      Boolean       | body |        可选参数，表示是否通过异步的方式调用该请求        |
|    filter     |      Boolean       | url  |    可选参数，指定返回的磁盘信息是否根据用户权限被过滤    |
| storageDomain | StorageDomainModel | body | 必要参数，需要指定StorageDomainModel的id字段或者name字段 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/disks/{diskId}/move
Body: 
{
	"storageDomain": {
        "id": "263612f2-d67b-4ce3-9c3e-af05fd4795bf"
	}
}
```

请求结果：待测试

### 1.4.7.分离虚拟机磁盘

请求参数：

无

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/disks/{diskId}
```

请求结果：成功

### 1.4.8.更新虚拟机磁盘信息

请求参数

|       参数名        |       类型       | 位置 |                             说明                             |
| :-----------------: | :--------------: | :--: | :----------------------------------------------------------: |
|        async        |     Boolean      | body |            可选，表示是否通过异步的方式调用该请求            |
|      bootable       |     Boolean      | body | 磁盘是否作为可引导磁盘。需要注意的是，每个虚拟机只能有一个可引导磁盘 |
|      interface      |      String      | body | 磁盘接口类型，磁盘接口类型一共有三种：IDE、VIRTIO_SCSI、VIRTIO |
|      readOnly       |     Boolean      | body |                        磁盘是否只读。                        |
|        name         |      String      | body |       磁盘别名，已废弃的字段，目前使用alias字段代替。        |
|   provisionedSize   |       Long       | body |                     磁盘大小，单位为b。                      |
|        alias        |      String      | body |                          磁盘别名。                          |
|       format        |      String      | body |            定磁盘格式，可选的磁盘格式有RAW和COW。            |
|   propagateErrors   |     Boolean      | body |               是否传播错误到Guest，作用未知。                |
|   wipeAfterDelete   |     Boolean      | body |                    表示是否在删除后清理。                    |
|     logicalName     |      String      | body |                    磁盘逻辑名，作用未知。                    |
|     description     |      String      | body |                          磁盘描述。                          |
|     lunStorage      | HostStorageModel | body |           仅在磁盘类型为lun时有效，这里先不展开。            |
| usesScsiReservation |     Boolean      | body |             仅在磁盘类型为lun时有效，作用未知。              |
|        sgio         |      String      | body | 仅在磁盘类型为lun时有效，可选值有FILTERED和UNFILTERED，作用未知。 |
|       imageId       |      String      | body |                       磁盘所在卷的id。                       |
|      snapshot       |  SnapshotModel   | body | 磁盘快照，只需要指定SnapshotModel的id字段，来指定磁盘快照id。 |
|       sparse        |     Boolean      | body |     磁盘是否为精简分配，true为精简分配，false为预分配。      |
|   storageDomains    |  StorageDomains  | body | 代表磁盘所在的存储域，StorageDomains对象只持有一个StorageDomainModel集合的引用storageDomains字段，指定磁盘存储域时只需要指定集合中一个StorageDomainModel的id字段即可。 |
|        quota        |    QuotaModel    | body |       磁盘配额模式，只需要指定QuotaModel的id字段即可。       |
|     diskProfile     |   DiskProfile    | body |       磁盘配置集，只需要指定DiskProfile的id字段即可。        |

请求：

```http
PUT /acloudageks/api/vms/{vmid}/disks/{diskId}
Body: 
{
	"interface": "VIRTIO",
	"readOnly": false,
	"provisionedSize": 2147483648,
	"name": "update"
}
```

请求结果：成功

## 1.5.虚拟机附加磁盘

与[1.4]()中的接口区别在于返回对象的属性不同，是`ovirt3.0`到`ovirt4.0`版本变迁的结果，与[2.8]()中接口相比缺少了部分的接口。

### 1.5.1.添加虚拟机附加磁盘

请求参数：

|  参数名   |   类型    | 位置 |                             说明                             |
| :-------: | :-------: | :--: | :----------------------------------------------------------: |
| bootable  |  Boolean  | body | 可选参数，表示是否作为将磁盘作为可引导磁盘，默认值为false。需要注意的是，每个虚拟机只能有一个可引导磁盘 |
| interface |  String   | body | 必要参数，表示磁盘接口类型，磁盘接口类型一共有三种：IDE、VIRTIO_SCSI、VIRTIO； |
|  active   |  Boolean  | body |          可选参数，表示是否激活磁盘，默认值为false           |
|   disk    | DiskModel | body |                    封装1.4.1中的其他参数                     |

请求：

- 附加磁盘

```http
POST /acloudageks/api/vms/{vmid}/diskattachments/
Body: 
{
	"bootable": false,
	"interface": "IDE",
	"active": true,
	"disk": {
		"id": "220b052e-b0fe-4d6e-9412-a232d42342a9",
         "readOnly": false,
         "snapshot": {
         	"id": "d764f497-85a7-4585-b6f7-6fa3e4f8b085"
         }
	}
}
```

请求结果：

未添加`snapshot`参数时请求成功，添加后请求失败。

失败原因：

```json
{
    "detail": "DiskModel [snapshot.id] required for attachDiskToVm",
    "reason": "Incomplete parameters"
}
```

- 新建磁盘并附加到虚拟机

请求：

```http
POST /acloudageks/api/vms/{vmid}/diskattachments/
Body: 
{
	"bootable": false,
	"interface": "IDE",
	"active": false,
	"disk": {
         "readOnly": true,
         "format": "RAW",
         "provisionedSize": 1073741824
	}
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "DiskModel [provisionedSize|size, format] required for invoke0",
    "reason": "Incomplete parameters"
}
```

### 1.5.2.列出虚拟机所有附加磁盘

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/diskattachments
```

请求结果：成功

### 1.5.3.获得虚拟机指定的附加磁盘信息

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/diskattachments/{diskId}
```

请求结果：成功

### 1.5.4.删除或分离虚拟机附加磁盘

[1.4.7的删除接口]()只能执行磁盘分离操作，但该接口可以通过`detach_only`参数来选择分离磁盘还是直接删除磁盘。

请求参数

|   参数名    |  类型   | 位置 |                             说明                             |
| :---------: | :-----: | :--: | :----------------------------------------------------------: |
| detach_only | Boolean | url  | 可选，表示从虚拟机分离磁盘还是直接从系统删除磁盘，默认值为true |

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/disksattachments/{diskId}?detach_only=false
```

请求结果：成功

### 1.5.5.更新虚拟机附加磁盘信息

请求参数：

|  参数名   |   类型    | 位置 |                             说明                             |
| :-------: | :-------: | :--: | :----------------------------------------------------------: |
| bootable  |  Boolean  | body | 可选参数，表示是否作为将磁盘作为可引导磁盘，默认值为false。需要注意的是，每个虚拟机只能有一个可引导磁盘 |
| interface |  String   | body | 必要参数，表示磁盘接口类型，磁盘接口类型一共有三种：IDE、VIRTIO_SCSI、VIRTIO； |
|  active   |  Boolean  | body |          可选参数，表示是否激活磁盘，默认值为false           |
|   disk    | DiskModel | body |                    封装1.4.1中的其他参数                     |

请求：

```http
PUT /acloudageks/api/vms/{vmid}/disksattachments/{diskId}
Body: 
{
	"interface": "IDE",
	"disk": {
		"name": "update2"
	}
}
```

请求结果：成功

## 1.6.虚拟机网络接口

### 1.6.1.添加虚拟机网络接口

请求参数：

|   参数名    |       类型       | 位置 |                             说明                             |
| :---------: | :--------------: | :--: | :----------------------------------------------------------: |
|    name     |      String      | body |               必要参数，表示虚拟机网络接口名。               |
| vnicProfile | VnicProfileModel | body | 可选参数，表虚拟机网络接口的配置集，该参数只需要指定VnicProfileModel的id字段，默认值为null |
|  interface  |      String      | body | 可选参数，表示网卡的接口类型，可选的值一共有6种：E1000、VIRTIO、RTL8139、RTL8139_VIRTIO、SPAPR_VLAN、PCI_PASSTHROUGH。默认值为RTL8139_VIRTIO，但是该值在后台标记为废弃状态，因此建议手动设置该值；另外该与操作系统类型有关，并不是所有的系统都能够设置6种可选值 |
|     mac     |     MacModel     | body | 可选参数，代表虚拟机的mac地址，需要指定MacModel的address字段。若未设置此值，后台会从mac地址池分配合适的mac地址 |
|   linked    |     Boolean      | body |   可选参数，表示虚拟机网络接口是否连接网线，默认值为false    |
|   plugged   |     Boolean      | body | 可选参数，表示虚拟机的网卡是已插入还是拔出状态，默认值为false |

请求：

```http
POST /acloudageks/api/vms/{vmid}/nics/
Body: 
{
	"name": "test_nic",
	"vnicProfile": {
        "id": "86684825-e8f8-43bb-97b0-6972e99eaf47"
	},
	"interface": "VIRTIO",
	"linked": true,
	"plugged": true
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "NicModel [name] required for add",
    "reason": "Incomplete parameters"
}
```

### 1.6.2.列出虚拟机所有的网络接口

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大虚拟机网络接口数量 |

请求：

```http
GET /acloudageks/api/vms/{vmid}/nics?max=1
```

请求结果：成功

### 1.6.3.将虚拟机网络接口网卡设置为插入状态

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/nics/{nicId}/activate
Body: 
{
	
}
```

请求结果：成功

### 1.6.4.将虚拟机网络接口网卡设置为拔出状态

请求参数

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/nics/{nicId}/deactivate
Body: 
{
	
}
```

请求结果：成功

### 1.6.5.获得虚拟机指定的网络接口信息

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/nics/{nicId}
```

请求结果：成功

### 1.6.6.删除指定的虚拟机网络接口

请求参数：

无

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/nics/{nicId}
```

请求结果：成功

### 1.6.7.更新虚拟机网络接口信息

请求参数：

|   参数名    |       类型       | 位置 |                             说明                             |
| :---------: | :--------------: | :--: | :----------------------------------------------------------: |
|    async    |     Boolean      | body |            可选，表示是否通过异步的方式调用该请求            |
|    name     |      String      | body |                    修改虚拟机网络接口名。                    |
| vnicProfile | VnicProfileModel | body | 修改虚拟机网络接口的配置集，该参数只需要指定VnicProfileModel的id字段 |
|  interface  |      String      | body | 修改网卡的接口类型，可选的值一共有6种：E1000、VIRTIO、RTL8139、RTL8139_VIRTIO、SPAPR_VLAN、PCI_PASSTHROUGH。默认值为RTL8139_VIRTIO，但是该值在后台标记为废弃状态，因此建议手动设置该值；另外该与操作系统类型有关，并不是所有的系统都能够设置6种可选值 |
|     mac     |     MacModel     | body | 修改虚拟机网络接口的mac地址，需要指定MacModel的address字段。 |
|   linked    |     Boolean      | body |                修改虚拟机网络接口是否连接网线                |
|   plugged   |     Boolean      | body |             修改虚拟机的网卡是已插入还是拔出状态             |

请求：

```http
PUT /acloudageks/api/vms/{vmid}/nics/{nicId}
Body: 
{
	"name": "updated_nic",
	"vnicProfile": {
        "id": "86684825-e8f8-43bb-97b0-6972e99eaf47"
	},
	"interface": "RTL8139",
	"linked": false,
	"plugged": false
}
```

请求结果：成功

----

### 注意

虚拟机网卡虽然支持热插拔功能，但热插拔功能仅支持具有热插拔操作的虚拟机操作系统。 示例操作系统包括：

- Red Hat Enterprise Linux 6
- Red Hat Enterprise Linux 5
- Windows Server 2008 and
- Windows Server 2003

## 1.7.虚拟机快照

### 1.7.1.创建虚拟机快照

请求参数

|       参数名       |      类型       | 位置 |                             说明                             |
| :----------------: | :-------------: | :--: | :----------------------------------------------------------: |
|    description     |     String      | body |                     必要参数，快照描述。                     |
| persistMemorystate |     Boolean     | body | 可选参数，是否保存虚拟机内存状态作为快照的一部分，默认值为false |
|  diskAttachments   | DiskAttachments | body | 可选参数，代表需要创建快照的磁盘列表，DiskAttachments对象为DiskAttachmentModel集合的映射只持有DiskAttachment集合的引用diskAttachments字段，而DiskAttachment对象又持有DiskModel对象的引用disk字段，该参数应该是用来指定需要打快照磁盘，只要设置DiskModel对象的id字段即可。 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/snapshots
Body: 
{
	"description": "My snapshot"
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "SnapshotModel [description] required for doAdd",
    "reason": "Incomplete parameters"
}
```

### 1.7.2.列出虚拟机的快照

请求参数

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大虚拟机快照数量 |

请求：

```http
GET /acloudageks/api/vms/{vmid}/snapshots?max=1
```

请求结果：未创建快照时，调用成功；当创建快照以后`SecurityLevelMapper`的map方法执行失败，需要进行以下改动：

```java
@Mapping(from = org.ovirt.engine.core.common.businessentities.SecurityLevel.class, to = SecurityLevel.class)
public static SecurityLevel map(org.ovirt.engine.core.common.businessentities.SecurityLevel level) {
    if (level == null) {
        return com.phy.vms.api.model.SecurityLevel.NONE;
    }

    switch (level) {
        case LEVEL3:
            return com.phy.vms.api.model.SecurityLevel.LEVEL3;
        case LEVEL2:
            return com.phy.vms.api.model.SecurityLevel.LEVEL2;
        case LEVEL1:
            return com.phy.vms.api.model.SecurityLevel.LEVEL1;
        case LEVEL0:
            return com.phy.vms.api.model.SecurityLevel.LEVEL0;
        default:
            return com.phy.vms.api.model.SecurityLevel.NONE;
    }
}
```

### 1.7.3.获得虚拟机指定的快照信息

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/snapshots/{snapshotId}
```

请求结果：成功

### 1.7.4.删除虚拟机指定的快照

请求参数：

无

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/snapshots/{snapshotId}
```

请求结果：成功

### 1.7.5.将虚拟机恢复到指定快照

该接口实际上执行了两步操作。首先，进行快照预览，如果未出现异常，则接着进行预览提交操作；否则返回异常信息。

请求参数：

|    参数名     |  类型   | 位置 |                             说明                             |
| :-----------: | :-----: | :--: | :----------------------------------------------------------: |
| restoreMemory | Boolean | body |     可选参数，表示是否恢复快照的内存状态，默认值为false      |
|     disks     |  Disks  | body | 可选参数，Disks对象为DiskModel集合的映射，Disks对象只持有DiskModel集合的引用disks字段，需要DiskModel的完整的信息，默认对所有磁盘进行快照预览。 |

说明：参数中DiskModel的完整的信息不是很合理，这里应该改为只需要指定id字段和imageId字段。`ovirt4.3`中的逻辑也是如此。

请求：

```http
POST /acloudageks/api/vms/{vmid}/snapshots/{snapshotId}/restore
Body: 
{
	
}
```

请求结果：成功

## 1.8数据中心

### 1.8.1.新建数据中心

请求参数

|   参数名    |     类型     | 位置 |                             说明                             |
| :---------: | :----------: | :--: | :----------------------------------------------------------: |
|    name     |    String    | body |                     必要参数，数据中心名                     |
|    local    |   Boolean    | body |       必要参数，表示数据中心使用本地存储还是共享存储。       |
| description |    String    | body |            可选参数，数据中心描述，默认为空（""）            |
|  quotaMode  |    String    | body | 可选参数，数据中心的配额类型，可选的配额类型一共有3种：ENABLED（启动）、DISABLED（禁用）、AUDIT（审计），默认的配额类型为DISABLED |
|   macPool   | MacPoolModel | body | 可选参数，数据中心的mac地址池对象，需要指定MacPoolModel的id字段 |

请求：

```http
POST /acloudageks/api/datacenters
Body: 
{
	"name": "mydc",
	"local": false,
	"description": "test dc",
	"quotaMode": "DISABLED",
	"macPool": {
        "id": "5dd7d3e9-00ef-0112-03e6-0000000002cb"
	}
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "DataCenterModel [name] required for add",
    "reason": "Incomplete parameters"
}
```

### 1.8.2.获得所有数据中心列表

请求参数

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
| case_sensitive | Boolean | url  | 可选参数，与search参数一起使用，指定在执行search参数时是否考虑大小写 |
|     filter     | Boolean | url  |    可选参数，指定返回的数据中心信息是否根据用户权限被过滤    |
|      max       | Integer | url  |             可选参数，指定返回的最大数据中心数量             |
|     search     | String  | url  |    可选参数，查询约束条件约束返回信息，例如name="myname"     |

请求：

```http
GET /acloudageks/api/datacenters
```

请求结果：成功

### 1.8.3.获得指定的数据中心信息

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/datacenters/{datacenterId}
```

请求结果：成功

### 1.8.4删除数据中心

删除数据中心时如果没有添加任何参数，那么连接到数据中心的存储域将被分离，然后从存储中删除。如果执行此操作时某些操作失败，例如，如果没有主机可用于从存储中删除存储域，则整个操作将失败。

如果force参数为true，则删除操作无论如何都会成功，例如，即使在删除一个存储域时某些操作失败。失败的操作会被忽略该，并且从数据库中删除数据中心的信息。

请求参数：

| 参数名 |  类型   | 位置 |                             说明                             |
| :----: | :-----: | :--: | :----------------------------------------------------------: |
| force  | Boolean | url  | 可选参数，表示是否强制从数据库删除数据中心信息，即使在删除过程中，某些操作失败了，默认值为false |

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/datacenters/{datacenterId}?force=false
```

请求结果：成功

### 1.8.5.更新数据中心信息

请求参数

|   参数名    |     类型     | 位置 |                             说明                             |
| :---------: | :----------: | :--: | :----------------------------------------------------------: |
|    name     |    String    | body |                          数据中心名                          |
|    local    |   Boolean    | body | 设置数据中心使用本地存储还是共享存储，但是一旦数据中心附加了存储域，就无法修改该值。 |
| description |    String    | body |                         数据中心描述                         |
|  quotaMode  |    String    | body | 配额类型，可选的配额类型一共有3种：ENABLED（启动）、DISABLED（禁用）、AUDIT（审计） |
|   macPool   | MacPoolModel | body |    数据中心的mac地址池对象，需要指定MacPoolModel的id字段     |

请求：

```http
PUT /acloudageks/api/datacenters/{datacenterId}
Body: 
{
	"name": "updateDefault"
}
```

请求结果：成功

## 1.9数据中心存储

### 1.9.1.附加存储域到数据中心

可以通过该接口附加所有类型的存储域，即数据域、iso域和导出域；但前提条件是要附加的存储域未附加到任何数据中心，并且与被附加的数据中心的存储类型以及存储格式向匹配。

请求参数

| 参数名 |  类型  | 位置 |                             说明                             |
| :----: | :----: | :--: | :----------------------------------------------------------: |
|   id   | String | body | 可选必要参数，要附加到数据中心的存储域id，必须指定id或者name字段中的一个，若两者都指定，id字段的优先级高于name字段。 |
|  name  | String | body | 可选必要参数，要附加到数据中心的存储域名字，必须指定id或者name字段中的一个，若两者都指定，id字段的优先级高于name字段。 |

请求：

```http
POST /acloudageks/api/datacenters/{datacenterId}/storagedomains
Body: 
{
	"name": "data"
}
```

请求结果：待测试

### 1.9.2.获得数据中心所有的附加存储域

请求参数

| 参数名 |  类型   | 位置 |                说明                |
| :----: | :-----: | :--: | :--------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大存储域数量 |

请求：

```http
GET /acloudageks/api/datacenters/{datacenterId}/storagedomains?max=1
```

请求结果：成功

### 1.9.3.将数据中心指定的存储域设置为维护状态

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/datacenters/{datacenterId}/storagedomains/{storagedomainId}/deactivate
Body: 
{
	
}
```

请求结果：成功

### 1.9.4.将数据中心指定的存储域设置为激活状态

只能激活状态为维护的存储域。

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/datacenters/{datacenterId}/storagedomains/{storagedomainId}/activate
Body: 
{
	
}
```

请求结果：成功

### 1.9.5.获得数据中心信息指定的存储域信息

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/datacenters/{datacenterId}/storagedomains/{storagedomainId}
```

请求结果：成功

### 1.9.6.从数据中心分离存储域

要分离数据中心的存储域，必须先将存储域的设置为维护状态。

请求参数：

无

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/datacenters/{datacenterId}/storagedomains/{storagedomainId}
```

请求结果：待测试

## 2.0数据中心逻辑网络

### 2.0.1.为数据中心创建逻辑网络

当创建逻辑网络时，是必须指定其所属的数据中心的；通过该接口调用逻辑网络创建接口，实际上是把需要的数据中心参数放在url进行传递。

请求参数：

|     参数名      |        类型        | 位置 |                             说明                             |
| :-------------: | :----------------: | :--: | :----------------------------------------------------------: |
|      name       |       String       | body |                     必要参数，逻辑网络名                     |
|   description   |       String       | body |                    可选参数，逻辑网络描述                    |
|      vlan       |     VlanModel      | body | 可选参数，表示该网络是否为vlan，只需要设置vlan标识id，id为Integer类型 |
|       ip        |      IpModel       | body | 可选参数，代表逻辑网络的ip配置，拥有address字段和gateway字段，目前还不知道该参数有什么作用，通过ui界面新建逻辑网络时，这些字段都为空。 |
|     usages      | NetworkUsageValues | body | 可选参数，该对象为NetworkUsage集合的映射，只持有NetworkUsage枚举集合的引用networkUsage，该参数用于指定新建的逻辑网络，可以作为哪些网络使用，包括以下四种：DISPLAY（显示网络）、MANAGEMENT（管理网络）、MIGRATION（迁移网络）、VM（虚拟机网络）。其中前三种是相对于集群的（这三种网络对于集群来说都是必须的，设置的目的是是否把当前的逻辑网络作为集群的某种或多种网络），最后一种是相对于虚拟机的（即该网络是否能作为虚拟机的通信网络）。由于在数据中心无法设置集群相关内容，只能设置该网络是否为虚拟机网络；设置非虚拟机网络的方法为在NetworkUsageValues列表中添加任一非VM网络即可；不设置该值时，默认该网络为虚拟机网络。 |
|       mtu       |      Integer       | body | 可选参数，表示逻辑网络的最大传输单元，默认值为0，该值有范围要求必须等于0或者大于等于68且小于等于4094 |
| profileRequired |      Boolean       | body |  可选参数，表示是否需要创建默认的网络配置集，默认值为false   |
|       qos       |      QosModel      | body |  可选参数，设置逻辑网络的质量服务，需要指定QosModel的id字段  |

请求：

```http
POST /acloudageks/api/datacenters/{datacenterId}/networks
Body: 
{
	"name": "mynetwork",
	"description": "test dc network",
	"vlan": {
        "id": 6342
	},
	"usages": {
        "networkUsage": [
            "DISPLAY"
        ]
	},
	"mtu": 1500,
	"profileRequired": false
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "NetworkModel [name] required for add",
    "reason": "Incomplete parameters"
}
```

### 2.0.2.列出数据中心所有的逻辑网络

请求参数：

无

请求：

```http
GET /acloudageks/api/datacenters/{datacenterId}/networks
```

请求结果：成功

### 2.0.3.获得数据中心指定的逻辑网络

请求参数：

无

请求：

```http
GET /acloudageks/api/datacenters/{datacenterId}/networks/{networkId}
```

请求结果：成功

### 2.0.4.删除数据中心指定的逻辑网络

请求参数：

无

请求：

```http
DELETE /acloudageks/api/datacenters/{datacenterId}/networks/{networkId}
```

请求结果：成功

### 2.0.5.更新数据中心指定的逻辑网络信息

请求参数：

|   参数名    |        类型        | 位置 |                             说明                             |
| :---------: | :----------------: | :--: | :----------------------------------------------------------: |
|    name     |       String       | body |                          逻辑网络名                          |
| description |       String       | body |                         逻辑网络描述                         |
|    vlan     |     VlanModel      | body | 设置该网络是否为vlan，只需要设置vlan标识id，id为Integer类型  |
|   usages    | NetworkUsageValues | body | 该对象为NetworkUsage集合的映射，只持有NetworkUsage枚举集合的引用networkUsage，该参数用于指定新建的逻辑网络，可以作为哪些网络使用，包括以下四种：DISPLAY（显示网络）、MANAGEMENT（管理网络）、MIGRATION（迁移网络）、VM（虚拟机网络）。其中前三种是相对于集群的（这三种网络对于集群来说都是必须的，设置的目的是是否把当前的逻辑网络作为集群的某种或多种网络），最后一种是相对于虚拟机的（即该网络是否能作为虚拟机的通信网络）。由于在数据无法设置集群相关内容，只能设置该网络是否为虚拟机网络；设置非虚拟机网络的方法为在NetworkUsageValues列表中添加任一非VM网络即可；不设置该值时，默认该网络为虚拟机网络。 |
|     mtu     |      Integer       | body | 逻辑网络的最大传输单元，默认值为0，该值有范围要求必须等于0或者大于等于68且小于等于4094 |
|     qos     |      QosModel      | body |       设置逻辑网络的质量服务，需要指定QosModel的id字段       |

请求：

```http
PUT /acloudageks/api/datacenters/{datacenterId}/networks/{networkId}
Body: 
{
	"name": "mynetwork",
	"description": "test dc network update",
	"vlan": {
        "id": null
	}，
	"usages": {
        "networkUsage": [
            "DISPLAY"
        ]
	},
	"mtu": 68
}
```

请求结果：成功

## 2.1质量服务（qos）

服务质量相关的接口已经被删除，推测可能是删quota相关的接口时，错删了qos接口，留下了quota接口。

## 2.2集群

### 2.2.1.创建集群

请求参数

|              参数名              |         类型          | 位置 |                             说明                             |
| :------------------------------: | :-------------------: | :--: | :----------------------------------------------------------: |
|               name               |        String         | body |                       必要参数，集群名                       |
|            dataCenter            |    DataCenterModel    | body | 必要参数，表示集群所属的数据中心，需要指定数据中心name或者id字段 |
|               cpu                |       CpuModel        | body | 必要参数，可以设置**architecture**字段（代表集群cpu架构）和**type**字段（代表集群的cpu类型），通常只需要设置type即可，因为cpu架构会根据设置的cpu类型进行设置，即使设置了错误cpu架构，也会根据cpu类型进行纠正。可选的cpu架构有x86_64和ppc64。ppc64下可选的cpu类型只有`IBM POWER8`；而x86_64架构下cpu类型较多，包括AMD和因特尔CPU家族，常见的有`Intel SandyBridge Family`、`AMD Opteron G2`等。 |
|        managementNetwork         |     NetworkModel      | body | 必要参数，用于设置集群的管理网络，需要指定逻辑网络的id或者name字段 |
|           description            |        String         | body |                      可选参数，集群描述                      |
|            switchType            |        String         | body | 可选参数，集群交换机类型，可选的交换机类型有2种：LEGACY、OVS，默认值为LEGACY |
|          threadsAsCores          |        Boolean        | body | 可选参数，集群优化策略，是否将主机线程作为虚拟机cpu内核，默认值为false |
|            migration             | MigrationOptionsModel | body | 可选参数，MigrationOptionsModel的几个属性映射了集群虚拟机迁移策略的3个方面。1.**policy**字段是MigrationPolicyModel的引用，代表集群的迁移策略，集群可选的迁移策略一共有3种：Legacy（id为00000000-0000-0000-0000-000000000000）、Minimal downtime（id为80554327-0569-496b-bdeb-fcbbf52b827b）和Suspend workload if needed（id为80554327-0569-496b-bdeb-fcbbf52b827c），其中Legacy只设置了最大的迁移数量为2，而另外2种选项的详细配置见**vdc_optins**表的`MigrationPolicies`选项，迁移策略需要设置MigrationPolicyModel对象的id字段，默认值为null；2.**bandwidth**字段是MigrationBandwidthModel的引用，代表集群迁移带宽限制，由MigrationBandwidthModel的**assignmentMethod**字段设置，可选的限制有3种：AUTO、HYPERVISOR_DEFAULT以及CUSTOM，默认值为AUTO。当选择的限制为CUSTOM时，需要设置MigrationBandwidthModel的**customValue**字段对带宽进行限制（单位为Mbps），该值可以设置的最小值为1。3.**autoConverge**字段和**compressed**字段，分别表示是否自动聚合迁移以及是否启用迁移压缩。只有在迁移策略为`Legacy`时，这两个字段在虚拟机迁移过程中才会生效，它们的可选值有三种：TRUE、FALSE、INHERIT（表示从全局设置继承），这两个字段的默认值为null。 |
|          errorHandling           |  ErrorHandlingModel   | body | 可选参数，代表集群虚拟机迁移的弹性策略，即当某台主机宕机，主机上的虚拟机如何处理的策略。ErrorHandlingModel的**onError**字段代表了弹性策略，可选值有3种：MIGRATE（迁移所有的虚拟机）、DO_NOT_MIGRATE（不迁移虚拟机）、MIGRATE_HIGHLY_AVAILABLE（只迁移高可用虚拟机）。如果不设置该值，那么会根据集群的cpu架构是否支持迁移自动设置为迁移所有的虚拟机或不迁移虚拟机。 |
|         schedulingPolicy         | SchedulingPolicyModel | body | 可选参数，代表集群的调度策略，可以设置调度策略的id字段或者name字段，id字段的优先级高于name字段。内置的调度策略有以下几种，格式为`name(id)`：常规模式（b4ed2332-a7ac-4d5f-9596-99a439cb2812）、节电模式（5a2b0939-7d46-4b73-a469-e9c2c7fc6a53）、虚拟机均匀分布模式（8d5d7bec-68de-4a67-b53e-0ac54686d579）、资源均匀分布模式（20d25257-b4bd-4589-92a6-c4c5c5d3fd1a），具体配置我也看不懂，具体信息定义在`InternalClusterPolicies`类中。如果通过id和name都没找到匹配的调度策略则使用默认的调度策略，默认调度策略为常规模式。 |
| customSchedulingPolicyProperties |      Properties       | body | 可选参数，代表调度策略的可覆盖属性，可以对调度策略的属性进行覆盖。Properties对象是PropertyModel对象集合的映射，该对象只持有集合引用字段properties，需要覆盖的属性可以设置PropertyModel对象的name和value字段。 |
|          trustedService          |        Boolean        | body |        可选参数，表示是否启动可信服务，默认值为false         |
|          haReservation           |        Boolean        | body |    可选参数，表示是否启动高可用性资源预留，默认值为false     |
|          fencingPolicy           |  FencingPolicyModel   | body | 可选参数，代表集群隔离策略的几个方面。1.**enabled**字段表示是否开启集群隔离策略，只有开启集群策略以后其他字段才能生效。2.**skipIfSdActive**字段是SkipIfSdActive对象的引用，SkipIfSdActive对象的**enable**字段表示是否在主机储存上有实时租约时，忽略隔离操作。3.**skipIfConnectivityBroken**字段是SkipIfConnectivityBroken对象引用，该对象拥有**enable**字段和**threshold**字段，enable字段表示是否在主机存在问题的连接达到阈值时忽略隔离操作，而threshold则是对阈值进行设置。当未设置FencingPolicyModel对象时，集群默认启动隔离策略，并且关闭2和3两个设置。 |

请求：

```http
POST /acloudageks/api/clusters
Body: 
{
	"name": "cluster",
	"dataCenter": {
        "name": "Default"
	},
	"cpu": {
        "type": "Intel Penryn Family"
	},
	"managementNetwork": {
        "id": "00000000-0000-0000-0000-000000000009"
	},
	"description": "test creat cluster",
	"switchType": "LEGACY",
	"migration": {
        "policy": {
            "id": "80554327-0569-496b-bdeb-fcbbf52b827b"
        },
        "bandwidth": {
            "assignmentMethod": "CUSTOM",
            "customValue": 1234
        }
	},
	"errorHandling": {
        "onError": "MIGRATE_HIGHLY_AVAILABLE"
	}
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "ClusterModel [name, dataCenter.name|id] required for add",
    "reason": "Incomplete parameters"
}
```

### 2.2.2.获得所有的集群列表

请求参数

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
| case_sensitive | Boolean | url  | 可选参数，与search参数一起使用，指定在执行search参数时是否考虑大小写 |
|     filter     | Boolean | url  |      可选参数，指定返回的集群信息是否根据用户权限被过滤      |
|      max       | Integer | url  |               可选参数，指定返回的最大集群数量               |
|     search     | String  | url  |    可选参数，查询约束条件约束返回信息，例如name="myname"     |

请求：

```http
GET /acloudageks/api/clusters
```

请求结果：成功

### 2.2.3.获得指定的集群信息

请求参数

| 参数名 |  类型   | 位置 |                        说明                        |
| :----: | :-----: | :--: | :------------------------------------------------: |
| filter | Boolean | url  | 可选参数，指定返回的集群信息是否根据用户权限被过滤 |

请求：

```http
GET /acloudageks/api/clusters/{clustersId}
```

请求结果：成功

### 2.2.4.删除指定的集群

删除集群前，必须保证集群中不存在虚拟机和主机

请求参数：

无

请求：

```http
DELETE /acloudageks/api/clusters/{clustersId}
```

请求结果：成功

### 2.2.5.强制重置仿真机

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
| async  | Boolean | body | 可选参数，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/clusters/{clustersId}/resetemulatedmachine
Body: 
{
	
}
```

请求结果：成功

### 2.2.6.更新集群信息

请求参数

|              参数名              |         类型          | 位置 |                             说明                             |
| :------------------------------: | :-------------------: | :--: | :----------------------------------------------------------: |
|               name               |        String         | body |                       必要参数，集群名                       |
|               cpu                |       CpuModel        | body | 可以修改**architecture**字段（代表集群cpu架构）和**type**字段（代表集群的cpu类型），通常只需要设置type即可，因为cpu架构会根据设置的cpu类型进行设置，即使设置了错误cpu架构，也会根据cpu类型进行纠正。可选的cpu架构有x86_64和ppc64。ppc64下可选的cpu类型只有`IBM POWER8`；而x86_64架构下cpu类型较多，包括AMD和因特尔CPU家族，常见的有`Intel SandyBridge Family`、`AMD Opteron G2`等。 |
|           description            |        String         | body |                           集群描述                           |
|            switchType            |        String         | body | 集群交换机类型，可选的交换机类型有2种：LEGACY、OVS，默认值为LEGACY |
|          threadsAsCores          |        Boolean        | body |        集群优化策略，是否将主机线程作为虚拟机cpu内核         |
|            migration             | MigrationOptionsModel | body | MigrationOptionsModel的几个属性映射了集群虚拟机迁移策略的3个方面。1.**policy**字段是MigrationPolicyModel的引用，代表集群的迁移策略，集群可选的迁移策略一共有3种：Legacy（id为00000000-0000-0000-0000-000000000000）、Minimal downtime（id为80554327-0569-496b-bdeb-fcbbf52b827b）和Suspend workload if needed（id为80554327-0569-496b-bdeb-fcbbf52b827c），其中Legacy只设置了最大的迁移数量为2，而另外2种选项的详细配置见**vdc_optins**表的`MigrationPolicies`选项，迁移策略需要设置MigrationPolicyModel对象的id字段。2.**bandwidth**字段是MigrationBandwidthModel的引用，代表集群迁移带宽限制，由MigrationBandwidthModel的**assignmentMethod**字段设置，可选的限制有3种：AUTO、HYPERVISOR_DEFAULT以及CUSTOM。当选择的限制为CUSTOM时，需要设置MigrationBandwidthModel的**customValue**字段对带宽进行限制（单位为Mbps），该值可以设置的最小值为1。3.**autoConverge**字段和**compressed**字段，分别表示是否自动聚合迁移以及是否启用迁移压缩。只有在迁移策略为`Legacy`时，这两个字段在虚拟机迁移过程中才会生效，它们的可选值有三种：TRUE、FALSE、INHERIT（表示从全局设置继承）。 |
|          errorHandling           |  ErrorHandlingModel   | body | 代表集群虚拟机迁移的弹性策略，即当某台主机宕机，主机上的虚拟机如何处理的策略。ErrorHandlingModel的**onError**字段代表了弹性策略，可选值有3种：MIGRATE（迁移所有的虚拟机）、DO_NOT_MIGRATE（不迁移虚拟机）、MIGRATE_HIGHLY_AVAILABLE（只迁移高可用虚拟机）。 |
|         schedulingPolicy         | SchedulingPolicyModel | body | 代表集群的调度策略，可以设置调度策略的id字段或者name字段，id字段的优先级高于name字段。内置的调度策略有以下几种，格式为`name(id)`：常规模式（b4ed2332-a7ac-4d5f-9596-99a439cb2812）、节电模式（5a2b0939-7d46-4b73-a469-e9c2c7fc6a53）、虚拟机均匀分布模式（8d5d7bec-68de-4a67-b53e-0ac54686d579）、资源均匀分布模式（20d25257-b4bd-4589-92a6-c4c5c5d3fd1a），具体配置我也看不懂，具体信息定义在`InternalClusterPolicies`类中。 |
| customSchedulingPolicyProperties |      Properties       | body | 代表调度策略的可覆盖属性，可以对调度策略的属性进行覆盖。Properties对象是PropertyModel对象集合的映射，该对象只持有集合引用字段properties，需要覆盖的属性可以设置PropertyModel对象的name和value字段。 |
|          trustedService          |        Boolean        | body |                     表示是否启动可信服务                     |
|          haReservation           |        Boolean        | body |                 表示是否启动高可用性资源预留                 |
|          fencingPolicy           |  FencingPolicyModel   | body | 代表集群隔离策略的几个方面。1.**enabled**字段表示是否开启集群隔离策略，只有开启集群策略以后其他字段才能生效。2.**skipIfSdActive**字段是SkipIfSdActive对象的引用，SkipIfSdActive对象的**enable**字段表示是否在主机储存上有实时租约时，忽略隔离操作。3.**skipIfConnectivityBroken**字段是SkipIfConnectivityBroken对象引用，该对象拥有**enable**字段和**threshold**字段，enable字段表示是否在主机存在问题的连接达到阈值时忽略隔离操作，而threshold则是对阈值进行设置。 |

请求：

```http
PUT /acloudageks/api/clusters/{clustersId}
Body: 
{
	"name": "cluster_updated",
	"cpu": {
        "type": "Intel Penryn Family"
	},
	"description": "test update cluster",
	"switchType": "LEGACY",
	"migration": {
        "policy": {
            "id": "80554327-0569-496b-bdeb-fcbbf52b827b"
        },
        "bandwidth": {
            "assignmentMethod": "CUSTOM",
            "customValue": 1234
        }
	},
	"errorHandling": {
        "onError": "MIGRATE_HIGHLY_AVAILABLE"
	}
}
```

请求结果：成功

## 2.3.集群逻辑网络

### 2.3.1.分配集群逻辑网络

在集群所在的数据中心拥有的逻辑网络分配给指定集群，并且也可以对网络进行以下设置：

- 设置逻辑网络是否是该集群必须的
- 设置逻辑网络是否作为该集群的显示网络
- 设置逻辑网络是否作为该集群的迁移网络

需要注意的是分配网络时，不能将该网络设置为集群管理网络，原因可能是，只有网络为集群的必须网络才能被设置为管理网络。

请求参数：

|  参数名  |        类型        | 位置 |                             说明                             |
| :------: | :----------------: | :--: | :----------------------------------------------------------: |
|   name   |       String       | body |     必要参数，由于后台验证的原因这里必须指定；逻辑网络名     |
| display  |      Boolean       | body | 可选参数，是否将该网络作为集群的显示网络，方便快速设置指定网络为显示网络 |
|  usages  | NetworkUsageValues | body | 可选参数，该对象为NetworkUsage集合的映射，只持有NetworkUsage枚举集合的引用networkUsage，该参数用于指定新建的逻辑网络，可以作为哪些网络使用，包括以下四种：DISPLAY（显示网络）、MANAGEMENT（管理网络）、MIGRATION（迁移网络）、VM（虚拟机网络）。其中前三种是相对于集群的（这三种网络对于集群来说都是必须的，设置的目的是是否把当前的逻辑网络作为集群的某种或多种网络），最后一种是相对于虚拟机的（即该网络是否能作为虚拟机的通信网络）。由于该接口只能分配数据中心已有的逻辑网络到指定集群，所以只能对集群相关的内容（除了管理网络以外）进行操作。 |
| required |      Boolean       | body |           可选参数，是否将该网络作为集群的必须网络           |

请求：

```http
POST /acloudageks/api/clusters/{clustersId}/networks
Body: 
{
	"name": "re0",
	"usages": {
        "networkUsage": [
            "MIGRATION",
            "DISPLAY"
        ]
	},
	"required": false
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "NetworkModel [id|name] required for add",
    "reason": "Incomplete parameters"
}
```

### 2.3.2.获得集群的网络接口列表

请求参数

| 参数名 |  类型   | 位置 |                   说明                   |
| :----: | :-----: | :--: | :--------------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大集群逻辑网络数量 |

请求：

```http
GET /acloudageks/api/clusters/{clusterId}/networks
```

请求结果：成功

### 2.3.3.更新集群网络信息

该接口可以更新逻辑网络与集群相关的内容，与2.3.1的差异在于，该接口不能分配网络给集群，但是可以设置逻辑网络是否为管理网络，而且不需要在请求体传递逻辑网络的id或者name字段，将id作为url的一部分在url传递。

需要注意的是，不能在一个非空集群（有主机和虚拟机的集群）进行管理网络的修改，而且对应的网络在主机上必须要拥有ip。

请求参数：

|  参数名  |        类型        | 位置 |                             说明                             |
| :------: | :----------------: | :--: | :----------------------------------------------------------: |
| display  |      Boolean       | body | 可选参数，是否将该网络作为集群的显示网络，方便快速设置指定网络为显示网络 |
|  usages  | NetworkUsageValues | body | 可选参数，该对象为NetworkUsage集合的映射，只持有NetworkUsage枚举集合的引用networkUsage，该参数用于指定新建的逻辑网络，可以作为哪些网络使用，包括以下四种：DISPLAY（显示网络）、MANAGEMENT（管理网络）、MIGRATION（迁移网络）、VM（虚拟机网络）。其中前三种是相对于集群的（这三种网络对于集群来说都是必须的，设置的目的是是否把当前的逻辑网络作为集群的某种或多种网络），最后一种是相对于虚拟机的（即该网络是否能作为虚拟机的通信网络）。由于该接口只能分配数据中心已有的逻辑网络到指定集群，所以只能对集群相关的内容进行操作。 |
| required |      Boolean       | body |           可选参数，是否将该网络作为集群的必须网络           |

请求：

```http
PUT /acloudageks/api/clusters/{clustersId}/networks/{networkId}
Body: 
{
	"usages": {
        "networkUsage": [
            "MIGRATION",
            "DISPLAY",
            "MANAGEMENT"
        ]
	},
	"required": true
}
```

请求结果：成功

### 2.3.4.获得指定的集群逻辑网络

请求参数：

无

请求：

```http
GET /acloudageks/api/clusters/{clustersId}/networks/{networkId}
```

请求结果：待测试

### 2.3.5.分离指定的集群网络

请求参数：

无

请求：

```http
DELETE /acloudageks/api/clusters/{clustersId}/networks/{networkId}
```

请求结果：待测试

## 2.4集群CPU配置集

### 2.4.1.添加集群CPU配置集

请求参数：

|   参数名    |   类型   | 位置 |                             说明                             |
| :---------: | :------: | :--: | :----------------------------------------------------------: |
|    name     |  String  | body |                   必要参数，CPU配置集名称                    |
| description |  String  | body |                   可选参数，CPU配置集描述                    |
|     qos     | QosModel | body | 可选参数，需要设置cpu质量服务id，默认值为null，表示qos为无限 |

请求：

```http
POST /acloudageks/api/clusters/{clustersId}/cpuprofiles
Body: 
{
	"name": "test cpu profile"
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "CpuProfileModel [name] required for validateParameters",
    "reason": "Incomplete parameters"
}
```

### 2.4.2.获得集群CPU配置集列表

请求参数

| 参数名 |  类型   | 位置 |              说明              |
| :----: | :-----: | :--: | :----------------------------: |
|  max   | Integer | url  | 可选，指定返回的最大存储域数量 |

请求：

```http
GET /acloudageks/api/clusters/{clustersId}/cpuprofiles
```

请求结果：成功

### 2.4.3.获得集群指定的CPU配置集信息

请求参数：

无

请求：

```http
GET /acloudageks/api/clusters/{clustersId}/cpuprofiles/{cpuprofileId}
```

请求结果：成功

### 2.4.4.删除集群指定CPU配置集

请求参数：

无

请求：

```http
DELETE /acloudageks/api/clusters/{clustersId}/cpuprofiles/{cpuprofileId}
```

请求结果：成功

## 2.5.主机

### 2.5.1.新建主机

请求参数

|     参数名      |         类型         | 位置 |                             说明                             |
| :-------------: | :------------------: | :--: | :----------------------------------------------------------: |
|      name       |        String        | body |                       必要参数，主机名                       |
|     address     |        String        | body |                   必要参数，主机域名或者ip                   |
|     cluster     |     ClusterModel     | body |  必要参数，指定主机所属的集群，需要指定集群的id或者name字段  |
|       ssh       |       SshModel       | body | 可选必要参数，SshModel对象代表了与主机进行ssh连接的几个选项。其中authenticationMethod字段表示进行ssh认证的方式，一共有两种选择：PASSWORD（通过用户名和密码认证），PUBLICKEY（通过ssh公钥认证），默认值为PASSWORD，新建主机时只能使用PASSWORD方式；fingerprint字段代表主机指纹，具体作用不太清楚；port字段代表主机ssh服务的端口号，默认值为22；user字段是UserModel对象的引用，代表了连接主机的用户信息，通常只需要指定其userName（用户名）字段和password（密码）字段，UserModel对象只在认证方式为PASSWORD有效。该参数与rootPassword参数必须有一个正确填写。 |
|  rootPassword   |        String        | body | 可选必要参数，因为新建主机时只能使用用户名和密码进行认证，所以提供了，一种便利的方式，默认使用root用户进行认证，因此只需要指定root用户密码即可。该参数与ssh参数必须有一个正确填写。 |
|       spm       |       SpmModel       | body | 可选参数，SpmModel代表了主机spm相关属性，包括主机的spm状态（status）和spm优先级（priority）。新建主机时，可以指定主机的spm优先级，可选范围为-1~10，默认值为0。 |
|       os        | OperatingSystemModel | body | 可选参数，OperatingSystemModel对象代表与系统引导有关的参数。它的customKernelCmdline字段，表示在主机安装时更改主机的内核引导参数，可以选择的默认引导参数有4个：intel_iommu=on、 kvm-intel.nested=1、vfio_iommu_type1.allow_unsafe_interrupts=1、pci=realloc，当添加多个引导参数时，需要在引导参数间用空格隔开。 |
| powerManagement | PowerManagementModel | body | 可选参数，PowerManagementModel代表了主机的电源管理策略中的几个选项。其中enabled字段表示是否启用电源管理，默认值为false，只有启用了电源管理其他参数才能起作用；automaticPmEnabled字段表示，是否禁用电源管理策略控制，这里需要注意的是false表示禁用，而true表示启用，默认值为false；pmProxies字段是PmProxies对象的引用，代表了电源管理代理的集合映射，PmProxies对象只持有一个PmProxyModel对象集合的引用字段pmProxies，PmProxyModel只有一个type字段用于指定代理类型，可选的电源管理代理类型有三种：CLUSTER、DC、OTHER_DC；kdumpDetection字段表示是否集成KDump，默认值为false |
|      port       |       Integer        | body | 可选参数，代表主机vdsm服务的端口号，默认值为54321，一般不会使用这个值。 |
|    protocol     |        String        | body | 可选参数，设置与主机的通信协议，可选值有：XML、STOMP，默认值为STOMP，因为目前使用的都是STOMP协议，这个值也不需要设置。 |

请求：

```http
POST /acloudageks/api/hosts
Body: 
{
	"name": "myhost",
	"address": "172.16.2.242",
	"rootPassword": "adcd1234",
	"spm": {
        "priority": 5
	},
	"powerManagement": {
        "enabled": true,
        "automaticPmEnabled": false,
        "pmProxies": {
            "pmProxies": [
                {"type": "CLUSTER"},
                {"type": "DC"}
            ]
        }
	}
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "HostModel [name, address] required for add",
    "reason": "Incomplete parameters"
}
```

### 2.5.2.获得主机列表

 请求参数

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
| case_sensitive | Boolean | url  | 可选，与search参数一起使用，指定在执行search参数时是否考虑大小写 |
|     filter     | Boolean | url  |        可选，指定返回的主机信息是否根据用户权限被过滤        |
|      max       | Integer | url  |                 可选，指定返回的最大主机数量                 |
|     search     | String  | url  |      可选，查询约束条件约束返回信息，例如name="myname"       |

请求：

```http
GET /acloudageks/api/hosts
```

请求结果：成功

### 2.5.3.激活主机

请求参数

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/activate
Body: 
{
	
}
```

请求结果：待测试

### 2.5.4.批准主机

批准预安装的Hypervisor主机以在虚拟化环境中使用。 此操作还接受可选的cluster元素，以为此主机定义目标集群。

请求参数

|    参数名    |     类型     | 位置 |                             说明                             |
| :----------: | :----------: | :--: | :----------------------------------------------------------: |
|    async     |   Boolean    | body |            可选，表示是否通过异步的方式调用该请求            |
|   cluster    | ClusterModel | body |            可选参数，需要指定集群的name或者id字段            |
|     ssh      |   SshModel   | body | 可选参数，SshModel对象代表了与主机进行ssh连接的几个选项。其中authenticationMethod字段表示进行ssh认证的方式，一共有两种选择：PASSWORD（通过用户名和密码认证），PUBLICKEY（通过ssh公钥认证）；user字段是UserModel对象的引用，代表了连接主机的用户信息，通常只需要指定其userName（用户名）字段和password（密码）字段，UserModel对象只在认证方式为PASSWORD有效。批准主机默认使用的认证方式是PUBLICKEY，在不想使用默认的ssh方式时，可以选择。 |
| rootPassword |    String    | body | 可选参数，使用root用户进行认证，因此只需要指定root用户密码即可。在不想使用默认的ssh方式时，可以选择。 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/approve
Body: 
{
	
}
```

请求结果：待测试

### 2.5.5.保存主机网络配置

将网络配置标记为良好，并持久化到主机上。（把网络信息保存到`/var/lib/vdsm/persistence/netconf/nets/ovirtmgmt`目录下？？？？）

用户可以提交网络配置用于持久化主机网络接口的附加或分离，或持久化绑定接口的创建和删除。

仅在引擎确定主机连接不会由于配置更改而丢失后，才提交网络配置。 如果主机连接丢失，则主机需要重新启动并自动恢复为以前的网络配置。

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/commitnetconfig
Body: 
{
	
}
```

请求结果：待测试

### 2.5.6.使主机进入维护状态

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |
| reason | String  | body |      可选参数，设置维护主机的原因      |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/deactivate
Body: 
{
	"reason": "test deactivate host"
}
```

请求结果：待测试

### 2.5.7.注册主机证书

当主机证书快要过期时使用。

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/enrollcertificate
Body: 
{
	
}
```

请求结果：待测试

### 2.5.8.主机电源管理

请求参数：

|  参数名   |  类型   | 位置 |                             说明                             |
| :-------: | :-----: | :--: | :----------------------------------------------------------: |
|   async   | Boolean | body |            可选，表示是否通过异步的方式调用该请求            |
| fenceType | String  | body | 必要参数，用于控制主机的电源管理设备，可选的参数有：MANUAL、RESTART、START、STOP、STATUS |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/fence
Body: 
{
	"fenceType": "RESTART"
}
```

请求结果：待测试

### 2.5.9.将主机作为Spm

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/forceselectspm
Body: 
{
	
}
```

请求结果：待测试

### 2.5.10.获得指定的主机信息

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}
```

请求结果：成功

### 2.5.11.重新安装主机

请求参数：

|    参数名    |   类型   | 位置 |                             说明                             |
| :----------: | :------: | :--: | :----------------------------------------------------------: |
|    async     | Boolean  | body |            可选，表示是否通过异步的方式调用该请求            |
|     ssh      | SshModel | body | 可选必要参数，SshModel对象代表了与主机进行ssh连接的几个选项。其中authenticationMethod字段表示进行ssh认证的方式，一共有两种选择：PASSWORD（通过用户名和密码认证），PUBLICKEY（通过ssh公钥认证），默认值为PASSWORD；fingerprint字段代表主机指纹，具体作用不太清楚；port字段代表主机ssh服务的端口号，默认值为22；user字段是UserModel对象的引用，代表了连接主机的用户信息，通常只需要指定其userName（用户名）字段和password（密码）字段，UserModel对象只在认证方式为PASSWORD有效。该参数与rootPassword参数必须有一个正确填写。 |
| rootPassword |  String  | body | 可选必要参数，提供了一种便利的方式，默认使用root用户进行认证，因此只需要指定root用户密码即可。该参数与ssh参数必须有一个正确填写。 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/install
Body: 
{
	"ssh": {
        "authenticationMethod": "PUBLICKEY"
	}
}
```

请求结果：待测试

### 2.5.12.更新主机信息

请求参数：

|     参数名      |         类型         | 位置 |                             说明                             |
| :-------------: | :------------------: | :--: | :----------------------------------------------------------: |
|      name       |        String        | body |                            主机名                            |
|     cluster     |     ClusterModel     | body | 指定主机所属的集群，需要指定集群的id或者name字段，只有主机在维护状态下才能修改主机所属集群 |
|       ssh       |       SshModel       | body | 可选必要参数，SshModel对象代表了与主机进行ssh连接的几个选项。其中authenticationMethod字段表示进行ssh认证的方式，一共有两种选择：PASSWORD（通过用户名和密码认证），PUBLICKEY（通过ssh公钥认证），默认值为PASSWORD；user字段是UserModel对象的引用，代表了连接主机的用户信息，通常只需要指定其userName（用户名）字段和password（密码）字段，UserModel对象只在认证方式为PASSWORD有效。该参数与rootPassword参数必须有一个正确填写。 |
|  rootPassword   |        String        | body | 可选必要参数，提供了一种便利的方式，默认使用root用户进行认证，因此只需要指定root用户密码即可。该参数与ssh参数必须有一个正确填写。 |
|       spm       |       SpmModel       | body | SpmModel代表了主机spm相关属性，包括主机的spm状态（status）和spm优先级（priority）。新建主机时，可以指定主机的spm优先级，可选范围为-1~10，默认值为0。 |
|       os        | OperatingSystemModel | body | OperatingSystemModel对代表与系统引导有关的参数。它的customKernelCmdline字段，表示在主机安装时更改主机的内核引导参数，可以选择的默认引导参数有4个：intel_iommu=on、 kvm-intel.nested=1、vfio_iommu_type1.allow_unsafe_interrupts=1、pci=realloc，当添加多个引导参数时，需要在引导参数间用空格隔开。 |
| powerManagement | PowerManagementModel | body | PowerManagementModel代表了主机的电源管理策略中的几个选项。其中enabled字段表示是否启用电源管理，默认值为false，只有启用了电源管理其他参数才能起作用；automaticPmEnabled字段表示，是否禁用电源管理策略控制，这里需要注意的是false表示禁用，而true表示启用，默认值为false；pmProxies字段是PmProxies对象的引用，代表了电源管理代理的集合映射，PmProxies对象只持有一个PmProxyModel对象集合的引用字段pmProxies，PmProxyModel只有一个type字段用于指定代理类型，可选的电源管理代理类型有三种：CLUSTER、DC、OTHER_DC；kdumpDetection字段表示是否集成KDump，默认值为false |

请求：

```http
PUT /acloudageks/api/hosts/{hostId}
Body: 
{
	"spm": {
        "priority": 7
	},
	"ssh": {
        "authenticationMethod": "PUBLICKEY"
	}，
	"powerManagement": {
        "enabled": false
	}
}
```

请求结果：待测试

### 2.5.13.刷新主机能力

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/refresh
Body: 
{
	
}
```

请求结果：待测试

### 2.5.14删除主机

请求参数：

无

请求：

```http
DELETE /acloudageks/api/hosts/{hostId}
```

请求结果：待测试

### 2.5.15.获得主机网络接口列表

该接口会返回主机的上的所有网络接口信息，包括虚拟网络接口（vlan）也会被视为一个网络接口。

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/nics
```

请求结果：成功

### 2.5.15.获得指定的主机网络接口

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/nics/{nicId}
```

请求结果：成功

### 2.5.15.获得指定的主机网络接口的网络附件列表

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/nics/{nicId}/networkattachments
```

请求结果：成功

### 2.5.15.获得主机网络附件列表

主机网络附件即分配给主机网络接口的engine逻辑网络的配置信息。

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/networkattachments
```

请求结果：成功

### 2.5.16.添加主机网络附件

相当于把engine逻辑网络分配给主机网络接口，这里的网络接口不能是虚拟机网络接口（vlan），而且网络配置不能存在冲突（多个网卡在同一个网段），不然会导致网络冲突使主机网络失效。

请求参数：

|         参数名         |         类型         | 位置 |                             说明                             |
| :--------------------: | :------------------: | ---- | :----------------------------------------------------------: |
|         async          |       Boolean        | body |          可选参数，表示是否通过异步的方式调用该请求          |
|        network         |     NetworkModel     | body | 必要参数，代表了NetworkAttachment所属的主机网络接口，hostNic必须指定HostNicModel的name字段或者id字段。 |
|        hostNic         |     HostNicModel     | body | 必要参数，代表NetworkAttachment所属的逻辑网络，network必须指定HostNicModel的name字段或者id字段。 |
|       properties       |      Properties      | body | 可选参数，Properties对象只持有集合引用，是PropertyModel对象集合的映射，通过设置PropertyModel对象的name和value字段来设置自定义属性，目前可选的name只有`bridge_opts`。properties字段字段可以为空 |
|  ipAddressAssignments  | IpAddressAssignments | body | 可选参数，IpAddressAssignments对象只持有一个IpAddressAssignmentModel列表的引用ipAddressAssignments。IpAddressAssignmentModel有两个字段，其中assignmentMethod字段表示网络启动协议，可选的值有：DHCP、STATIC、AUTOCONF、NONE。ip字段是IpModel的引用表示网络配置信息，IP有4个字段，address字段表示网址；netmask表示子网掩码；gateway表示网关；version表示网络协议版本，网络协议版本必须指定，可选值有：V4、V6，分别代表IPv4和IPv6协议。因此IpAddressAssignments的列表中最多只能有代表IPv4和IPv6的2个IpAddressAssignmentModel对象，当更新时即使不只更新V6或V4中的一个，也要同时提交未更新的内容，否则视为置空。 |
|          qos           |       QosModel       | body | 可选参数，表示对主机网络qos的覆盖。首先QosModel必须设置type字段，且必须为HOSTNETWORK；QosModel可以设置的字段有outboundAverageLinkshare（加权重的共享）、outboundAverageRealtime（实现的速率）以及outboundAverageUpperlimit（速率限制），qos字段可以为空。设置qos时，所有的网络接口都必须设置，不能只设置部分网络接口的qos。 |
| override_configuration |        String        | url  |             可选参数，代表是否保存主机网络配置。             |

请求：

```http
POST /acloudageks/api/hosts/{hostId}/networkattachments
Body: 
{
    "hostNic": {
        "name": "ens33"
    },
    "network": {
        "name": "re0"
    }
}
```

请求结果：成功

### 2.5.16.获得指定的主机网络附件

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/networkattachments/{networkattachmentId}
```

请求结果：成功

### 2.5.16.更新主机网络附件

请求参数：

|         参数名         |         类型         | 位置 |                             说明                             |
| :--------------------: | :------------------: | ---- | :----------------------------------------------------------: |
|         async          |       Boolean        | body |          可选参数，表示是否通过异步的方式调用该请求          |
|        network         |     NetworkModel     | body | 代表了NetworkAttachment所属的主机网络接口，hostNic必须指定HostNicModel的name字段或者id字段。 |
|        hostNic         |     HostNicModel     | body | 代表NetworkAttachment所属的逻辑网络，network必须指定HostNicModel的name字段或者id字段。 |
|       properties       |      Properties      | body | Properties对象只持有集合引用，是PropertyModel对象集合的映射，通过设置PropertyModel对象的name和value字段来设置自定义属性，目前可选的name只有`bridge_opts`。properties字段字段可以为空 |
|  ipAddressAssignments  | IpAddressAssignments | body | IpAddressAssignments对象只持有一个IpAddressAssignmentModel列表的引用ipAddressAssignments。IpAddressAssignmentModel有两个字段，其中assignmentMethod字段表示网络启动协议，可选的值有：DHCP、STATIC、AUTOCONF、NONE。ip字段是IpModel的引用表示网络配置信息，IP有4个字段，address字段表示网址；netmask表示子网掩码；gateway表示网关；version表示网络协议版本，网络协议版本必须指定，可选值有：V4、V6，分别代表IPv4和IPv6协议。因此IpAddressAssignments的列表中最多只能有代表IPv4和IPv6的2个IpAddressAssignmentModel对象，当更新时即使不只更新V6或V4中的一个，也要同时提交未更新的内容，否则视为置空。 |
|          qos           |       QosModel       | body | QosModel对象的引用，当该字段不为空时表示对主机网络qos的覆盖。首先QosModel必须设置type字段，且必须为HOSTNETWORK；QosModel可以设置的字段有outboundAverageLinkshare（加权重的共享）、outboundAverageRealtime（实现的速率）以及outboundAverageUpperlimit（速率限制），qos字段可以为空。设置qos时，所有的网络接口都必须设置，不能只设置部分网络接口的qos。 |
| override_configuration |        String        | url  |             代表是否覆盖主机网络配置文件的内容。             |

请求：

```http
PUT /acloudageks/api/hosts/{hostId}/networkattachments/{networkattachmentId}
Body: 
{
    "ipAddressAssignments": {
        "ipAddressAssignments": [
            {
                "assignmentMethod": "DHCP",
                "ip": {
                    "version": "V6"
                }
            }
        ]
    }
}
```

请求结果：成功

### 2.5.16.删除主机网络附件

删除主机网络附件相当于将engine逻辑网络从主机网络接口上分离。

请求参数：

无

请求：

```http
DELETE /acloudageks/api/hosts/{hostId}/networkattachments/{networkattachmentId}
```

请求结果：成功

### 2.5.15.设立主机网络

设立主机网络这个部分比较复杂，涉及的对象很多，这里根据我个人的理解对几个对象进行说明。

- LogicalNetworkModel

该对象代表了Engine逻辑网络和外部逻辑网络（没用过），对应的页面信息如下：

![1576638178644](C:\Users\phy\AppData\Roaming\Typora\typora-user-images\1576638178644.png)

在页面上可以通过拖动逻辑网络将Engine逻辑网络分配给主机网络接口，但是一个主机网络接口上只能有一个非Vlan网络。

- NetworkAttachments

该对象代表分配给主机网络接口的**Engine逻辑网络**的属性信息（一个Engine逻辑网络在不同主机网络接口上的属性信息不同），相当于修改物理主机上的网络配置文件，包括IPv4和IPv6信息、qos信息以及自定义属性，注意只有将Engine逻辑网络分配给主机网络接口以后，Engine逻辑网络才能拥有对应的NetworkAttachments。对应页面信息如下：

![1576637053495](C:\Users\phy\AppData\Roaming\Typora\typora-user-images\1576637053495.png)

但是，该对象的信息并不一定与主机网络接口的**Engine逻辑网络**的属性信息一致，可以通过同步网络的方式，将该对象设置的属性应用到分配给主机网络接口的**Engine逻辑网络**上。

- NetworkLabelModel

该对象代表逻辑网络标签，逻辑网络标签分配给网络接口以后可以与分配给同一网络接口的逻辑网络进行绑定（通过编辑或修改逻辑网络），这样只对标签进行操作就可以同时对与标签绑定的所有逻辑同时进行操作。对应的页面信息如下：

![1576717128833](C:\Users\phy\AppData\Roaming\Typora\typora-user-images\1576717128833.png)

在页面上可以通过拖动标签将标签分配给主机网络接口，一个主机网络接口可以有多个标签。

- NetworkInterfaceModel

该对象代表主机网络接口，包括了Engine逻辑网络、主机网络接口的Engine逻辑网络属性信息以及网络标签。对应的页面信息如下：

![1576639764216](C:\Users\phy\AppData\Roaming\Typora\typora-user-images\1576639764216.png)

该对象在后台对应的实体对象为VdsNetworkInteface，VdsNetworkInteface对象实际上是将Engine逻辑网络、主机网络接口的Engine逻辑网络属性信息以及网络标签等信息应用到物理主机的网络接口以后的生成的信息抽象对象。

- Bond

该对象是多个主机网络接口绑定的抽象，主机网络接口绑定即多个网络接口共用多个逻辑网络，实际上Bond对象名字（name）、绑定模式（bondOptions）以及绑定的接口名字集合（slaves）用于代表两者的绑定关系。绑定关系在数据库并没有对应的映射表，而是在VdsNetworkInteface以isbond字段代表是否绑定，并且通过共有的bondName进行聚合，VdsNetworkInteface的信息是在主机network信息更新后，从vdsm重新获得主机信息，之后再更新VdsNetworkInteface的数据库信息。对应的页面信息如下：

![1576649337797](C:\Users\phy\AppData\Roaming\Typora\typora-user-images\1576649337797.png)

---

请求参数：

|             参数名             |        类型        | 位置 |                             说明                             |
| :----------------------------: | :----------------: | :--: | :----------------------------------------------------------: |
|             async              |      Boolean       | body |            可选，表示是否通过异步的方式调用该请求            |
|   modifiedNetworkAttachments   | NetworkAttachments | body | 可选参数，代表了修改或者增加的NetworkAttachment对象的集合映射，该对只持有一个NetworkAttachmentModel对象集合的引用networkAttachments。NetworkAttachmentModel对象在新建或者更新时，以下几个字段可以设置：1.id字段，设置id字段表示更新，未设置表示新建。2.ipAddressAssignments字段是IpAddressAssignments对象的引用，IpAddressAssignments对象只持有一个IpAddressAssignmentModel列表的引用ipAddressAssignments。IpAddressAssignmentModel有两个字段，其中assignmentMethod字段表示网络启动协议，可选的值有：DHCP、STATIC、AUTOCONF、NONE。ip字段是IpModel的引用表示网络配置信息，IP有4个字段，address字段表示网址；netmask表示子网掩码；gateway表示网关；version表示网络协议版本，网络协议版本必须指定，可选值有：V4、V6，分别代表IPv4和IPv6协议。因此IpAddressAssignments的列表中最多只能有代表IPv4和IPv6的2个IpAddressAssignmentModel对象，IpAddressAssignmentModel可以为空，IPv4和IPv6的所有默认属性都为空，也可以只添加其中一个，那么另一个的所有属性则默认为空。3.qos字段是QosModel对象的引用，当该字段不为空时表示对主机网络qos的覆盖。首先QosModel必须设置type字段，且必须为HOSTNETWORK；QosModel可以设置的字段有outboundAverageLinkshare（加权重的共享）、outboundAverageRealtime（实现的速率）以及outboundAverageUpperlimit（速率限制），qos字段可以为空。4.properties字段是Properties对象的引用代表了自定义属性，Properties对象只持有集合引用，是PropertyModel对象集合的映射，通过设置PropertyModel对象的name和value字段来设置自定义属性，目前可选的name只有`bridge_opts`。properties字段字段可以为空5.hostNic字段是HostNicModel引用，代表了NetworkAttachment所属的主机网络接口，hostNic必须指定HostNicModel的name字段或者id字段。6.network字段代表了NetworkAttachment所属的逻辑网络，network必须指定HostNicModel的name字段或者id字段。当5.6结合在一起时，就可以表示把逻辑网络分配给了指定的主机网络接口。 |
| synchronizedNetworkAttachments | NetworkAttachments | body | 可选参数，该对象的networkAttachments字段表示需要立即同步（立即应用）NetworkAttachment的网络配置信息到物理主机的网络接口的NetworkAttachment集合，该集合中的NetworkAttachment对象只需要设置id字段即可，该集合NetworkAttachment既可以是modifiedNetworkAttachments中修改了的对象，也可以是后台数据库已经存在的对象，但是不能是新建的NetworkAttachment对象，因为新建的对象不能设置id字段。 |
|   removedNetworkAttachments    | NetworkAttachments | body | 可选参数，NetworkAttachments的networkAttachments字段表示需要删除的NetworkAttachment对象集合，只需要指定NetworkAttachment对象的id字段即可。当NetworkAttachment被直接删除，NetworkAttachment附加到的逻辑网络也会从主机网络接口分离。 |
|         modifiedLabels         |   NetworkLabels    | body | 可选参数，代表了新增的NetworkLabel对象的集合映射，该对象只持有一个NetworkLabelModel对象集合的引用networkLabels。在新增网络接口标签时，NetworkLabelModel需要指定两个字段1.hostNic字段，hostNic字段是网络接口标签所附加到的主机网络接口HostNicModel对象的引用，至少需要指定HostNicModel对象的id字段或name字段。2.id字段，代表标签名，即为标签取一个名字。 |
|         removedLabels          |   NetworkLabels    | body | 可选参数，代表需要从主机网络接口解绑的网络接口标签集合，该对象只持有一个NetworkLabelModel对象集合的引用networkLabels。在指定要解绑的网络标签对象时，只需要指定NetworkLabelModel的id字段。 |
|         modifiedBonds          |      HostNics      | body | 可选参数，代表对主机网络接口绑定关系的修改（从多个绑定中去掉某个绑定、合并绑定、在绑定中添加一个网络接口等）和新增（将两个或两个以上的接口组合一个绑定）操作的HostNic对象的集合映射，HostNics对象只持有一个HostNic对象集合的引用hostNics，一个HostNic对象代表一个网络接口绑定。HostNic对象在新建或者修改时，以下几个字段可以设置：1.id字段，当设置id时表示对已经存在的网络接口绑定进行修改，未设置id时表示新建一个网络接口绑定。2.name字段，name字段不能为空且必须是`bond`+数字的形式，如果name已经存在则表示对已有绑定进行修改，否则表示新建一个绑定。3.bonding字段，该字段代表需要绑定在一起的网络接口集合BondingModel对象，BondingModel有两个字段可以影响网络接口的绑定属性。4.slaves字段，slaves是HostNics对象的引用，代表需要绑定在一起的网络接口集合映射，HostNics对象只持有一个HostNic对象集合的引用hostNics，这里的一个HostNic对象代表一个需要被绑定的网络接口，我们只需要指定HostNic对象的id字段即可。5.options字段，是Options对象的引用，代表绑定网络接口的绑定模式，Options对象只持有OptionModel集合的引用字段options，需要设置的属性可以通过设置OptionModel对象的name和value字段来赋值；目前绑定模式能够设置2个参数，其一是mode可选值的范围为0-6，另一个是miimon可填的值目前只知道后台的默认值为100。 |
|          removedBonds          |      HostNics      | body | 可选参数，代表对主机网络接口绑定关系的删除操作的HostNic对象的集合映射，HostNics对象只持有一个HostNic对象集合的引用hostNics，一个HostNic对象代表一个需要删除网络接口绑定，删除集合中只需要指定HostNic对象的id字段即可。 |
|       checkConnectivity        |      Boolean       | body | 可选参数，表示是否在设置完成之后，校验主机和Engine之间的网络连接性，默认值为false。 |
|      connectivityTimeout       |      Integer       | body | 可选参数，设置与主机连接的超时时间，默认值为`vdc_options`表中的NetworkConnectivityCheckTimeoutInSeconds参数对应的值。 |

说明：在前台操作中，一个拖动操作可能对应了多个对象修改或者创建，具体的对照关系请参考`NetworkOperation`枚举类中每个枚举对象的getTarget方法返回的`NetworkOperationCommandTarget`对象的`executeNetworkCommand`所对应的操作。

请求：

```http
POST /acloudageks/api/hosts/{hostId}/setupnetworks
{
    "modifiedNetworkAttachments": {
        "networkAttachments": [
            {
                "ipAddressAssignments": {
                    "ipAddressAssignments": [
                        {
                            "assignmentMethod": "STATIC",
                            "ip": {
                                "address": "192.168.18.127",
                                "netmask": "255.255.255.0",
                                "gateway": "192.168.18.2",
                                "version": "V4"
                            }
                        },
                        {
                            "assignmentMethod": "DHCP",
                            "ip": {
                                "version": "V6"
                            }
                        }
                    ]
                },
                "qos": {
                    "type": "HOSTNETWORK",
                    "outboundAverageLinkshare": 10
                },
                "hostNic": {
                    "name": "ens33"
                },
                "network": {
                    "name": "ovirtmgmt"
                }
            }
        ]
    },
    "synchronizedNetworkAttachments": {
        "networkAttachments": [
            {
                "id": "a19e6492-0d6c-4a0a-8f24-27b5eee85380"
            }
        ]
    },
    "removedNetworkAttachments": {
        "networkAttachments": [
            {
                "id": "a19e6492-0d6c-4a0a-8f24-27b5eee85380"
            }
        ]
    },
    "modifiedLabels": {
        "networkLabels": [
            {
                "id": "test_label",
                "hostNic": {
                    "name": "ens33"
                }
            }
        ]
    },
    "removedLabels": {
        "networkLabels": [
            {
                "id": "test_remove_label"
            }
        ]
    },
    "modifiedBonds": {
        "hostNics": [
            {
                "name": "bond0",
                "bonding": {
                    "slaves": {
                        "hostNics": [
                            {
                                "id": "a19e6492-0d6c-4a0a-8f24-27b5eee85380"
                            },
                            {
                                "id": "a19e6492-0d6c-4a0a-8f24-27b5eee85381"
                            }
                        ],
                        "options": {
                            "options": [
                                {
                                    "name": "mode",
                                    "value": "1"
                                },
                                {
                                    "name": "miimon",
                                    "value": "100"
                                }
                            ]
                        }
                    }
                }
            }
        ]
    },
    "removedBonds": {
        "hostNics": [
            {
                "id": "a19e6492-0d6c-4a0a-8f24-27b5eee85383"
            }
        ]
    },
    "checkConnectivity": true
}
```

请求结果：待测试

### 2.5.16.获得主机设备列表

请求参数：

| 参数名 |  类型   | 位置 |                 说明                 |
| :----: | :-----: | :--: | :----------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大主机设备数量 |

请求：

```http
GET /acloudageks/api/hosts/{hostId}/devices
```

请求结果：成功

### 2.5.17.获得指定的主机设备信息

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/devices/{deviceId}
```

请求结果：成功

### 2.5.18.获得主机HOOK列表

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
|  max   | Integer | url  | 可选参数，指定返回的最大的主机HOOK数量 |

请求：

```http
GET /acloudageks/api/hosts/{hostId}/hooks
```

请求结果：成功

### 2.5.19.获得指定的主机HOOK信息

请求参数：

无

请求：

```http
GET /acloudageks/api/hosts/{hostId}/hooks/{hookId}
```

请求结果：成功

## 2.6.虚拟机模板

### 2.6.1.创建虚拟机模板

请求参数：

|        参数名         |         类型          | 位置 |                             说明                             |
| :-------------------: | :-------------------: | :--: | :----------------------------------------------------------: |
|         name          |        String         | body |              必要参数，表示创建的虚拟机模板名字              |
|          vm           |        VmModel        | body | 必要参数，其中必须指定作为模板的虚拟机的id字段或者name字段，id字段的优先级高于name字段。如果你想要修改虚拟机模板的磁盘相关的参数，你需要指定VmModel的diskAttachments字段，diskAttachments是DiskAttachments对象代表虚拟机附加磁盘列表，DiskAttachments只拥有diskAttachments一个字段，是DiskAttachment集合的引用，DiskAttachment对象的disk字段所指向的DiskModel才是真正的磁盘信息。当需要修改磁盘参数时必须指定要修改DiskModel的id字段。及要修改的参数，例如DiskModel的name字段表示要修改的磁盘名；DiskModel的format字段表示要修改的磁盘格式，可选值包括COW和RAW；DiskModel的storageDomains字段是StorageDomains对象的引用，表示指定磁盘的存储域，StorageDomains对象只持有一个StorageDomainModel集合的引用storageDomains字段，指定磁盘存储域时只需要指定集合中一个StorageDomainModel的id字段即可。 |
|        cluster        |     ClusterModel      | body | 可选参数，指定虚拟机模板所属的集群，可以指定集群的id字段或者name字段，如果指定了集群的id字段，那么name字段就不会有任何作用。不指定该参数时，虚拟机模板所属的集群为作为模板的虚拟机所在的集群。 |
|      cpuProfile       |    CpuProfileModel    | body | 可选参数，当虚拟机模板所属的集群与作为模板的虚拟机所在的集群不同时需要指定，虚拟机模板的CPU配置集必须是虚拟机模板所属的集群下的CPU配置集之一，只需指定CpuProfileModel的id字段即可。默认值为作为模板的虚拟机所在集群的第一个cpu配置集。 |
|      description      |        String         | body |                   可选参数，虚拟机模板描述                   |
|        version        | TemplateVersionModel  | body | 可选参数，当该字段不为空时，代表为当前虚拟机已存在的模板创建子模板。TemplateVersionModel对象有2个字段需要指定：1.baseTemplate字段，是TemplateModel的引用，代表了要创建的模板父模板，只需要指定TemplateModel的id字段即可。2.versionName字段，用于指定子模板的版本名。 |
|     storageDomain     |  StorageDomainModel   | body | 可选参数，方便统一设置虚拟机模板的磁盘所在的模板存储域，只要设置StorageDomainModel的id字段即可。这里需要注意的是，即使在VmModel中设置了DiskAttachment对象的DiskModel存储域，只要一旦设置StorageDomainModel的id字段，VmModel中对于磁盘储存域的设置便会失效。 |
|   clone_permissions   |        Boolean        | url  | 可选参数，表示是否复制作为模板的虚拟机的权限到创建的虚拟机模板上。 |
|          usb          |       UsbModel        | body | 可选参数，代表虚拟机模板的usb策略，UsbModel具有2个字段：1.enabled字段，表示是否启用usb策略，只有该字段为true时，其他字段才能生效。2.type字段，代表usb策略类型，可选值有LEGACY和NATIVE。 |
|    consoleEnabled     |        Boolean        | body | 可选参数，代表是否启用控制台。默认值是根据为模板的虚拟机是否具有控制台设备决定的。 |
|   virtioScsiEnabled   |        Boolean        | body |       可选参数，代表是否启用virtioScsi。默认值为null。       |
|   soundcardEnabled    |        Boolean        | body | 可选参数，代表是否启用声卡。默认值是根据为模板的虚拟机是否具有声卡设备决定的。 |
|        display        |     DisplayModel      | body | 可选参数，DisplayModel代表虚拟机的显示模型，有以下几个字段：1.type字段，代表虚拟机使用的图形协议，可选的图形协议选项有VNC和SPICE，当设置了该字段虚拟的视频类型（defaultDisplayType）会被置为空，由后台决定。2.monitors字段，代表监视器数量。3.singleQxlPci字段，表示是否为单个PCI接口。4.smartcardEnabled字段，表示是否启用智能卡。5.allowOverride字段，代表是否允许控制台重连（不知道啥意思）。在界面中没有对应的选项，与虚拟机是服务器还是桌面相关，当虚拟机为服务器时该会被设为true。6.keyboardLayout字段，代表vnc键盘布局（作用未知），在界面中该选项被隐藏，可选内容见`vdc_options`表的`VncKeyboardLayoutValidValues`选项。7.fileTransferEnabled字段，表示是否启用SPICE文件转换（作用未知），在界面中该选项被隐藏。8.copyPasteEnabled字段，表示是否启用SPICE剪贴板复制和粘贴（作用未知），在界面中该选项被隐藏。9.disconnectAction字段，代表控制台连接中断以后虚拟机的行为，可选值有：NONE（无操作）、LOCK_SCREEN（锁定屏幕）、LOGOUT（用户登出）、REBOOT（重启虚拟机）、SHUTDOWN（关闭虚拟机）。 |
|        filter         |        Boolean        | url  | 可选参数，表示当未指定虚拟机id字段而指定了虚拟机name字段时，根据虚拟机名查询指定虚拟机时是否根据当前登入的用户权限过滤查询的虚拟机。 |
|        memory         |         Long          | body |         可选参数，表示虚拟机模板的内存大小，单位为b          |
|    NumOfIoThreads     |        Integer        | body |             可选参数，表示虚拟机模板的IO线程数量             |
|          cpu          |       CpuModel        | body | 可选参数，CpuModel的topology字段是CpuTopologyModel的引用，代表了cpu的拓扑结构。CpuTopologyModel有几个字段：1.cores字段代表了每个虚拟机插槽的内核数；2.sockets字段代表了虚拟机cpu插槽数量；3.threads字段代表每个内核的cpu内核的线程数量，可选值只有1和2。CpuModel的architecture字段，代表集群cpu架构。可选的cpu架构有x86_64和ppc64 |
|   highAvailability    | HighAvailabilityModel | body | 可选参数，代表虚拟机高可用相关的配置，其中enabled字段表示是否开启虚拟机高可用；priority字段表示虚拟机迁移的优先级，为Interge类型。 |
|   migrationDowntime   |        Integer        | body |  可选参数，代表虚拟机自定义的迁移下线时间，为Interge类型。   |
|       migration       | MigrationOptionsModel | body | 可选参数，MigrationOptionsModel的几个属性映射了集群虚拟机迁移策略的3个方面。1.**policy**字段是MigrationPolicyModel的引用，代表集群的迁移策略，集群可选的迁移策略一共有3种：Legacy（id为00000000-0000-0000-0000-000000000000）、Minimal downtime（id为80554327-0569-496b-bdeb-fcbbf52b827b）和Suspend workload if needed（id为80554327-0569-496b-bdeb-fcbbf52b827c），其中Legacy只设置了最大的迁移数量为2，而另外2种选项的详细配置见**vdc_optins**表的`MigrationPolicies`选项，迁移策略需要设置MigrationPolicyModel对象的id字段，默认值为null；2.**autoConverge**字段和**compressed**字段，分别表示是否自动聚合迁移以及是否启用迁移压缩。只有在迁移策略为`Legacy`时，这两个字段在虚拟机迁移过程中才会生效，它们的可选值有三种：TRUE、FALSE、INHERIT（表示从全局设置继承），这两个字段的默认值为null。 |
|    customCpuModel     |        String         | body | 可选参数，代表虚拟机的cpu类型，需要注意的是虚拟机cpu类型，必须与虚拟机所在集群的cpu类型为同一产商，且代数必须等于或低于集群cpu类型。 |
| customEmulatedMachine |        String         | body |                可选参数，代表虚拟机的仿真机。                |
|     memoryPolicy      |   MemoryPolicyModel   | body | 可选参数，MemoryPolicyModel对象代表虚拟机的内存策略，其guaranteed字段代表虚拟机保证的物理内存，单位为b。 |
|          os           | OperatingSystemModel  | body | 可选参数，OperatingSystemModel对象代表与系统引导有关的参数。以下几个字段会影响模板的创建：1.boot字段是BootModel对象的引用，代表引导设备序列。BootModel只持有devices字段，是BootDeviceValues对象的引用。BootDeviceValues对象是BootDevice的集合映射，只持有BootDevice集合的引用bootDevice字段。BootDevice代表虚拟机的引导设备序列，可选值有：CDROM（光驱）、HD（硬盘）和NETWORK（网络）。列表中BootDevice各个值的排列顺序，代表虚拟机引导设备顺序，如果出现重复的值，会先去重。2.type字段，代表操作系统类型，例如other_linux，windows_7x64等，可选值参考`/share/ovirt-engine/conf/osinfo-defaults.properties`文件中的配置。3.kernel字段，代表Linux引导选项中的内核路径，因此只有在操作系统类型是linux时才会生效，该字段填写的是linux内核在iso域的路径。4.initrd字段，代表Linux引导选项中的initrd路径，因此只有在操作系统类型是linux时才会生效，该字段填写的是initrd在iso域的路径。5.cmdline字段，代表虚拟机自定义的内核参数，格式目前不知道。 |
|    bootMenuEnabled    |        Boolean        | body |               可选参数，表示是否启用引导菜单。               |
|       timeZone        |     TimeZoneModel     | body | 可选参数，TimeZoneModel对象代表了虚拟机的时区设置。其name字段，代表虚拟机时区，可选值详见`TimeZoneType`枚举类的`initializeTimeZoneList`方法返回的map的key值。 |
|        origin         |        String         | body | 可选参数，代表虚拟机来源（不会手动设置吧？），可选值有VMWARE、XEN、OVIRT、EXTERNAL、KVM。 |
|       stateless       |        Boolean        | body |              可选参数，表示虚拟机是否为无状态。              |
|    deleteProtected    |        Boolean        | body |            可选参数，表示虚拟机是否启用删除保护。            |
|      ssoEnabled       |        Boolean        | body | 可选参数，表示虚拟机是否启用单点登入（即是否使用GuestAgent）。 |
|         type          |        String         | body | 可选参数，代表虚拟机类型，可选值有：DESKTOP（桌面）、SERVER（服务器）。 |
|    tunnelMigration    |        Boolean        | body | 可选参数，代表虚拟机是否启用通道迁移（作用未知），界面没找到与之对应的选项。 |
|      startPaused      |        Boolean        | body |           可选参数，代表虚拟机是否以暂停模式启动。           |
|   customProperties    |   CustomProperties    | body | 可选参数，CustomProperties是虚拟机自定义参数集合的映射，持有唯一的customProperties字段是CustomPropertyModel列表的引用。CustomPropertyModel代表了每一个自定义参数，其中name字段代表参数名，value字段代表参数值，regexp代表正则表达式。 |
|         quota         |      QuotaModel       | body | 可选参数，QuotaModel代表了虚拟机的配额，配额具体的作用不清楚，界面上也没找到对应的内容，这里只需要指定id字段即可。 |
|    initialization     |  InitializationModel  | body |   可选参数，虚拟机初始化相关的内容，太多了，暂时先不展开。   |

对vm的name字段以及id字段和cluster的name字段关系的一个说明：如果指定了vm的id字段，创建虚拟机模板的信息来源是根据虚拟机id查询获得唯一的虚拟机信息。如果未指定vm的id字段，而是指定了vm的name字段，这时如果未指定cluster的name字段，创建虚拟机模板的信息来源是在所有虚拟机中获得名字相同的虚拟机信息（不同数据中心的虚拟机可以有一样的名字）；这时如果指定了cluster的name字段并且未指定cluster的id字段，创建虚拟机模板的信息来源则是根据集群所在的数据中心以及虚拟机名来获得唯一的虚拟机信息。

请求：

```http
POST /acloudageks/api/templates
Body: 
{
	"name": "test_tp",
	"vm": {
        "id": "66dcbd82-7644-4566-ad1f-125f4deafb87"
	},
	"cluster": {
        "name": "cluster_updated"
	},
	"cpuProfile": {
        "id": "f517fb01-ffbb-4514-85e7-410f2c7ab662"
	},
	"description": "test add vm template",
	"display": {
        "type": "VNC"
	}
}
```

请求结果：成功

### 2.6.2.获得所有的虚拟机模板

 请求参数：

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
| case_sensitive | Boolean | url  | 可选，与search参数一起使用，指定在执行search参数时是否考虑大小写 |
|     filter     | Boolean | url  |     可选，指定返回的虚拟机模板信息是否根据用户权限被过滤     |
|      max       | Integer | url  |              可选，指定返回的最大虚拟机模板数量              |
|     search     | String  | url  |      可选，查询约束条件约束返回信息，例如name="myname"       |

请求：

```http
GET /acloudageks/api/templates
```

请求结果：成功

### 2.6.3.导出虚拟机模板到导出域

请求参数：

|    参数名     |        类型        | 位置 |                             说明                             |
| :-----------: | :----------------: | :--: | :----------------------------------------------------------: |
| storageDomain | StorageDomainModel | body | 必要参数，表示需要虚拟机模板被导出到的导出域，需要指定StorageDomainModel的id字段或者name字段。 |
|   exclusive   |      Boolean       | body | 可选参数，表示当导出域存在同名虚拟机模板时是否覆盖已经存在的虚拟机模板。默认值为false. |

请求：

```http
POST /acloudageks/api/templates/{templateId}/export
Body: 
{
	"storageDomain": {
        "id": "263612f2-d67b-4ce3-9c3e-af05fd4795bf"
	},
	"exclusive": true
}
```

请求结果：待测试

### 2.6.4.获得指定的虚拟机模板信息

 请求参数：

| 参数名 |  类型   | 位置 |                         说明                         |
| :----: | :-----: | :--: | :--------------------------------------------------: |
| filter | Boolean | url  | 可选，指定返回的虚拟机模板信息是否根据用户权限被过滤 |

请求：

```http
GET /acloudageks/api/templates/{templateId}
```

请求结果：成功

### 2.6.5.删除虚拟机模板

 请求参数：

无

请求：

```http
DELETE /acloudageks/api/templates/{templateId}
```

请求结果：成功

### 2.6.6.更新虚拟模板信息

和虚拟机创建一样，可选的参数太多了这里先不进行深入的探究

 请求参数：

| 参数名 | 类型 | 位置 |          说明           |
| :----: | :--: | :--: | :---------------------: |
|  ...   | ...  | body | 与2.6.1中的参数大致相同 |

请求：

```http
PUT /acloudageks/api/templates/{templateId}
{
	"description": "test update vm template",
	"type": "DESKTOP"
}
```

请求结果：待测试

## 2.7.虚拟机模板网络接口

### 2.7.1.添加虚拟机模板网络接口

请求参数：

|   参数名    |       类型       | 位置 |                             说明                             |
| :---------: | :--------------: | :--: | :----------------------------------------------------------: |
|    name     |      String      | body |               必要参数，表示虚拟机网络接口名。               |
| vnicProfile | VnicProfileModel | body | 可选参数，表虚拟机网络接口的配置集，该参数只需要指定VnicProfileModel的id字段，默认值为null |
|  interface  |      String      | body | 可选参数，表示网卡的接口类型，可选的值一共有6种：E1000、VIRTIO、RTL8139、RTL8139_VIRTIO、SPAPR_VLAN、PCI_PASSTHROUGH。默认值为RTL8139_VIRTIO，但是该值在后台标记为废弃状态，因此建议手动设置该值；另外该与操作系统类型有关，并不是所有的系统都能够设置6种可选值 |
|     mac     |     MacModel     | body | 可选参数，设置虚拟机网络接口的mac地址，若未设置此值，后台会从mac地址池分配合适的mac地址 |
|   linked    |     Boolean      | body |   可选参数，表示虚拟机网络接口是否连接网线，默认值为false    |
|   plugged   |     Boolean      | body | 可选参数，表示虚拟机的网卡是已插入还是拔出状态，默认值为false |

请求：

```http
POST /acloudageks/api/templates/{templateId}/nics
Body: 
{
	"name": "test_nic",
	"vnicProfile": {
        "id": "86684825-e8f8-43bb-97b0-6972e99eaf47"
	},
	"interface": "VIRTIO",
	"linked": true,
	"plugged": true
}
```

请求结果：成功

### 2.7.2.获得虚拟机模板所有的网络接口列表

请求参数

| 参数名 |  类型   | 位置 |                    说明                    |
| :----: | :-----: | :--: | :----------------------------------------: |
|  max   | Integer | url  | 可选，指定返回的最大虚拟机模板网络接口数量 |

请求：

```http
GET /acloudageks/api/templates/{templateId}/nics?max=1
```

请求结果：成功

### 2.7.3.获得虚拟机模板指定的网络接口信息

请求参数：

无

请求：

```http
GET /acloudageks/api/templates/{templateId}/nics/{nicId}
```

请求结果：成功

### 2.7.4.删除指定的虚拟机模板网络接口

请求参数：

无

请求：

```http
DELETE /acloudageks/api/templates/{templateId}/nics/{nicId}
```

请求结果：成功

### 2.7.5.更新虚拟机模板网络接口信息

请求参数：

|   参数名    |       类型       | 位置 |                             说明                             |
| :---------: | :--------------: | :--: | :----------------------------------------------------------: |
|    async    |     Boolean      | body |            可选，表示是否通过异步的方式调用该请求            |
|    name     |      String      | body |                    修改虚拟机网络接口名。                    |
| vnicProfile | VnicProfileModel | body | 修改虚拟机网络接口的配置集，该参数只需要指定VnicProfileModel的id字段 |
|  interface  |      String      | body | 修改网卡的接口类型，可选的值一共有6种：E1000、VIRTIO、RTL8139、RTL8139_VIRTIO、SPAPR_VLAN、PCI_PASSTHROUGH。默认值为RTL8139_VIRTIO，但是该值在后台标记为废弃状态，因此建议手动设置该值；另外该与操作系统类型有关，并不是所有的系统都能够设置6种可选值 |
|     mac     |     MacModel     | body |                 修改虚拟机网络接口的mac地址                  |
|   linked    |     Boolean      | body |                修改虚拟机网络接口是否连接网线                |
|   plugged   |     Boolean      | body |             修改虚拟机的网卡是已插入还是拔出状态             |

请求：

```http
PUT /acloudageks/api/templates/{templateId}/nics/{nicId}
Body: 
{
	"name": "updated_nic",
	"vnicProfile": {
        "id": "86684825-e8f8-43bb-97b0-6972e99eaf47"
	},
	"interface": "RTL8139",
	"linked": false,
	"plugged": false
}
```

请求结果：成功

## 2.8.虚拟机模板磁盘

与虚拟机模板磁盘相关的接口在整理时丢失了，所以需要在`BackendTemplateResource`接口中添加以下接口方法才能使虚拟机磁盘接口生效。在`ovirt3.0`虚拟机磁盘相关的操作在此接口下执行，但是在`ovirt4.0`以后的虚拟机磁盘相关接口在磁盘附加（DiskAttachments）接口下执行，但是磁盘附加接口缺少部分操作（例如磁盘的复制），并且接口返回的对象属性，将会在[2.9]()详细介绍。

```java
@Path("disks")
TemplateDisksResource getDisksResource();
```

### 2.8.1获得虚拟机模板磁盘列表

请求参数：

| 参数名 |  类型   | 位置 |                  说明                  |
| :----: | :-----: | :--: | :------------------------------------: |
|  max   | Integer | url  | 可选，指定返回的最大虚拟机模板磁盘数量 |

请求：

```http
GET /acloudageks/api/templates/{templateId}/disks?max=1
```

请求结果：待测试

### 2.8.2.复制虚拟机模板磁盘

请求参数：

|    参数名     |        类型        | 位置 |                             说明                             |
| :-----------: | :----------------: | :--: | :----------------------------------------------------------: |
|     async     |      Boolean       | body |            可选，表示是否通过异步的方式调用该请求            |
| storageDomain | StorageDomainModel | body | 必要参数，指定复制的磁盘所在的存储域，需要指定StorageDomainModel的id字段或者name字段。 |

请求：

```http
POST /acloudageks/api/templates/{templateId}/disks/{diskId}/copy
Body: 
{
	"storageDomain": {
        "id": "263612f2-d67b-4ce3-9c3e-af05fd4795bf"
	}
}
```

请求结果：待测试

### 2.8.3.获得虚拟机模板指定的磁盘信息

请求参数：

无

请求：

```http
GET /acloudageks/api/templates/{templateId}/disks/{diskId}
```

请求结果：待测试

### 2.8.4.删除虚拟机模板指定的磁盘

仅当其他存储域上还有其他现有磁盘副本时，才会删除该磁盘

请求参数：

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
| storage_domain | String  | url  | 必要参数，当删除虚拟机模板的磁盘时必须指定磁盘所在的存储域Id |
|     force      | Boolean | url  |     可选参数，表示是否强制删除指定的磁盘，默认值为false      |

请求：

```http
DELETE /acloudageks/api/templates/{templateId}/disks/{diskId}?storage_domain=263612f2-d67b-4ce3-9c3e-af05fd4795bf
```

请求结果：待测试

## 2.9.虚拟模板附加磁盘

与[2.8]()中的接口区别在于返回对象的属性不同，是`ovirt3.0`到`ovirt4.0`版本变迁的结果，与[2.8]()中接口相比缺少了复制磁盘的接口。

### 2.9.1获得虚拟机模板附加磁盘列表

请求参数：

无

请求：

```http
GET /acloudageks/api/templates/{templateId}/diskattachments
```

请求结果：成功

### 2.9.2.获得虚拟机模板指定的附加磁盘信息

请求参数：

无

请求：

```http
GET /acloudageks/api/templates/{templateId}/diskattachments/{diskattachmentId}
```

请求结果：成功

### 2.9.3.删除虚拟机模板指定的磁盘

仅当其他存储域上还有其他现有磁盘副本时，才会删除该磁盘

请求参数：

|     参数名     |  类型   | 位置 |                             说明                             |
| :------------: | :-----: | :--: | :----------------------------------------------------------: |
| storage_domain | String  | url  | 必要参数，当删除虚拟机模板的磁盘时必须指定磁盘所在的存储域Id |
|     force      | Boolean | url  |     可选参数，表示是否强制删除指定的磁盘，默认值为false      |

请求：

```http
DELETE /acloudageks/api/templates/{templateId}/disks/{diskId}?storage_domain=263612f2-d67b-4ce3-9c3e-af05fd4795bf
```

请求结果：成功

## 3.0用户登入登出

### 3.0.1.登入

登入的过程是一个用户认证加页面跳转的过程。

在页面选择三员的操作时，实际执行`/acloudageks/webadmin/?locale=zh_CN`请求，对应的相应服务未找到，该请求会返回302重定向状态码。将页面重定向至`/acloudageks/webadmin/sso/login`并且对`app_url`参数进行了赋值，`/acloudageks/webadmin/sso/login`对应的服务也没有找到，该请求也返回302状态码，将请求重定向至`/acloudageks/sso/interactive-to-auth`（由InteractiveToAuthServlet进行处理）并且对`redirect_uri`、`scope`、`reauthenticate`参数进行赋值，然后跳转到`/acloudageks/sso/login.html`登入页面。

所以三员选择实际上是在后台对`redirect_uri`、`scope`、`reauthenticate`和`app_url`参数进行赋值，然后点击登入按钮才是正则的用户认证过程。

用户的认证也在`InteractiveToAuthServlet`中（url为`/acloudageks/sso/interactive-to-auth`）进行，认证成功以后会将认证信息（包括access_token）储存到request的Session的`ovirt-ssoSession`域所指向的SsoSession对象中，并SsoSession对象注册到认证中心（`SsoContext`）。然后向前台发起Redirect指令（302），请求地址为`/acloudageks/sso/interactive-to-app`，该请求在`InteractiveToAppServlet`中进行处理，做的操作只是简单的将SsoSession对象的access_token字段和app_url字段做为Redirect指令（302）的参数，重定向到`/acloudagekswebadmin/sso/oauth2-callback`，该请求由`SsoPostLoginServlet`进行处理，主要是对access_token进行验证，并将会话存储到数据库，最后跳转到app_url指向的地址。所以登入请求不会返回token。

请求参数：

|     参数名     |  类型  |   位置   |                             说明                             |
| :------------: | :----: | :------: | :----------------------------------------------------------: |
|   source_mac   | String | url/body |      必要参数，表mac地址？，默认传的是00:00:00:00:00:00      |
|    username    | String | url/body |                   必要参数，代表登录用户名                   |
|    password    | String | url/body |                    必要参数，代表用户密码                    |
|    profile     | String | url/body |                      必要参数，代表域名                      |
|  redirect_uri  | String | url/body | 必要参数，必须手动指定`ip:port/acloudageks/webadmin/sso/oauth2-callback`否则无法正确进入SsoPostLoginServlet的处理逻辑将会话存储到数据库。 |
|    app_url     | String | url/body |         必要参数？代表认证工作完成以后要跳转的页面。         |
|     scope      | String | url/body |                      可选参数？意义不明                      |
| reauthenticate | String | url/body |                  可选参数？认证失败重试次数                  |
|   dynamicKey   | String | url/body |         可选参数，当开启动态口令时，需要指定此参数。         |

说明：这些参数既可以在url传递，也可以在body传递。在body传递时必须以表单（即Content-Type为application/x-www-form-urlencoded）的形式进行。这些参数都是以明文的方式传递的。

请求：

任何http请求方法都可以

```http
POST /acloudageks/sso/interactive-to-auth?source_mac=00:00:00:00:00:00&username=systemAdmin&password=abcd1234&profile=internal&redirect_uri=http://172.16.2.240:8080/acloudageks/webadmin/sso/oauth2-callback&scope=ovirt-app-admin ovirt-app-api&app_url=http://172.16.2.240:8080/acloudageks/webadmin/&reauthenticate=0
```

请求结果：成功

### 3.0.2登出

登出的逻辑是一个清除Session加页面跳转的过程。

请求参数：

无

请求：

任何http请求方法都可以

```http
GET /acloudageks/webadmin/sso/logout
```

请求结果：成功

## 3.1虚拟机主机设备

### 3.1.1添加虚拟机主机设备

添加主机设备的到虚拟机的前提是先把虚拟机固定到主机上

请求参数：

| 参数名 |  类型  | 位置 |                             说明                             |
| :----: | :----: | :--: | :----------------------------------------------------------: |
|   id   | String | body | 可选必要参数，添加主机设备到虚拟机时，需要指定主机设备的id字段或者name字段，id字段的优先级高于name字段。 |
|  name  | String | body | 可选必要参数，添加主机设备到虚拟机时，需要指定主机设备的id字段或者name字段，id字段的优先级高于name字段。 |

请求：

```http
POST /acloudageks/api/vms/{vmid}/hostdevices/
Body: 
{
	"name": "pci_0000_00_07_1"
}
```

请求结果：成功

### 3.1.2获得虚拟机主机设备列表

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/hostdevices/
```

请求结果：成功

### 3.1.3.获得虚拟机指定的主机设备

请求参数：

无

请求：

```http
GET /acloudageks/api/vms/{vmid}/hostdevices/{hostdeviceId}
```

请求结果：成功

### 3.1.4.删除虚拟机指定的主机设备

请求参数：

无

请求：

```http
DELETE /acloudageks/api/vms/{vmid}/hostdevices/{hostdeviceId}
```

请求结果：成功

## 3.2.网络

### 3.2.1.获取网络

请求：

```http
GET /acloudageks/api/networks
```

请求结果：成功

### 3.2.2.获取指定id的网络

请求：

```http
GET /acloudageks/api/networks/{network_id}
```

请求结果：成功

### 3.2.3.添加网络

请求参数

| 参数名     | 类型   | 位置 | 说明                                   |
| ---------- | ------ | ---- | -------------------------------------- |
| name       | String | body | 网络名字                               |
| dataCenter | String | body | 虚拟机所在数据中心，需要指定名字或者id |
| ...        | ...    | body | 与网络相关的所有属性                   |

请求：

```http
POST /acloudageks/api/networks
Body: 
{
    "name": "myTest",
    "description": "测试postman",
    "dataCenter": {
        "id": "5dc116ca-02db-01cb-014d-000000000343"
    }
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "NetworkModel [name, dataCenter.name|id] required for add",
    "reason": "Incomplete parameters"
}
```

### 3.2.4.删除网络

请求参数：

| 参数名 | 类型    | 位置 | 说明                                   |
| ------ | ------- | ---- | -------------------------------------- |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
DELETE /acloudageks/api/networks/{network_id}
```

请求结果：成功

### 3.2.5.更新网络

更新网络的信息

请求参数：

| 参数名    | 类型                      | 位置 | 说明                                   |
| --------- | ------------------------- | ---- | -------------------------------------- |
| async     | Boolean                   | body | 可选，表示是否通过异步的方式调用该请求 |
| `network` | [Network](#types/network) | body |                                        |

请求：

```http
PUT /acloudageks/api/networks/{network_id}
Body:
{
    "description": "测试"
}

```

请求结果：成功

### 3.2.6.获取网络的vnic配置

请求：

```http
GET /acloudageks/api/networks/{network_id}/vnicprofiles
```

请求结果：成功

### 3.2.7.增加网络的vnic配置

请求参数：

| 参数名 | 类型   | 位置 | 说明                        |
| ------ | ------ | ---- | --------------------------- |
| name   | String | body | vnicprofile配置文件名字     |
| `...   | ...    | body | 与vnicprofile相关的所有属性 |

请求：

```http
POST /acloudageks/api/networks/{network_id}/vnicprofiles
Body：
{
    "name": "test",
    "description": "增加vnicprofile"
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "VnicProfileModel [name] required for validateParameters",
    "reason": "Incomplete parameters"
}
```

### 3.2.8.更新网络的vnic配置

请求参数：

| 参数名           | 类型        | 位置 | 说明                                   |
| ---------------- | ----------- | ---- | -------------------------------------- |
| async            | Boolean     | body | 可选，表示是否通过异步的方式调用该请求 |
| networkFilter.id | ID          | body | 可选，网络过滤器                       |
| description      | String      | body | 描述                                   |
| ...              | VnicProfile | body | profile的配置                          |

请求：

```http
PUT /acloudageks/api/vnicprofiles/{vnicProfiles_id}
Body：
{
    "description": "测试修改vnicprofile描述",
    "networkFilter": {
        "id": "5dc116cb-03ac-0143-0396-0000000003c1"
    }
}
```

请求结果：成功

### 3.2.9.删除网络的vnic配置

请求参数：

| 参数名 | 类型    | 位置 | 说明                                   |
| ------ | ------- | ---- | -------------------------------------- |
| async  | Boolean | body | 可选，表示是否通过异步的方式调用该请求 |

请求：

```http
DELETE /acloudageks/api/networks/{network_id}/vnicprofiles/{vnicProfiles_id}
```

请求结果：成功

### 3.2.10.查找网络的网络标签

请求：

```http
GET /acloudageks/api/networks/{network_id}/networklabels
```

请求结果：成功

### 3.2.11.添加网络的网络标签

请求参数：

| 参数名 | 类型   | 位置 | 说明 |
| ------ | ------ | ---- | ---- |
| id     | String | body |      |

请求：

```http
POST /acloudageks/api/networks/{network_id}/networklabels
Body
{
    "id": "abb"
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "NetworkLabelModel [id] required for add",
    "reason": "Incomplete parameters"
}
```

### 3.2.12.删除网络的网络标签

请求：

```http
DELETE /acloudageks/api/networks/{network_id}/vnicprofiles/{vnicProfiles_id}
```

请求结果：成功

### 3.2.13.获取使用这个网络的虚拟机信息

请求：

```http
GET /acloudageks/api/networks/{network_id}/getvmnetwork
```

请求结果：成功

## 3.3.存储

### 3.3.1.添加存储域

创建新的StorageDomain需要名称，类型，主机和存储属性。用id或name属性标识主机属性。在oVirt 3.6及更高版本中，您可以默认在存储域上启用“删除后擦除”选项。要进行配置，请在POST请求中指定wipe_after_delete。创建域后可以编辑此选项，但是这样做不会更改已存在的磁盘的delete after delete属性后的擦除。

请求参数：

| 参数名            | 类型          | 位置 | 说明                                               |
| ----------------- | ------------- | ---- | -------------------------------------------------- |
| name              | String        | body |                                                    |
| type              | String        | body | 域类型：MASTER, DATA,ISO,IMPORTEXPORT,IMAGE,VOLUME |
| host.name/host.id | String        | body | 主机属性                                           |
| storage.type      | String        | body | 存储类型：ISCSI,FCP,NFS,LOCALFS,POSIXFS,GLANCE;    |
| storage.address   | String        | body | 域的ip                                             |
| storage.path      | String        | body | 域的地址                                           |
| ...               | StorageDomain | body | StorageDomain结构体                                |

请求：添加一个data域的nfs存储

```http
POST /acloudageks/api/storagedomains
Body
{
    "name": "mydata",
    "type": "DATA",
    "storage": {
        "type": "NFS",
        "address": "mynfs.example.com",
        "path": "/exports/mydata"
    },
    "host": {
        "name": "myhost"
    }
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "StorageDomainModel [host.id|name, type, storage] required for add",
    "reason": "Incomplete parameters"
}
```

请求：添加一个iso域的nfs存储

```http
POST /acloudageks/api/storagedomains
Body
{
    "name": "myisos",
    "type": "ISO",
    "storage": {
        "type": "NFS",
        "address": "mynfs.example.com",
        "path": "/export/myisos"
    },
    "host": {
        "name": "myhost"
    }
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "StorageDomainModel [host.id|name, type, storage] required for add",
    "reason": "Incomplete parameters"
}
```

请求：添加一个data域的iscsi存储

```http
POST /acloudageks/api/storagedomains
Body
{
    "name": "myiscsi",
    "type": "DATA",
    "storage": {
        "type": "ISCSI",
        "logicalUnits": {
            "logicalUnits": [
                {
                    "id": "3600144f09dbd050000004eedbd340001"
                },
                {
                    "id": "3600144f09dbd050000004eedbd340002"
                }
            ]
        }
    },
    "host": {
        "name": "myhost"
    }
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "StorageDomainModel [host.id|name, type, storage] required for add",
    "reason": "Incomplete parameters"
}
```



### 3.3.2.获取存储域

请求：

```http
GET /acloudageks/api/storagedomains
```

请求结果：成功

### 3.3.3.根据存储域ID获取存储域信息

请求参数：

| 参数名 | 类型    | 位置 | 说明                             |
| ------ | ------- | ---- | -------------------------------- |
| filter | Boolean | body | 指示是否应根据用户权限过滤结果。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storagedomainId}
```

请求结果：成功

### 3.3.4.存储附加到数据中心

请求参数：

| 参数名            | 类型                      | 位置 | 说明                         |
| ----------------- | ------------------------- | ---- | ---------------------------- |
| async             | [Boolean](#types/boolean) | In   | 可选，指示是否应异步执行操作 |
| host.id/host.name | [Host](#types/host)       | In   | 主机名或主机id               |

请求：

```http
POST /acloudageks/api/storagedomains/{storagedomainId}/isattached
Body
{
	"host":{
		"name":"172"
	}
}
```

请求结果：失败

失败原因：

```http
{
    "detail": "ActionModel [host.id|name] required for isAttached",
    "reason": "Incomplete parameters"
}
```

### 3.3.5.刷新lun的大小

在存储服务器上增加基础LUN的大小后，用户可以刷新LUN的大小。如果需要，此操作将强制重新扫描提供的LUN并使用新大小更新数据库。

参数：

| 参数名       | 类型                      | 位置 | 说明                         |
| ------------ | ------------------------- | ---- | ---------------------------- |
| async        | [Boolean](#types/boolean) | In   | 可选，指示是否应异步执行操作 |
| logicalUnits | LogicalUnit               | In   | 指定的luns                   |

例如，为了刷新两个LUN的大小，请发送如下请求：

```http
POST /acloudageks/api/storagedomains/{storageDomainId}/refreshluns
{
    "logicalUnits": {
        "logicalUnits": [
            {
                "id": "3600144f09dbd050000004eedbd340001"
            },
            {
                "id": "3600144f09dbd050000004eedbd340002"
            }
        ]
    }
}
```

请求结果：失败

失败原因：没有iscsi存储类型

```http
{
  "detail": "[Cannot edit Storage. Storage Domain type is illegal.]",
  "reason": "Operation Failed"
}
```

### 3.3.6.删除存储域

请求参数：

| 参数名  | 类型                      | 位置 | 说明                                                         |
| ------- | ------------------------- | ---- | ------------------------------------------------------------ |
| async   | [Boolean](#types/boolean) | body | 可选，应异步执行操作                                         |
| destroy | [Boolean](#types/boolean) | url  | 指示该操作是否应该成功，并且即使无法访问存储也从数据库中删除了存储域 |
| format  | [Boolean](#types/boolean) | url  | 可选：默认值为false。指示是否应格式化实际存储，并从基础LUN或目录中删除所有元数据：[源代码] ----删除/ ovirt-engine / api / storagedomains / 123？format = true |
| host    | [String](#types/string)   | url  | 指示应使用哪个主机删除存储域。id/name                        |

请求：

```http
DELETE  /acloudageks/api/storagedomains/{storageDomainId}?host={hostName}
```

请求结果：成功

### 3.3.7.修改域的信息

请求参数：

| 参数名 | 类型                      | 位置 | 说明                         |
| ------ | ------------------------- | ---- | ---------------------------- |
| async  | [Boolean](#types/boolean) | body | 可选，指示是否应异步执行操作 |
| name   | String                    | body | 修改名称                     |
| ...    | StorageDomain             | body | StorageDomain的属性          |

请求：

```http
PUT /acloudageks/api/storagedomains/{storageDomainId}
Body
{
    "name": "test2",
    "description":"测试修改域的描述"
}
```

请求结果：成功

### 3.3.8.获取使用此存储域的虚拟机

请求参数：

| 参数名 | 类型    | 位置 | 说明                                 |
| ------ | ------- | ---- | ------------------------------------ |
| max    | integer | body | 可选，设置要返回的虚拟机的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/vms
```

请求结果：成功

### 3.3.9.获取使用此存储域的模版

请求参数：

| 参数名 | 类型    | 位置 | 说明                               |
| ------ | ------- | ---- | ---------------------------------- |
| max    | integer | body | 可选，设置要返回的模版的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/templates
```

请求结果：成功

### 3.3.10.根据虚拟机id获取域中的虚拟机的信息

参数：

| 参数名 | 类型    | 位置 | 说明                                 |
| ------ | ------- | ---- | ------------------------------------ |
| max    | integer | body | 可选，设置要返回的虚拟机的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/vms/{vm_id}
```

请求结果：成功

### 3.3.11.导入虚拟机

参数：

| 参数名            | 类型                                   | 位置 | 说明                                                         |
| ----------------- | -------------------------------------- | ---- | ------------------------------------------------------------ |
| async             | [Boolean](#types/boolean)              | body | 可选，指示是否应异步执行操作                                 |
| clone             | Boolean                                | body | 可选，指示是否应重新生成导入的虚拟机的标识符。默认情况下，导入虚拟机时将保留标识符。这意味着同一虚拟机不能多次导入，因为标识符必须唯一。要允许多次导入同一台计算机，请将此参数设置为true，因为默认值为false。 |
| cluster           | [Cluster](#types/cluster)              | body |                                                              |
| collapseSnapshots | [Boolean](#types/boolean)              | body | 可选，指示应该折叠导入的虚拟机的快照，以使结果将是没有快照的虚拟机。此参数是可选的，如果未明确指定，则默认值为false。 |
| storageDomains    | [StorageDomain](#types/storage_domain) | body | 可选                                                         |
| vm                | [Vm](#types/vm)                        | body | 可选                                                         |

请求：

```http
POST /acloudageks/api/storagedomains/{storageDomainId}/vms/{vm_id}/import
Body
{
    "storageDomain": {
        "name": "mydata"
    },
    "cluster": {
        "name": "mycluster"
    }
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "ActionModel [cluster.id|name, storageDomain.id|name] required for doImport",
    "reason": "Incomplete parameters"
}
```

### 3.3.12.删除域中的虚拟机

参数：

| 参数名 | 类型                      | 位置 | 说明                         |
| ------ | ------------------------- | ---- | ---------------------------- |
| async  | [Boolean](#types/boolean) | In   | 可选，指示是否应异步执行操作 |

请求：

```http
DELETE /acloudageks/api/storagedomains/{storageDomainId}/vms/{vm_id}
```

请求结果：成功

### 3.3.13.根据模版id获取域中的模版的信息

参数：

| 参数名 | 类型    | 位置 | 说明                                 |
| ------ | ------- | ---- | ------------------------------------ |
| max    | integer | url  | 可选，设置要返回的虚拟机的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/vms/{vm_id}
```

请求结果：成功

### 3.3.14.导入模版

参数：

| 参数名         | 类型                                   | 位置 | 说明                                                         |
| -------------- | -------------------------------------- | ---- | ------------------------------------------------------------ |
| async          | [Boolean](#types/boolean)              | body | 可选，指示是否应异步执行操作                                 |
| clone          | Boolean                                | body | 可选，指示是否应重新生成导入的模版的标识符。默认情况下，导入虚拟机时将保留标识符。这意味着同一虚拟机不能多次导入，因为标识符必须唯一。要允许多次导入同一台计算机，请将此参数设置为true，因为默认值为false。 |
| cluster        | [Cluster](#types/cluster)              | body |                                                              |
| exclusive      | [Boolean](#types/boolean)              | body | 可选                                                         |
| storageDomains | [StorageDomain](#types/storage_domain) | body | 可选                                                         |
| template       |                                        |      |                                                              |
| vm             | [Vm](#types/vm)                        | body | 可选                                                         |

请求：

```http
POST /acloudageks/api/storagedomains/{storageDomainId}/templates/{templates_id}/import
Body
{
    "storageDomain": {
        "name": "mydata"
    },
    "cluster": {
        "name": "mycluster"
    }
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "ActionModel [cluster.id|name, storageDomain.id|name] required for doImport",
    "reason": "Incomplete parameters"
}
```

### 3.3.15.删除域中的模版

参数：

| 参数名 | 类型                      | 位置 | 说明                         |
| ------ | ------------------------- | ---- | ---------------------------- |
| async  | [Boolean](#types/boolean) | body | 可选，指示是否应异步执行操作 |

请求：

```http
DELETE /acloudageks/api/storagedomains/{storageDomainId}/templates/{vm_id}
```

请求结果：成功

### 3.3.16.获取域中的磁盘列表

| 参数名 | 类型    | 位置 | 说明                         |
| ------ | ------- | ---- | ---------------------------- |
| max    | integer | body | 可选，设置要返回的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/disks
```

请求结果：成功

### 3.3.17.根据磁盘id删除域中的某个磁盘

请求：

```http
DELETE /acloudageks/api/storagedomains/{storageDomainId}/disks/{disk_id}
```

请求结果：成功

### 3.3.18.获取磁盘快照

参数：

| 参数名 | 类型    | 位置 | 说明                         |
| ------ | ------- | ---- | ---------------------------- |
| max    | integer | body | 可选，设置要返回的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/disksnapshots
```

请求结果：成功

### 3.3.19.删除磁盘快照

请求：

```http
DELETE /acloudageks/api/storagedomains/{storageDomainId}/disksnapshots/{disksnapshots_id}
```

请求结果：成功

### 3.3.20.添加磁盘配置集

参数：

| 参数名           | 类型        | 位置 | 说明           |
| ---------------- | ----------- | ---- | -------------- |
| name             | String      | body |                |
| storageDomain.id | String      | body | 存储域的id     |
| ...              | diskprofile | body | 其他的一些属性 |

请求：

```http
POST /acloudageks/api/storagedomains/{storageDomainId}/diskprofiles
{
    "name": "333",
    "storageDomain": {
        "id": "{{storageDomainId}}"
    }
}
```

请求结果：失败

```json
{
    "detail": "DiskProfile [name] required for validateParameters",
    "reason": "Incomplete parameters"
}
```

### 3.3.21.获取磁盘配置集

参数：

| 参数名 | 类型    | 位置 | 说明                         |
| ------ | ------- | ---- | ---------------------------- |
| max    | integer | body | 可选，设置要返回的最大数量。 |

请求：

```http
GET /acloudageks/api/storagedomains/{storageDomainId}/diskprofiles
```

请求结果：成功

### 3.3.22.获取指定磁盘配置集的信息

请求：

```http
GET /acloudageks/api/storagedomains/{{storageDomainId}}/diskprofiles/diskprofile_id
```

请求结果：成功

### 3.3.23.更新磁盘配置集属性

参数：

| 参数名 | 类型        | 位置 | 说明           |
| ------ | ----------- | ---- | -------------- |
| name   | String      | body |                |
| ...    | diskprofile | body | 其他的一些属性 |

请求：

```http
PUT /acloudageks/api/diskprofiles/diskprofiles_id
Body
{
	"name":"3333",
	"description":"测试更新磁盘配置集描述"
}
```

请求结果：成功

### 3.3.24.删除磁盘配置集

请求：

```http
DELETE /acloudageks/api/storagedomains/{storageDomainId}/diskprofiles/{diskprofile_id}
```

请求结果：成功

## 3.4.磁盘

### 3.4.1.获取所有磁盘信息

参数：

| 参数名         | 类型                      | 位置 | 说明                                                     |
| -------------- | ------------------------- | ---- | -------------------------------------------------------- |
| case_sensitive | [Boolean](#types/boolean) | url  | 可选；指示是否应考虑大小写来执行使用搜索参数执行的搜索。 |
| max            | integer                   | url  | 可选；设置要返回的最大磁盘数。                           |
| search         | String                    | url  | 查询字符串，用于限制返回的磁盘。                         |

请求：

```http
GET /acloudageks/api/disks
```

请求结果：成功

### 3.4.2.添加磁盘

参数：

| 参数名          | 类型       | 位置 | 说明                                                         |
| --------------- | ---------- | ---- | ------------------------------------------------------------ |
| name            | String     | body |                                                              |
| format          | DiskFormat | body | cow，raw，row：“写时复制”格式允许快照，而性能开销很小。raw：原始格式不允许使用快照，但是可以提高性能。 |
| provisionedSize | integer    | body | 磁盘的虚拟大小，以字节为单位。                               |
| ....            | Disk       | body | 其他的disks属性                                              |

请求：

```http
POST /acloudageks/api/disks
Body
{
    "storageDomain": {
        "id": "{storageDomainId}"
    },
    "name": "test_add_disk",
    "description":"test_add_disk",
    "provisionedSize":1073741824,
    "format": "COW"
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "DiskModel [provisionedSize|size, format] required for add",
    "reason": "Incomplete parameters"
}
```

### 3.4.3.删除磁盘

参数：

| 参数名 | 类型                      | 位置 | 说明                         |
| ------ | ------------------------- | ---- | ---------------------------- |
| async  | [Boolean](#types/boolean) | body | 可选，指示是否应异步执行操作 |

请求：

```http
DELETE /acloudageks/api/disks/{disk_id}
```

请求结果：成功

### 3.4.4.复制磁盘

参数：

| 参数名        | 类型                | 位置 | 说明                             |
| ------------- | ------------------- | ---- | -------------------------------- |
| async         | Boolean             | body | 指示是否应异步执行复制。         |
| disk          | [Disk](#types/disk) | body |                                  |
| filter        | Boolean             | body | 指示是否应根据用户权限过滤结果。 |
| storageDomain | StorageDomain       | body | 其他的disks属性                  |

请求：

```http
POST /acloudageks/api/disks/{disk_id}/copy
Body
{
    "storageDomain": {
        "id": "{{storageDomainId}}"
    },
    "disk": {
        "name": "mydisk"
    }
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "ActionModel [storageDomain.id|name] required for copy",
    "reason": "Incomplete parameters"
}
```

### 3.4.5.移动磁盘

参数：

| 参数名        | 类型          | 位置 | 说明                             |
| ------------- | ------------- | ---- | -------------------------------- |
| async         | Boolean       | body | 指示是否应异步执行复制。         |
| filter        | Boolean       | body | 指示是否应根据用户权限过滤结果。 |
| storageDomain | StorageDomain | body | 其他的disks属性                  |

请求：

```http
POST /acloudageks/api/disks/{disk_id}/copy
Body
{
    "storageDomain": {
        "id": "{{storageDomainId}}"
    }
}
```

请求结果：失败

失败原因：

```json
{
    "detail": "ActionModel [storageDomain.id|name] required for move",
    "reason": "Incomplete parameters"
}
```

## 3.5.虚拟机池

### 3.5.1 .添加虚拟机池

当模版为精简分配，模版的磁盘为RAW时，创建虚拟机池会失败，

参数：

| 参数名        | 类型    | 位置 | 说明               |
| ------------- | ------- | ---- | ------------------ |
| name          | String  | body |                    |
| cluster.id    | String  | body | 集群id             |
| template.id   | String  | body | 模版id             |
| size          | integer | body | 虚拟机数量         |
| maxUserVms    | integer | body | 其他的disks属性    |
| prestartedVms | integer | body | 预启动的虚拟机数量 |
| ......        | VmPool  | body | 其余的VmPool的属性 |

请求：

```http
POST  /acloudageks/api/vmpools
Body
{
    "name": "test_add_pool",
    "size": 2,
    "maxUserVms": 1,
    "prestartedVms": 0,
    "cluster": {
        "id": "{{test_add_cluster_id}}"
    },
    "template": {
        "id": "{{test_export_domain_template_id}}"
    }
}
```

请求结果：

当模版为精简分配，模版的磁盘为RAW时，创建虚拟机池会失败，返回结果为下

请求原因：

```
{
    "detail": "[Cannot create VM-Pool. Thin provisioned template disks can not be defined as Raw.]",
    "reason": "Operation Failed"
}
```

当磁盘为QCOW2时，成功

### 3.5.2.获取虚拟机池

请求参数：

| 参数名         | 类型    | 位置 | 说明                                               |
| -------------- | ------- | ---- | -------------------------------------------------- |
| case_sensitive | Boolean | url  | 指示是否应该考虑大小写来执行使用搜索参数执行的搜索 |
| filter         | Boolean | url  | 指示是否应根据用户权限过滤结果。                   |
| max            | Integer | url  | 设置要返回的最大池数。                             |
| search         | String  | url  | 查询字符串，用于限制返回的池                       |

请求：

```http
GET /acloudageks/api/vmpools
```

请求结果：成功

### 3.5.3.获取指定的虚拟机池的信息

请求参数：

| 参数名 | 类型    | 位置 | 说明                             |
| ------ | ------- | ---- | -------------------------------- |
| filter | Boolean | url  | 指示是否应根据用户权限过滤结果。 |

请求：

```http
GET  /acloudageks/api/vmpools/{vm_pool_id}
```

请求结果：成功

### 3.5.4.更新虚拟机池的信息

请求参数：

| 参数名        | 类型    | 位置 | 说明               |
| ------------- | ------- | ---- | ------------------ |
| description   | String  | body | 描述               |
| size          | integer | body | 增加的虚拟机数量   |
| maxUserVms    | integer | body | 其他的disks属性    |
| prestartedVms | integer | body | 预启动的虚拟机数量 |
| ......        | VmPool  | body | 其余的VmPool的属性 |

请求：

```http
POST /acloudageks/api/vmpools/{vm_pool_id}
Body
｛
	"description": "update vm pool"
｝
```

请求结果：成功

### 3.5.5 .删除虚拟机池

请求参数：

| 参数名 | 类型    | 位置 | 说明                                   |
| ------ | ------- | ---- | -------------------------------------- |
| async  | Boolean | url  | 可选，表示是否通过异步的方式调用该请求 |

请求

```http
DELETE /acloudageks/api/vmpools/{vm_pool_id}
```

请求结果：成功

## 3.6.系统日志

### 3.6.1.获得日志

只定义了接口，实现为空

请求参数：

无

请求：

```http
GET /acloudageks/api/logs
```

请求结果：未实现的接口

### 3.6.2.获得指定的日志

只定义了接口，实现为空

请求参数：

无

请求：

```http
GET /acloudageks/api/logs/{logId}
```

请求结果：未实现的接口

### 3.6.3.删除指定日志

只定义了接口，实现为空

请求参数：

无

请求：

```http
DELETE /acloudageks/api/logs/{logId}
```

请求结果：未实现的接口

### 3.6.4.导出日志

请求参数：

无

请求：

```http
GET /acloudageks/api/logs/export
```

请求结果：

接口不正确，无法下载文件，返回的内容如下：

```txt
file/log_file_75117da0-6b10-4377-92dc-85138ae49a1a.csv
```

### 3.6.5.获取所有警告日志

请求参数：

|                         参数名                          |   类型   | 位置 |   说明   |
| :-----------------------------------------------------: | :------: | :--: | :------: |
| page=1&username=systemAdmin%40internal&start=0&limit=20 | 原生参数 | body | 查询条件 |

请求：

```http
POST /acloudageks/api/logs/notifications
Body
limit=1
```

请求结果：成功

建议：请求参数的格式可能不太合适，建议改为url形式，或者以model的形式进行传递。

### 3.6.6.按条件查询日志

请求参数：

|                      参数名                      |   类型   | 位置 |   说明   |
| :----------------------------------------------: | :------: | :--: | :------: |
| moduleType=&sendIP=&riskLevel=&behaviourType=... | 原生参数 | body | 查询条件 |

请求：

```http
POST /acloudageks/api/logs/getByConditions
Body
limit=1
```

请求结果：成功

建议：请求参数的格式可能不太合适，建议改为url形式，或者以model的形式进行传递。

### 3.6.6.获得所有的日志

由于清空日志的操作是一个伪删除操作，所以需要过滤状态的为deleted的日志，但是此接口没有进行过滤，需要修改该接口。

请求参数：

无

请求：

```http
GET /acloudageks/api/logs/getAll
```

请求结果：成功

### 3.6.7.清空日志

这是一个伪删除操作，只是将日志的状态设置为deleted，实际上并没有删除日志。并且删除的日志与登入的角色相关。

请求参数：

无

请求：

```http
POST /acloudageks/api/logs/clear
```

请求结果：成功

说明：返回的结果的形式与其他接口不统一

### 3.6.8.获得kvm日志

只定义了接口，实现为空

请求参数：

无

请求：

```http
GET /acloudageks/api/logs/getKvmLog
```

请求结果：未实现的接口

### 3.6.8.获得Root日志

只定义了接口，实现为空

请求参数：

无

请求：

```http
GET /acloudageks/api/logs/getRootLog
```

请求结果：未实现的接口