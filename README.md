# 比特币多签指南
多签钱包，顾名思义，指需要多把私钥签名才能控制的钱包。作为一种更安全的资产保管方案，多签钱包诞生已久。但由于流程复杂，目前多签主要被企业/交易所等机构用于管理大量资产。实际上，只要理解了它的基本原理，每个人都可以通过多签来进一步提升资产的安全等级。

本文旨在提供一份详细的比特币多签钱包实践指南，主要面向对钱包、私钥、交易等概念有基本了解的用户（因此文中不会介绍这些基础知识）。对这些概念尚不清楚的读者，文末的扩展阅读部分精选了一系列资料，供您查阅。

本指南在写作过程中大量参考了 Michael Flaxman 的[《比特币十倍安全指南》](https://btcguide.github.io)，Michael 在他的指南中详细地列出了比特币多签涉及到的几乎所有细节。本指南对多签钱包的关键概念进行了梳理，总结了一套比特币多签的基本操作框架，引入了更简洁高效的描述符作为钱包备份方案，并在不牺牲安全性的前提下尽可能地简化了操作流程。读者可遵循本指南提供的方法一步步创建比特币多签钱包，完善自己的资产保管方案。

## 为什么您应该考虑多签钱包？

在开始多签实践之前，明确我们为什么需要使用多签钱包至关重要。

“我已经用上了硬件钱包来保管资产，这还不够安全吗？” 这是关于钱包安全的一个常见误解。不可否认的是，使用硬件钱包比起在线钱包而言，资产的安全性已经上升了一个等级。但实际上，硬件钱包可能遭受的攻击要远比您想象的多，感兴趣的读者可以看下[这个视频](https://www.youtube.com/watch?v=P5PI5MZ_2yo)。

硬件钱包的意义仅在于将私钥与在线设备分离，使得资产免受诸如在线设备遭到攻击/恶意软件 等因素造成的损失，但依然无法消除单签钱包可能导致的其它单点故障，而这些故障对于资产安全而言是致命的。这些单点故障包括但不限于：

* 助记词丢失/损坏
* 助记词被盗
* 硬件钱包遭到供应链攻击/使用了被做过手脚的硬件钱包
* 硬件钱包自身存在安全漏洞
* 计算机/手机被攻击，向硬件钱包传递恶意的待签名交易
* 用于生成私钥的随机数生成器有问题
* ······

针对第一点：助记词丢失/损坏，用户可以通过 多抄录几份助记词/异地备份/使用更坚固的保存介质（比如钢板）等途径来避免其造成的破坏。

针对第二点：助记词被盗，不少用户会采用一些组合策略，包括以下四种形式：

1.助记词乱序保存+单独存放混乱的顺序

2.助记词拆成两份，分开保存

3.助记词使用某种算法加密（比如 AES256）+ 单独保存加密密钥

4.助记词 + 密语（passphrase），使用隐藏钱包

方案一的实际效果非常糟糕，由于助记词的最后一个单词是前面所有单词的校验和，因此暴力破解乱序的助记词难度并不高。即便在真实的助记词中插入了其他单词来混淆视听，也起不到多大帮助，熟练的攻击者甚至只需几个小时便可破解出正确的顺序。针对这一方案的安全性，可以看看 Jameson Lopp 写的这篇[分析文章](https://blog.lopp.net/bitcoin-seed-security-analysis/)。

方案二、三、四在一定程度上提高了安全性，攻击者即便拿到了部分助记词，在缺失剩余组件的情况下依然无法盗取资产，可以看作**简陋**的 “2-2 多签 ”，但这些方案的另一面是更糟糕的容错性，钱包所有者一旦丢失备份中的任一部分，就会丢失资产。并且这些方案无法避免诸如 随机数生成器漏洞/硬件钱包漏洞 等因素带来的安全性风险。

到这里，您可以看出，尽管这些方法在一定程度上提高了助记词的安全性，但在遭遇其它的单点故障时，单签钱包就无能无力了。

相较于以上这些方案，**多签钱包的意义在于提供了一个可以抵御单点故障的保管方案，为资产提供更高的安全性和容错性**。只要遵守了基本的安全原则（**m-n 多签，n>m>1，使用不用品牌的硬件钱包且分开放置**），即便是最简单的多签钱包，其安全性也远胜采用了上述众多方案的单签钱包。攻击者要想盗取您的资产，至少需要破坏两道防线。当然，没有 100% 完美的安全方案。两次以上的致命故障，仍然会让您丢失所有资产。

那么，如何开始创建您的第一个多签钱包呢？

> 由于不同的区块链上实现多签钱包的原理并不相同，本指南提供的方法仅适用于比特币多签钱包，要创建其它区块链上的多签钱包，读者需自行寻找资料。如果读者想通过单一方案来为助记词提供更高的容错性，可以考虑 shamir 助记词分片方案（将助记词分成三份，集齐任意两份即可恢复出完整的助记词）。但针对上文提到的其它单点故障，该方案依然无济于事。

## 如何开始多签？

如果您对多签钱包的基本概念尚不了解，请先阅读[这篇文章](https://www.btcstudy.org/2021/10/18/what-is-multi-signature-wallet-and-recommend-choices/)，然后再进入下文的实践部分。

### 选定多签规则（m-n）
在开始创建多签钱包之前，首先需要选择适合您的多签规则（m-n）：即多签钱包的私钥总数（n）和成功发起交易所需的私钥数量（m）。例如：一个 2-3 的多签钱包代表这个钱包一共由 3 把私钥控制，只需动用其中任意两把私钥即可花费资金。

多签规则的选择因使用场景而异，不同场景的安全需求和风险偏好都不一样，企业用来管理资金而选用的多签规则和个人使用多签来保护资产的规则也不尽相同。并非更多的私钥数量就会带来更高的安全性，它也会带来更高的使用成本和备份难度，同时降低使用时的便利性。此外，m-n 的比例对于钱包的安全性也有重大影响。[这篇文章](https://www.btcstudy.org/2022/05/11/bitcoin-multisig-2-of-3-vs-3-of-5/#安全性离不开取舍)分析了不同多签规则的权衡。

对于个人用户而言，使用 2-3 多签钱包来保护资产安全是在安全性和便利性之间的一个很好的平衡。 2-3 多签在提供了高安全性和容错性的同时，也使得您所需保管的私钥备份数量最少，采购硬件签名器的成本最低。

### 多签基本元素
在 2-3 多签方案中，一共需要 3 把私钥来创建多签钱包。其中任意两把私钥作为签名私钥，用于平时发送交易。第三把私钥则作为恢复私钥，仅在前两把私钥中任意一把发生安全问题时用于恢复多签钱包，转移资产。

![](https://github.com/zengmi2140/multisig/blob/main/img/3-key.png?raw=true)

为了实现安全的多签，我们需要以下基本元素：

* 计算机上的软件钱包：用于创建多签钱包、收款、构造和发送交易
* 硬件钱包：用于离线生成及保存私钥、并作为签名器签署交易
* 钱包备份：用于恢复多签钱包，包括助记词的备份和扩展公钥的备份

> 由于硬件钱包实际上并不具备完整的钱包功能，它的主要作用是签名交易和离线保存私钥。在下文中，该用硬件签名器来称呼硬件钱包，和其它钱包区分开来。

#### 选择合适的软件钱包
当前对比特币多签支持较好的软件钱包有：[Specter](https://specter.solutions)、[Sparrow](https://sparrowwallet.com)、[Electrum](https://electrum.org)、[Nunchuk](https://nunchuk.io/) 等。综合考虑便利性和使用体验后，本文选用 Sparrow 作为示范。文末的扩展阅读部分提供了利用其它软件钱包实现比特币多签的教程。

请注意，无论选用哪款软件钱包，请务必从官网下载并进行验证。在每种钱包的官网，您都可以找到相应的验证方法。

#### 选择合适的硬件签名器（Signer）
为比特币多签选择硬件签名器有两条核心原则：
1. **务必**选用来自不同制造商生产的硬件签名器，这样，即便您的多签方案中有一款硬件签名器存在安全问题，资产依然可以安然无恙。如果都选用同一款硬件签名器，那么面对硬件问题这一单点故障，多签也无济于事。

2. 选择支持 PSBT 的硬件签名器。PSBT （部分签名的比特币交易）是一种标准格式，用于传递待签名的比特币交易，它可以帮助硬件签名器理解和验证当前正在签名的这笔交易信息。每个签名方对 PSBT 签名后，将相应的 PSBT 合并起来，即可形成一笔完整签名的交易，实现跨越不同硬件和软件钱包的多签。PSBT 使得比特币多签更加简单和安全，更详细的介绍可以看[这篇文章](https://www.btcstudy.org/2022/08/15/what-are-partially-signed-bitcoin-transactions-psbts/)。当下在售的支持 PSBT 的硬件签名器有：BitBox、Coldcard、Jade、Keystone、Ledger、NGrave、Passport、Seed Signer。

在符合以上两点原则的前提下， 我们可以根据以下标准来挑选硬件签名器。

1. 尽量选用带有安全芯片的硬件签名器
2. 尽量选用开源的硬件签名器
3. 选用有比特币专属（BTC-Only）固件的硬件签名器：支持山寨币会使得代码库变得更加复杂，引入安全漏洞的可能性也就越大。
4. 尽量选用完全离线（air-gap）的硬件签名器（没有 USB、蓝牙、NFC 等和在线设备直接连接的方式）

不过即便您选用的硬件签名器未能满足上述所有条件，比如选用了尚未开源的 Ledger，或者没有安全芯片的 Seed Signer，并非完全离线的 BitBox 等，由于多签钱包强大的容错能力，资产的安全性依然远高于单签钱包。

现阶段，符合上述所有条件的硬件签名器有：
* [Coldcard](https://coldcard.com)
* [Keystone](https://keyst.one)
* [Passport](https://foundationdevices.com)

请注意：购买硬件签名器唯一靠谱的渠道就是官网。在多签钱包的使用过程中，硬件签名器一定要分开保存在不同的地方。

#### 多签钱包的备份方法
多签钱包的备份有别于普通的单签钱包（一般通过助记词即可恢复），除了需要备份三把私钥的助记词以外，还需要备份私钥对应的扩展公钥。当恢复多签钱包时，需要 m 份私钥的助记词和 n  份扩展公钥，才能花费钱包内的资金。

关于助记词的备份方式，已经有非常多的相关文章，因此本文中不再展开说明。

备份多签钱包的扩展公钥主要有两种方式：多签配置文件和描述符。

##### 多签配置文件
在描述符出现之前，一般通过多签配置文件来备份扩展公钥。多签配置文件包含了构建多签钱包所需的所有信息，包括：

* 多签钱包的地址类型（传统地址还是隔离见证地址）
* 多签的规则（m-n）
* BIP32 路径：从助记词到扩展公钥的派生路径
* 每个签名方的的扩展公钥（xpub）
* 每个签名方的私钥标识符（Master Fingerprint ）

多签配置文件长下面这个样子（不同软件导出的多签配置文件格式上会有出入）：

![](https://github.com/zengmi2140/multisig/blob/main/img/multi-confi.png?raw=true)

有了这些信息，就可以在软件钱包上恢复出完整的多签钱包。

##### 描述符
描述符（Descriptor）是一串包含上述所有关键信息的字符串，精确地描述了如何通过扩展公钥派生出钱包的流程，极大地方便了用户在不同钱包软件之间导入导出钱包的操作。尤其是当我们在使用多签钱包时，描述符可以提升我们在不同软件之间迁移多签钱包的效率，简化备份文件的复杂程度。

描述符长下面这个样子：

![](https://github.com/zengmi2140/multisig/blob/main/img/descriptor-example.png?raw=true)

目前主流的比特币软件钱包均已支持描述符，因此本指南更推荐您使用描述符作为多签钱包的备份方案。

注意：任何获得您多签配置文件或者描述符的人，都可以在他的计算机上恢复出您的多签钱包，这并不会影响您的资金安全（因为没有满足条件的私钥就无法签名交易），但会泄漏钱包余额和交易历史等私人信息。因此，建议将多签钱包离线保存，或保存在密码管理器中。

### 多签的基本流程

一笔多签交易的基本流程如下：

1. 在计算机上利用软件钱包构造交易
2. 将待签名的交易传递给硬件签名器进行签名（通过 PSBT 格式）
3. 硬件签名器验证交易信息，签名交易。
4. 将已签名的交易传回计算机
5. 计算机收集到足够的签名后（在 2-3 多签中为 2 份签名），广播交易
6. 交易被打包上链

![multisig-step-by-step.png](https://github.com/zengmi2140/multisig/blob/main/img/multisig-step-by-step.png?raw=true)


## 动手实践吧（以 2-3 多签为例）

### 设备清单
![设备清单](https://github.com/zengmi2140/multisig/blob/main/img/equipment.jpeg?raw=true)

1. 硬件签名器两台，本指南选用的硬件签名器为 Coldcard MK3（$157） 和 Keystone Pro（$169）
2. 一台带有摄像头的计算机，用于安装比特币钱包作为观察钱包，来协调多签交易
3. 一粒标准骰子，作为私钥生成过程中的随机数来源
4. 一张 Micro SD 卡（存储空间 <= 32GB，FAT32 格式）及对应的读卡器
5. 纸和笔，用来备份钱包助记词

### 准备工作
#### 计算机端
在计算机上下载、验证和安装最新版的 Sparrow 钱包作为观察钱包（Windows/Mac/Linux 均适用）：
https://www.sparrowwallet.com/download/

按照此指南快速设置 Sparrow 钱包：
https://www.sparrowwallet.com/docs/quick-start.html

> 在 Sparrow 初始设置阶段，会让您选择 Sparrow 连接的服务器。对于只想体验多签钱包，或者管理的资金不多的用户而言，可以选择公共服务器（Public Server），但这会泄漏您的隐私。公共服务器的运营者可以很容易地将您的钱包地址和身份联系起来。如果打算存储大量的资金，强烈建议连接到自己运行的比特币节点。

#### 硬件签名器
1.从**官网**购买硬件签名器：

* Coldcard 官网： https://coldcard.com/
* Keystone 官网： https://keyst.one/

2.收到钱包后，首先验证是否被动过手脚，查验真伪，然后进行初始化设置，熟悉钱包的使用。Coldcard 和 Keystone 都采用了优秀的防伪机制，方便用户进行验证。教程如下：

Keystone 产品验证和初始化设置教程： https://support.keyst.one/v/traditional-chinese/kai-shi-shi-yong/kai-shi-shi-yong-keystone-qian-bao-new#di-1-bu-guan-wang-yan-zheng

Coldcard 产品验证和初始化设置教程： https://coldcard.com/docs/ultra-quick

3.升级钱包到最新固件

Coldcard 固件升级教程： https://coldcard.com/docs/upgrade

固件验证教程： https://www.youtube.com/watch?v=RYcB5HpfcaE

Keystone 出厂默认为多币种固件，请切换为比特币单币种（BTC-Only）固件，在[此处](https://keyst.one/firmware)下载固件，并按照此教程进行升级： https://support.keyst.one/v/traditional-chinese/kai-shi-shi-yong/ru-he-sheng-ji-gu-jian

固件验证教程： https://support.keyst.one/v/traditional-chinese/kai-shi-shi-yong/ru-he-sheng-ji-gu-jian/ru-he-jian-cha-guan-wang-sheng-ji-bao-de-sha256sum

### 创建及备份多签钱包

#### 创建三个私钥并导出扩展公钥

此过程需要注意如下三点：

1. 三把私钥都要在完全离线的环境中创建，然后导出扩展公钥（创建多签钱包时要用到，下文会给出导出方法）。
2. 为了避免随机数生成器可能存在漏洞而造成安全影响，同时简化操作流程，本指南中两把签名私钥分别由 coldcard 和 keystone 生成，恢复私钥的创建则使用掷骰子来生成随机数。
3. 三把私钥创建好后都要执行验证流程（创建钱包 > 抄下助记词 > 删除钱包 > 重新导入抄好的助记词 > 验证地址是否一致 > 完成验证），并做好备份，分开存放。

> Keystone 验证流程：设置 > 恢复出厂设置，然后重新初始化钱包，通过助记词导入钱包
> 
> Coldcard 验证流程：Advanced > Danger Zone > Seed Functions > Destroy Seed，然后重新通过助记词导入钱包

![](https://github.com/zengmi2140/multisig/blob/main/img/generate-key.png?raw=true)

首先，我们通过掷骰子生成恢复私钥（下文用 Key3 指代恢复私钥）:

出于操作上的便利性，本指南选用 Keystone 来完成掷骰子生成私钥的操作，方法如下：
https://support.keyst.one/v/traditional-chinese/gao-ji-gong-neng/zhu-ji-ci/shi-yong-tou-zi-chuang-jian-zhu-ji-ci

> Coldcard 也同样支持[掷骰子生成私钥](https://www.youtube.com/watch?v=Rc29d9m92xg)，另一款常用的工具是 [SeedPicker](https://seedpicker.net/)。

在完成 Key3 的创建、备份和验证后，将 Micro SD 卡插入 Keystone，点击 “多签钱包”，选择 “通用钱包”，按照下图所示步骤导出 Key3 对应的扩展公钥后，保存到计算机上。

![](https://github.com/zengmi2140/multisig/blob/main/img/keystone-export-xpub-0.png?raw=true)

![](https://github.com/zengmi2140/multisig/blob/main/img/keystone-export-xpub.png?raw=true)

接着，将 Keystone 恢复出厂设置，重新完成初始化设置后，创建新的种子助记词，作为第一把签名密钥 （Key1），然后按照上面的方法再次导出扩展公钥文件，保存至计算机。

最后，在 Coldcard 上生成新钱包，作为第二把签名密钥（Key2）。然后将 Micro SD 卡插入 Keystone，按照下图所示步骤导出扩展公钥，保存到计算机。

![](https://github.com/zengmi2140/multisig/blob/main/img/export-xpub-form-coldcard.jpeg?raw=true)

>如果不嫌麻烦的话，您也可以选择通过掷骰子的方式来生成三把私钥的助记词，彻底断绝随机随机数生成器可能带来的安全影响。

在完成上述步骤后，我们有了：
* 三把 Key 对应的助记词备份，请分开保存
* 三份与 Key 对应的扩展公钥文件

有了这三份扩展公钥文件，下面开始创建多签钱包。

#### 在 Sparrow 上创建多签钱包

在计算机上打开 Sparrow，在菜单栏中依次选择 “File” > “New Wallet”，然后在弹出的窗口中自定义钱包的名称，点击 “Create Wallet”：

![sparrow-new-wallet.png](https://github.com/zengmi2140/multisig/blob/main/img/sparrow-new-wallet.png?raw=true)

接下来会进入 Sparrow 的主界面：

![sparrow-1.png](https://github.com/zengmi2140/multisig/blob/main/img/sparrow-1.png?raw=true)

首先，在 Policy Type 中选择 Multi Signature，即表明当前创建的为多签钱包。确认您的界面和上图中第 2、3 部分一致，即脚本类型为 P2WSH（原生隔离见证地址），多签规则为 2/3，这是 Sparrow 默认的设置。

> 交易中包含签名越多，手续费就越高，使用隔离见证交易可以降低手续费。

然后在图中下方 Keystore 部分导入我们前面准备的三份扩展公钥文件。方法：依次选择 Keysotre1、2、3 选项卡，点击 “Airgapped Hardware Wallet” ，在弹出的界面中根据扩展公钥的来源选择对应的导入方式，然后导入相应的文件。

以 Keystone 为例：

![sparrow-import-from-keystone-1.png](https://github.com/zengmi2140/multisig/blob/main/img/sparrow-import-from-keystone-1.png?raw=true)

![sparrow-import-from-keystone-2.png](https://github.com/zengmi2140/multisig/blob/main/img/sparrow-import-from-keystone-2.png?raw=true)

导入后界面如下：

![import-json.png](https://github.com/zengmi2140/multisig/blob/main/img/import-json.png?raw=true)

依次导入三个扩展公钥后，点击右下角 “Apply”。

![apply-multi-wallet.png](https://github.com/zengmi2140/multisig/blob/main/img/apply-multi-wallet.png?raw=true)

Sparrow 会弹出一个窗口让您刚创建的这个钱包设置密码，该密码仅用于在计算机上解锁这个多签钱包（即在每次打开 Sparrow 进入该钱包时使用），也可选择 No Password 跳过。

接下来 Sparrow 会弹出备份窗口，提醒我们备份多签钱包。

下图中蓝色方框内这一长串字符（以 wsh 开头）便是刚创建的这个多签钱包的描述符，我们需要妥善保存。当我们想要恢复多签钱包时，除了要用到上面备份的三把私钥外，还需要加上钱包描述符一起，才能完成恢复。

点击 “Save PDF” 保存描述符（一份包含二维码和描述符的 PDF），同时可以复制蓝框内的描述符，保存一份到密码管理器中。

> 对于重要文件的保存，应该遵循 “3-2-1” 原则。即：至少保存三份备份 —— 存储于两种不同的介质中 —— 至少一份存放在异地。

然后点击右下角 “确定”。

![save-sparrow-descriptor.png](https://github.com/zengmi2140/multisig/blob/main/img/save-sparrow-descriptor.png?raw=true)

至此，我们的多签钱包创建完成。

#### 验证多签钱包备份
在正式使用钱包之前，我们需要检验一下是否正确备份了用于恢复钱包的描述符。

在 Sparrow 中按照新建一个钱包，依次选择 “File” > “New Wallet” > “Create Wallet”，流程和上文一致。

在钱包主界面点击 Descriptor 输入框右侧的 ”Edit...“ 

![backup-test-1.png](https://github.com/zengmi2140/multisig/blob/main/img/backup-test-1.png?raw=true)

在弹出的窗口中，删除蓝框内原有字符串，然后将前面备份好的描述符粘贴进蓝框，最后点击 “确定”。

![backup-test-2.png](https://github.com/zengmi2140/multisig/blob/main/img/backup-test-2.png?raw=true)

点击确认后，Sparrow 会根据输入的描述符自动设置好钱包的各项参数。比对新钱包里各项参数和之前创建的多签钱包是否一致。（比对 Keystore1、2、3 的 Master Fingerprint、Derivation、xpub，Keystore 的先后排序可能会乱，但没有影响。）参数一致的话，证明描述符备份无误。

![backup-test-3.png](https://github.com/zengmi2140/multisig/blob/main/img/backup-test-3.png?raw=true)

最后，删除新建的钱包。

#### 将多签钱包导入硬件签名器：
在 Sparrow 里创建多签钱包后，我们需要将多签钱包导入硬件签名器。

在 Sparrow 中点击左下角 “Export...”，然后选择对应的导出方式：

![sparrow-export.png](https://github.com/zengmi2140/multisig/blob/main/img/sparrow-export.png?raw=true)

将多签钱包导入 Keystone：

点击 “Export” 后，在弹出的窗口中选择 Keystone Multisig，点击 “Show...”

![export-to-keystone.png](https://github.com/zengmi2140/multisig/blob/main/img/export-to-keystone.png?raw=true)

此时计算机屏幕上会出现一个动态二维码。打开 Keystone，点击 “多签钱包” > “导入多签钱包” >扫描计算机屏幕上的动态二维码。

扫描完成后，此时 Keystone 会显示正在导入的多签钱包的具体信息， 将 Keystone 屏幕上的信息和 Sparrow 的信息进行一一比对，确认无误后在 Keystone 上点击 “确认”，至此，导入完成。

> 点击 “确认” 后，Keystone 屏幕上可能会出现钱包验证码，直接点击 “验证码一致，继续导入” 即可。此校验码仅用于在两台不同的 Keystone 上配置多签钱包时使用，在本文提供的方案中可以忽略。

![keystone-import-multisig.png](https://github.com/zengmi2140/multisig/blob/main/img/keystone-import-multisig.png?raw=true)

需要注意的是：导入过程中 Keystone 屏幕上显示的多签钱包信息中扩展公钥以 Zpub 开头，Sparrow 上默认以 Xpub 开头。为方便比对，可以在 Sparrow 里按下图所示点击右下角转换图标，切换 xpub 和 Zpub 显示。

![xpub-zpub.png](https://github.com/zengmi2140/multisig/blob/main/img/xpub-zpub.png?raw=true)


将多签钱包导入 Coldcard：

点击 “Export” 后，在弹出的窗口中选择 Coldcard Multisig，点击 “Export File...”，此步骤会导出一个 .txt 文件。

![export-to-coldcard.png](https://github.com/zengmi2140/multisig/blob/main/img/export-to-coldcard.png?raw=true)

将该文件存入 Micro SD 卡，然后将 Micro SD 卡插入 Coldcard，按下图所示步骤依次选择 Setting > Multisig Wallets > Import form SD > 选择刚刚的文件名 > 和 Sparrow 比对钱包信息是否无误 >  按照 Coldcard 的提示确认导入。

![coldcard-import-multisig.jpeg](https://github.com/zengmi2140/multisig/blob/main/img/coldcard-import-multisig.jpeg?raw=true)

![](https://github.com/zengmi2140/multisig/blob/main/img/coldcard-save-address.jpeg?raw=true)


### 开始交易

完成上述所有步骤后，就可以使用比特币多签钱包了。使用前务必先用小额资金进行测试。多签钱包使用方式如下：

#### 验证收款地址
在使用多签钱包接受资金之前，我们首先需要验证接收地址。为了安全地接收资金，我们应当默认联网的计算机是不安全的，即计算机上显示的地址可能是黑客给的虚假信息。因此，我们需要分别在 Keystone 和 Coldcard 上验证收款地址：

首先，查看 Sparrow 的 收款地址。

在 Sparrow 中点击左侧的 “Receive”，可以看到 Sparrow 给出的接收地址。

![verify-address.png](https://github.com/zengmi2140/multisig/blob/main/img/verify-address.png?raw=true)

选择左侧的 “Address” 可以看到更多地址。

![verify-address-all.png](https://github.com/zengmi2140/multisig/blob/main/img/verify-address-all.png?raw=true)


在 Keystone 上验证收款地址。

解锁 Keystone 后，点击 ”多签钱包“，即可看到接受地址（点击某个地址可以看到完整的地址信息）。可以点击左下角 “+” 号创建多个地址，和 Sparrow 上的地址进行比对。

![keystone-verify-address.jpeg](https://github.com/zengmi2140/multisig/blob/main/img/keystone-verify-address.jpeg?raw=true)

在 Coldcard 上验证收款地址。

解锁 Coldcard 后，依次选择 Address Explorer > 按照屏幕上的指示继续 > 往下滚动选择您导入的多签钱包 > 往下滚动即可看到接收地址。

![coldcard-verify-address.jpeg](https://github.com/zengmi2140/multisig/blob/main/img/coldcard-verify-address.jpeg?raw=true)

> 您还可以按照屏幕上的指示将接收地址存储到离线的 micro SD 卡中（一次可导出 250 条地址信息），这样就不用每次接收资金时都打开 coldcard 验证接收地址。毕竟两个硬件签名器在日常使用时是分开存放的，每次收款前都打开两个硬件签名器验证地址过于麻烦。但应在至少一处硬件签名器的屏幕上验证接受地址。

至此，我们完成了接收地址的验证流程，可以使用该地址接受比特币啦。

#### 发送交易

首先，我们需要在 Sparrow 中创建交易。

在 Sparrow 中点击左侧的 “Send”，依次填写交易信息，设置合理的手续费后，点击右下角的 “Create Transaction”。

![create-tx.png](https://github.com/zengmi2140/multisig/blob/main/img/create-tx.png?raw=true)

此时，Sparrow 会新建一个页面，展示交易的详细信息，验证无误后，点击 “Finalize Transaction for Signing”。

![finalize-tx.png](https://github.com/zengmi2140/multisig/blob/main/img/finalize-tx.png?raw=true)

至此完成了交易的创建。接下来需要将这笔待签名的交易传给 Keystone 和 Coldcard。分别在 Keystone 和 Coldcard 上完成签名，再将签名好的交易传回 Sparrow。

使用 Keystone 签名交易：

点击下图中的 “Show QR”，此时计算机屏幕上会出现一个动态二维码，这个二维码展示的就是待签名的交易信息。

![show-qr-to-keystone.png](https://github.com/zengmi2140/multisig/blob/main/img/show-qr-to-keystone.png?raw=true)

解锁 Keystone，选择 ”多签钱包“，按照下图所示点击右上方扫描图标，出现扫码界面。扫描计算机屏幕上的二维码，完成后 Keystone 屏幕上会出现交易确认界面。确认交易信息无误后点击 ”签名“，验证密码后，Keystone 的屏幕上会出现一个动态二维码，这个二维码展示的就是已完成部分签名的交易信息。

![keystone-sign-tx.png](https://github.com/zengmi2140/multisig/blob/main/img/keystone-sign-tx.png?raw=true)

在 Sparrow 中点击 “Scan QR”，Sparrow 会调用计算机的摄像头，扫描 Keystone 屏幕上的二维码，将 Keystone 上已经签名好的交易传给 Sparrow。

![scan-qr-from-keystone.png](https://github.com/zengmi2140/multisig/blob/main/img/scan-qr-from-keystone.png?raw=true)

使用 Coldcard 签名交易：

在 Sparrow 中点击 “Save Transaction”，此时 Sparrow 会生成一个 “xxxx.psbt” 的文件（xxxx 为创建交易时自定义的 “Label” 字段），将该文件存至 Micro SD 卡，插入 Coldcard。

![save-tx-to-coldcard.png](https://github.com/zengmi2140/multisig/blob/main/img/save-tx-to-coldcard.png?raw=true)

解锁 Coldcard，选择 “Ready to Sign”，屏幕上会展示 Micro SD 卡内这笔交易的详情，上下滚动确认无误后，按屏幕提示选择确认，Coldcard 会完成签名，生成一个新的 PSBT 文件。

![coldcard-sign-tx.jpeg](https://github.com/zengmi2140/multisig/blob/main/img/coldcard-sign-tx.jpeg?raw=true)

从 Coldcard 中拔出 Micro SD 卡，插入计算机。在 Sparrow 中点击 “Load Transaction”，选择刚刚签名好的文件（以 xxxx-part.psbt 格式命名）。

![load-tx-form-coldcard.png](https://github.com/zengmi2140/multisig/blob/main/img/load-tx-form-coldcard.png?raw=true)

最后，点击 “Broadcast Transaction” 即可广播交易。

![sparrow-boardcast-tx.png](https://github.com/zengmi2140/multisig/blob/main/img/sparrow-boardcast-tx.png?raw=true)

至此，我们已经走完了发送交易所需的所有步骤。


### 从测试网开始！
对于初次尝试比特币多签的读者，强烈建议您先使用测试网走一遍完整的流程。熟悉各项操作后，再创建自己的多签钱包，以免操作失误造成资金损失。以下为使用比特币测试网（Testnet）进行演练的方法。

#### 各钱包打开测试网模式的方法：
Sparrow
菜单中找到 “Tools”，选择 “Restart in Testnet”，Sparrow 会重新打开，进入测试网模式。

Coldcard
解锁 Coldcard，依次选择 Advanced > Danger Zone > Testnet Mode > Testnet: BTC。

![coldcard-testnet-mode.jpeg](https://github.com/zengmi2140/multisig/blob/main/img/coldcard-testnet-mode.jpeg?raw=true)

Keystone
打开设置 > 区块链网络 > 测试网 ，此时界面顶部会变成黄色。

![keystone-testnet.png](https://github.com/zengmi2140/multisig/blob/main/img/keystone-testnet.png?raw=true)

以下网站均可领取测试网比特币：
* https://bitcoinfaucet.uo1.net/
* https://onchain.io/bitcoin-testnet-faucet
* https://testnet.qc.to
* https://tbtc.bitaps.com
* https://testnet.help/en/btcfaucet/testnet

> 测试网比特币并无实际价值，请使用完后发回水龙头，供其他人继续使用。

### 紧急恢复

当在使用多签钱包的过程中遇到以下情况时，需要对多签钱包进行恢复操作。

#### 助记词没有出现安全问题的情况

硬件签名器故障：比如硬件签名器遭到物理损坏或出现硬件/软件上的故障，无法正常使用。

解决方式：买一个新的同款的硬件签名器，导入原来的助记词。然后按照上文的步骤将多签钱包导入新的硬件签名器，即可使用新钱包来进行签名。如果买不到同款钱包，则按照上文的硬件签名器选择标准购买其它款的硬件签名器。

#### 助记词出现了安全问题的情况
此情况包括如下几种情形：

1. 助记词备份破损，无法恢复
2. 助记词备份丢失，无法找回
3. 助记词被盗，或已经被他人看过（在线存储过的助记词应默认已被他人看过）
5. 硬件签名器被曝出存在安全问题
6. 硬件签名器丢失或被盗

当遇到以上情况时，应当视为助记词已出现/即将出现安全问题。我们需要立刻执行紧急恢复。如果是硬件签名器存在安全问题导致的助记词安全隐患，还需在紧急恢复的过程中替换掉有问题的硬件签名器。

紧急恢复流程如下：

1. 创建一把新的私钥，替换原来三把私钥中出现安全问题的那把私钥
2. 使用新的私钥和原有的两把安全的私钥按照上文的流程创建新的多签钱包
3. 将原多签钱包内的资金转至新的多签钱包

下图使用 Key3 代表出现安全问题的私钥，注意和前文中的恢复私钥区分。

![recovery-multisig.png](https://github.com/zengmi2140/multisig/blob/main/img/recovery-multisig.png?raw=true)


#### 额外的安全措施（可选）：
为了能够第一时间发现安全问题，可以在每个私钥对应的单签钱包内放入少量资金，并将对应的地址添加到观察钱包。这样一旦三把私钥中某一把私钥被盗（可能是助记词被盗，也可能是存放该私钥的硬件签名器出了安全问题），就能及时发现，然后迅速转移多签钱包内的资金。

## 结语

感谢您能读到这里。

我们常说 “复杂性是安全的敌人”。相较于单签钱包而言，多签钱包对使用者提出了更高的要求，比如更麻烦的使用流程，更多的备份管理工作，更高的成本 ······

实际上，想要实现完美无缺的单签钱包比多签钱包更为困难，而后者要比起前者要安全得多。其实，多签钱包烦琐的设置主要集中在创建钱包这一过程，只要理解了它的基本原理，就可以消除对于对于复杂性的恐惧。

复杂性不一定是安全的敌人，但怕麻烦一定是。

## 扩展阅读
1. [什么是多签钱包？](https://www.btcstudy.org/2021/10/18/what-is-multi-signature-wallet-and-recommend-choices/)
2. [比特币的私钥：转码与使用](https://www.btcstudy.org/2020/09/04/understand-bitcoin-public-and-private-keys-encode-and-extend/)
3. [比特币交易工作原理简介](https://www.btcstudy.org/2022/09/29/how-does-a-bitcoin-transaction-actually-work/)
4. [囤币路上的 21 个坑](https://www.btcstudy.org/2022/07/27/21-ways-lose-bitcoin/#19-不合理的多签人数阈值)
5. [多签和拆分备份：让您的比特币更安全](https://www.btcstudy.org/2021/10/07/multisig-and-split-backups-two-ways-to-make-your-bitcoin-more-secure/)
6. [助记词备份指南](https://www.btcstudy.org/2022/05/07/how-to-back-up-a-seed-phrase/)
7. [深冷存储：如何更安全保管您的助记词](https://mirror.xyz/keystonecn.eth/fhdm_LSTt3SEg5DkV0M8_b4Ix5kC5E4bhwcFLQx9iUc)
8. [2-of-3 vs. 3-of-5：更多密钥一定更安全吗？](https://www.btcstudy.org/2022/05/11/bitcoin-multisig-2-of-3-vs-3-of-5/)
9. [Keystone 為什麼要增加分片助記詞功能](https://mirror.xyz/keystonecn.eth/jngKwNznJARo5PdAlYxzuibDT9DWyvDkkSMpgjsBwQM)
10. 利用 Specter 实现比特币多签[教程1](https://btcguide.github.io)、[教程2](https://bitcoiner.guide/multisig/)
11. 利用 Electrum 实现比特币多签[教程1](https://armantheparman.com/msig/)、[教程2](https://support.keyst.one/v/traditional-chinese/di-san-fang-qian-bao/bi-te-bi-qian-bao/electrum/electrum-23-psbt-duo-qian-jiao-cheng)
12. 利用 Nunchuk 实现比特币多签[教程](https://www.youtube.com/watch?v=yWLzkaxI7og)
13. [多签钱包的已知问题](https://btcguide.github.io/known-issues/multisig)

## 参考资料
1. [10x Security Bitcoin Guide](https://btcguide.github.io)
2. [Bitcoin Multisig Guide](https://bitcoiner.guide/multisig/)
3. https://walletsrecovery.org
4. [软硬钱包概说](https://www.btcstudy.org/2022/08/20/overview-of-soft-wallets-and-hard-wallets-signers/)
5. [What is a multisig wallet configuration file and what is it for?](https://unchained.com/blog/what-is-a-multisig-wallet-configuration-file/)
6. [什么是 “部分签名的比特币交易（PSBT）”？](https://www.btcstudy.org/2022/08/15/what-are-partially-signed-bitcoin-transactions-psbts/)
7. [什么是输出描述符？](https://www.btcstudy.org/2022/05/10/what-are-output-descriptors/)
8. [When do I need to replace a key for my bitcoin multisig wallet?](https://unchained.com/blog/replace-key-multisig-wallet/)
9. [Best Hardware Wallets in 2022](https://blog.thebitcoinhole.com/best-hardware-wallets-31141ed1aa05)
