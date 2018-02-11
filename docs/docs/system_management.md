# 系统管理

## 节点白名单管理

CITA中节点分为共识节点和只读节点，交易由共识节点排序并打包成块，再广播
至其他节点，共识完成后即被确认为合法区块。只读节点不参与共识，只同步链
上所有的原始数据。

公有链没有节点准入机制，意味着任何节点都可以接入链并同步其全部的数据，
在满足一定的条件下都可以参加共识。而CITA对于共识节点和只读节点都进行了
准入管理。对于身份验证失败的节点，即使该节点能够在网络层与其他CITA节点
连通，这些CITA节点也会拒绝与之建立通讯会话，如此可避免信息泄漏。

目前CITA对于节点的准入管理采用白名单的方式。每个节点本地保存节点白名单
配置文件，其中记录着允许连接的p2p通信和数据同步的节点，包括其公钥、IP
地址、端口、对应的身份信息等。白名单由管理机构生成并分发，运维人员可对
其进行维护和管理，可选择连接若干其他节点同时可配置若干只读节点，使其承
担数据分析等工作。

## 共识节点管理

CITA作为一个面向企业级应用的区块链框架，需要保证监管方能够获得相关的权限对共识节点进行管理，包括增加、删除共识节点等操作。对于共识服务方面，需要对其提供实时读取共识节点列表的接口，而中心化管理的方式无法保证各个节点的共识节点列表的安全性及一致性。CITA采用合约的方式来实现共识节点的管理，通过区块链上的合约可以保证共识节点的安全性及一致性。

在CITA初始化创世块阶段，需要初始化一个管理员地址，其拥有管理员角色，将其写入到每个节点的创世块文件中，共识节点管理合约拥有的一个固定地址也写入其中。创世块内容在初始化以后不允许被修改。区块链正常启动之后，将合约写入到创世块中。链外的操作人员可以通过调用RPC接口来实现对共识节点的管理。

对于共识节点的管理，包括添加、删除及获得共识节点。

* 添加操作分为发起和确认，节点先调用发起请求，申请成为共识节点，由管理员(拥有管理员角色的账号)确认才完成了添加操作;
* 删除操作只可由管理员执行;
* 共识服务可获得共识节点列表。

只读节点安装生成后申请成为共识节点，需要进行以下操作：

* 将账号地址提交给管理员;
* 节点发起一个记录其为共识节点的合约，并由管理员完成确认; 
* 和其他节点共同修改本地节点白名单;
* 等待区块数据同步完成后即可参与下一次的共识。

### 共识节点管理合约接口

* 准备共识节点-newNode

    - 普通角色即可;
    - 成功后新节点准备成为共识节点，并将其记录在合约共识节点列表中，同时节点将处于new状态;
    - 传入参数address，为新增节点地址;
    - 返回类型为bool，可通过其判断操作成功与否。

* 确认共识节点-approveNode

    - 需要管理员角色;
    - 新节点成功准备后，可调用此方法确认节点成为共识节点，同时节点将处于consensus状态;
    - 传入参数string，为新增共识节点地址;
    - 返回类型为bool，可通过其判断操作成功与否。

* 删除共识节点-deleteNode

    - 需要管理员角色;
    - 成功后节点将从节点列表中删除，同时节点将处于close状态;
    - 传入参数为address，为节点地址;
    - 返回类型为bool，可通过其判断操作成功与否。

* 获取共识节点列表-listNode

    - 只读方法，普通角色即可;
    - 态获取共识节点列表，即状态为consensus的节点;
    - 返回结果为string。节点列表中的多个节点会拼接成一个字符串。之后可通过解析会获得节点列表。

* 获得节点状态-getStatus

    - 只读方法，普通角色即可;
    - 获取共识节点状态;
    - 传入参数为节点公钥;
    - 返回结果为uint8:

        * 0表示close状态
        * 1表示new状态
        * 2表示consensus状态

## 账号管理

CITA统一对账号进行基于角色的权限管理。系统内置了两种角色:

* 管理员角色拥有全部权限。包括管理节点的权限、发送交易的权限、创建合约的权限以及所有普通角色的权限;
* 普通角色拥有读取的权限以及创建角色的权限。包括验证节点是否是共识节点、获取共识节点列表、判断是否拥有角色、获取角色列表、判断是否拥有权限、获取权限列表等。

其中管理员角色和管理员绑定，即管理员账号有且只有管理员角色。管理员的添加只能由管理员操作，并且无法进行删除操作。角色可由用户创建，可相互授予及收回，而角色对应的权限只可由管理员进行设置。

CITA通过智能合约的方式来对账号进行管理。其中账号管理合约用来管理角色，权限管理合约用来管理角色的权限。

### 账号管理合约接口

- 创建角色-createRole 

    * 账号调用，需拥有用户角色;
    * 成功后创建一个新的角色，并拥有创建者指定权限，默认创建角色继承创建者角色权限，需调用权限管理合约接口;
    * 传入参数:

        - bytes32: 角色名称
        - uint8[]: 权限列表

    * 返回类型为bool，可通过其判断操作成功与否。

- 添加管理员-addAdmin

    * 账号调用，需拥有管理员角色;
    * 成功后授予账号管理员角色，即添加了新的管理员;
    * 传入参数: address，为账号地址;
    * 返回类型为bool，可通过其判断操作成功与否。

- 判断账号是否拥有角色-ownRole

    * 只读方法，账号调用，需拥有用户角色;
    * 判断账号是否拥有指定的角色;
    * 传入参数:

        - address: 账号地址;
        - bytes32: 角色名称。

    * 返回类型为bool，可通过其判断操作成功与否。

- 查询账号拥有的角色-listRole

    * 账号调用，需拥有用户角色;
    * 读取账号所拥有的权限;
    * 传入参数: address，为账号地址;
    * 返回类型为bytes32[MAX_ROLE]，为账号拥有的角色列表。其中MAX_ROLE为角色拥有的最大角色数。

- 授予指定账号角色-grandRole

    * 用户账号调用，需拥有用户角色;
    * 对给定账号授予已存在的角色，成功后该账号拥有此角色;
    * 传入参数:

        - address: 账号地址;
        - bytes32: 角色名称。

    * 返回类型为bool，可通过其判断操作成功与否。

- 收回指定账号角色-revokeRole

    * 用户账号调用。账号需为角色创建者或者拥有管理员角色;
    * 撤销账号拥有的角色，成功后该账号失去此角色;
    * 传入参数:

        - address: 账号地址;
        - bytes32: 角色名称。

    * 返回类型为bool，可通过其判断操作成功与否。

### 权限管理合约接口

- 设置角色权限-setRolePermission 

    * 账号调用，需拥有管理员角色;
    * 只能由管理员授予角色权限，成功后角色拥有指定权限;
    * 传入参数: 

        - bytes32: 角色名称;
        - uint8[]: 权限列表。

    * 返回类型为bool，可通过其判断操作成功与否。

- 判断角色是否拥有权限-ownPermission

    * 只读方法，账号调用，需拥有用户角色;
    * 判断角色是否拥有指定的权限;
    * 传入参数:

        - bytes32: 角色名称。
        - uint8: 权限名称;

    * 返回类型为bool，可通过其判断操作成功与否。

- 查询角色拥有的权限-listPermission

    * 账号调用，需拥有用户角色;
    * 读取角色所拥有的权限;
    * 传入参数: bytes32，为角色名称;
    * 返回类型为bytes32[MAX_PERMISSION]，为角色拥有的权限列表。其中MAX_PERMISSION为角色拥有的最大权限数。

## 配额管理

通过配额管理合约实现对区块(中的视图）以及用户配额消耗上限的管理:

* 设置区块配额上限即为每个区块设置统一的配额上限;
* 设置账号配额上限包括:

    - 默认的账号配额上限，全局设置，即若账号未指定配额上限，默认为此值;
    - 设置指定账号配额上限，可针对不同用户灵活分配对应的配额上限。

### 配额管理合约接口

- 设置区块配额上限-setBQL(BQL为BlockQuotaLimit缩写，下同)

    * 需要管理员角色;
    * 设置每个块的配额上限;
    * 传入参数uint，为设置的配额值;
    * 返回类型为bool，可通过其判断成功与否。

- 设置默认账号配额上限-setDefaultAQL(AQL为BlockQuotaLimit缩写，下同)

    * 需要管理员角色;
    * 设置默认的账号配额上限;
    * 传入参数为uint，为设置的配额值;
    * 返回类型为bool，可通过其判断成功与否。

- 设置指定账号配额上限-setAQL

    * 需要管理员角色;
    * 设置指定账号的配额上限;
    * 传入参数:

      - address: 指定的账号的地址;
      - uint: 设置的配额值。

    * 返回类型为bool，可通过其判断成功与否。

- 查询区块配额上限-getBQL

    * 普通角色即可;
    * 查询设置的区块配额上限;
    * 返回类型uint，为查询到的配额上限。

- 查询默认账号配额上限-getDefaultAQL

    * 普通角色即可;
    * 查询设置的默认账号配额上限;
    * 返回类型uint，为查询到的配额上限。

- 查询指定账号配额上限-getAQL

    * 普通角色即可;
    * 查询设置的指定账号配额上限;
    * 传入参数为address，为指定的账号地址;
    * 返回类型uint，为查询到的配额上限。