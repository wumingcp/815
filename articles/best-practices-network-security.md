<properties
   pageTitle="Microsoft 云服务与网络安全 | Windows Azure"
   description="了解 Azure 中用于创建安全网络环境的某些重要功能"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
   ms.service="virtual-network"
   ms.date="09/16/2015"
   wacn.date="11/12/2015"/>

# Microsoft 云服务与网络安全

Microsoft 云服务提供超大规模的服务和基础结构、企业级的功能，以及许多混合连接选项。客户可以选择通过 Internet 或 ExpressRoute（提供专用网络连接）访问这些服务。Windows Azure 平台可让客户无缝地将基础结构扩展到云中并构建多层体系结构。为了补充 Microsoft 的服务，第三方可以提供安全服务和虚拟设备，以启用增强的功能。本白皮书概述了当客户使用通过 ExpressRoute 访问的 Microsoft 云服务以及在 Windows Azure 的虚拟网络中创建安全服务时应该考虑的安全和体系结构问题。

## 快速开始
以下逻辑图表以具体示例说明了 Windows Azure 平台提供的许多安全技术。使用此图表可以快速了解具体需要做好哪些准备。你也可以从头开始阅读本文档以了解许多可用的选项。![安全选项流程图][0]

[示例 1 – 构建外围网络以通过 NSG 保护应用程序](#example-1-build-a-simple-dmz-with-nsgs)</br>
[示例 2 – 创建外围网络以通过防火墙和 NSG 保护应用程序](#example-2-build-a-dmz-to-protect-applications-with-a-firewall-and-nsgs)</br>
[示例 3 – 构建外围网络以通过防火墙、UDR 和 NSG 保护网络](#example-3-build-a-dmz-to-protect-networks-with-a-firewall-udr-and-nsg)</br>
[示例 4 – 使用站点到站点虚拟设备 VPN 添加混合连接](#example-4-adding-a-hybrid-connection-with-a-site-to-site-virtual-appliance-vpn)</br>
[示例 5 – 使用站点到站点 Azure 网关 VPN 添加混合连接](#example-5-adding-a-hybrid-connection-with-a-site-to-site-azure-gateway-vpn)</br>
[示例 6 – 使用 ExpressRoute 添加混合连接](#example-6-adding-a-hybrid-connection-with-expressroute)</br>
未来几个月内，本文档中将会加入有关添加 VNet 到 VNet 连接、高可用性和服务链接的示例。

## Microsoft 合规性与基础结构保护
Microsoft 为企业客户所需的合规性方案提供首屈一指的支持。下面是 Azure 取得的部分认证：![Azure 合规性徽章][1]

<!--有关更多详细信息，请访问：[http://azure.microsoft.com/support/trust-center/compliance/](http://azure.microsoft.com/support/trust-center/compliance/)-->

Microsoft 采取综合性的方案来保护运行超大规模全球服务所需的云基础结构。Microsoft 云基础结构包括硬件、软件、网络、管理和运营人员以及物理数据中心。

![Azure 安全功能][2]

上述方案提供安全的基础，使客户能够将服务部署在 Microsoft 云中。客户要采取的后续措施是设计和创建安全体系结构来保护这些服务。

## 传统的安全体系结构与外围网络 (DMZ)
尽管 Microsoft 为保护云基础结构投入了大量资金，但客户仍必须保护其云服务和资源组。安全的多层方案提供最佳的防御措施。安全外围网络 (DMZ) 可防止不受信任的网络访问内部网络资源。外围网络是企业 IT 网络安全体系结构中众所周知的概念，是指位于 Internet 与受保护企业 IT 基础结构之间的网络边缘或组件。

在典型的企业网络中，核心基础结构的周边有多层的安全设备严加保护。每一层的边界由设备和策略实施点组成。设备可能包括防火墙、DDoS（分布式拒绝服务）预防、IDS/IPS（入侵检测或保护系统）、VPN（虚拟专用网络）设备等。策略可能以防火墙策略、ACL 或特定路由等形式实施。网络的第一道防线（直接接受来自 Internet 的传入流量）由这些机制联合起来阻挡攻击和有害流量，但允许合法的请求进入网络。此流量直接路由到外围网络中的资源。该资源可能与网络中更深处的资源“对话”，在进一步路由到网络之前，需通过下一个边界的验证。最外层称为外围网络，因为网络的这个部分对 Internet 公开，而外围网络的两端通常有某种形式的保护。下图显示了企业网络中的单子网外围网络示例，其中有两个安全边界（外围网络 Internet 及外围网络后端 VLAN）。

![企业网络中的外围网络][3]

有许多体系结构可用于实施外围网络，从 Web 场前面的简单负载平衡器到多子网外围网络，每个边界上设有各种机制来阻止流量，并保护企业网络的更深入层。如何构建外围网络取决于组织的特定要求和相关的风险容限度。

当客户将工作负荷移到公有云时，Azure 中的外围网络体系结构必须支持类似的功能，才能满足合规性和安全性要求。本文提供有关客户如何在 Azure 中构建安全网络环境的指导，其中重点介绍了外围网络，但也全面讨论了网络安全的许多层面，从以下问题开始（不限于这些问题）：

1.	如何在 Azure 中构建外围网络？
2.	有哪些 Azure 功能可用来构建外围网络？
3.	如何保护后端工作负荷？
4.	如何控制 Internet 与 Azure 中工作负荷的通信？
5.	如何防止本地网络受到 Azure 中的部署的影响？
6.	何时使用本机 Azure 安全功能？何时又该使用第三方设备或服务？

下图显示了 Azure 以 Azure 平台本身内置的功能和通过客户定义的功能，提供给客户的各种安全层：

![Azure 安全体系结构][4]

对于来自 Internet 的入站流量，Azure DDoS 保护将会监视是否存在针对 Azure 的大规模攻击。通过此安全层后，将会到达客户定义的公共终结点，这些终结点用于判断哪些流量可以通过云服务进入虚拟网络。本机 Azure 虚拟网络隔离可确保与其他所有网络完全隔离，而且流量只能流经用户配置的路径和方法。这些路径和方法就是下一个安全层，在该层中，可以使用网络安全组 (NSG)、用户定义的路由 (UDR) 和网络虚拟设备来创建安全边界（包括外围网络），以保护受保护网络中的应用程序部署。

下一部分提供 Azure 虚拟网络的概述。Azure 虚拟网络由客户创建，它们是部署的工作负荷所要连接的目标。虚拟网络是创建外围网络来保护 Azure 中的客户部署时所需一切网络安全功能的基础。

## Azure 虚拟网络概述
在 Internet 流量进入 Azure 虚拟网络之前，Azure 平台本身将实施两层安全性：

1.	**DDoS 保护**：分布式拒绝服务保护 (DDoS) 是一个 Azure 物理网络层，可防范 Azure 平台本身受到大规模 Internet 攻击（攻击者利用多个“傀儡”节点尝试瘫痪 Internet 服务）。Azure 使用一个强大的 DDoS 保护网来检测所有入站 Internet 连接。此 DDoS 保护层没有用户可配置的属性，客户也无法对其进行访问。这可以防止 Azure 平台受到大规模攻击，但不直接保护单个客户应用程序。客户可以配置额外的弹性层来防止局部攻击。例如，如果客户 A 在公共终结点上遭到大规模 DDoS 攻击，Azure 将封锁该服务的连接。客户 A 可以故障转移到另一个未遭受攻击的虚拟网络或服务终结点，以还原服务。请注意，当客户 A 在该终结点上受到影响时，该终结点以外的其他任何服务不会受到影响。此外，其他客户和服务也不会受到此攻击的影响。
2.	**服务终结点**：终结点可让云服务或资源组公开公共的（在 Internet 上）IP 地址和端口，终结点将流量经由 NAT 发送到 Azure 虚拟网络上的内部地址和端口。这是外部流量进入 Azure 虚拟网络的主要路径。用户可以配置服务终结点，以确定可以传入哪些流量、如何在虚拟网络上转换该流量以及要将它路由到何处。 

流量进入虚拟网络后，有许多功能将派上用场，因为 Azure 虚拟网络是客户连接其工作负荷的基础，也是应用基本网络层安全的所在之处。它是客户在 Azure 中的专用网络（虚拟网络覆盖），并具有以下功能和特性：
 
1.	**流量隔离**：虚拟网络是 Azure 平台上的流量隔离边界。一个虚拟网络中的 VM 无法与不同虚拟网络中的 VM 直接通信，即使这两个虚拟网络是由同一个客户所创建。这是一个非常关键的属性，可确保客户 VM 与通信在虚拟网络中保持私密性。 
2.	**多层拓扑**：虚拟网络允许客户通过分配子网并为工作负荷的不同元素或“层”指定独立地址空间，来定义多层拓扑。这些逻辑分组和拓扑可让客户根据工作负荷类型来定义不同的访问策略，以及控制各层之间的流量。 
3.	**跨界连接**：客户可以通过 Azure VPN 网关或第三方网络虚拟设备，在虚拟网络和多个本地站点或 Azure 中的其他虚拟网络之间创建跨界连接。Azure 支持使用标准 IPsec/IKE 协议和 ExpressRoute 专用连接的站点到站点 (S2S) VPN。 
4.	**网络安全组** (NSG) 允许客户根据所需的粒度（网络接口、单个 VM 或虚拟子网）创建规则 (ACL)。客户可以从客户网络上的系统，通过跨界连接或直接 Internet 通信来允许或拒绝虚拟网络内的工作负荷，以控制访问。 
5.	**用户定义的路由** (UDR) 和 **IP 转发**允许客户定义虚拟网络中不同层之间的通信路径。客户可以部署防火墙、IDS/IPS 和其他虚拟设备，并通过这些安全设备来路由网络流量，以实施安全边界策略、审核和检查。
6.	Azure 应用商店中的**网络虚拟设备**：Azure 应用商店和 VM 映像库中提供了防火墙、负载平衡器和 IDS/IPS（入侵检测/预防服务）等安全设备。客户可将这些设备部署到其虚拟网络，特别是安全边界（包括外围网络子网），以实现多层安全网络环境。

以下示例演示了如何使用这些功能在 Azure 中构造外围网络体系结构：

![Azure 虚拟网络中的外围网络][5]

## 外围网络特征与要求
本部分介绍 Azure 中外围网络的特征和要求。如前所述，外围网络主要作为网络的前端，直接应对来自 Internet 的通信。传入数据包应该在流经安全设备（即外围网络中的防火墙、ID、IPS 等）之后才到达后端服务器。来自工作负荷的 Internet 绑定数据包可能也会流经外围网络中的安全设备，经过策略实施、检查和审核之后，才离开网络。此外，外围网络还可用来托管客户虚拟网络与本地网络之间的跨界 VPN 网关。

### 外围网络的特征
参考上图中 Azure 虚拟网络内的某个外围网络。良好外围网络的一些特征如下：

- 外围网络面向 Internet（前端）：
    - 外围网络子网本身面向 Internet，直接与 Internet 通信
    - 公共 IP、VIP 和/或服务终结点将 Internet 流量传递给前端网络和设备
    - 来自 Internet 的入站流量先通过安全设备，然后再通过前端网络上其他资源
    - 如果已启用出站安全性，则流量先通过安全设备（最后一个步骤），然后才传递到 Internet
- “受保护网络”（后端）：面向核心基础结构
    - Internet 与核心基础结构之间没有直接的路径
    - 连往核心基础结构的通道必须遍历外围网络中的 NSG、防火墙或 VPN 设备等安全设备
    - 外围网络中的其他设备不得桥接 Internet 和核心基础结构
    - 在面向 Internet 的外围网络和面向受保护网络边界的外围网络上（例如上图所示的两个防火墙图标），安全设备实际上可能是单个虚拟设备，但有不同的规则或接口处理 Internet 边界和后端边界（即，一个在逻辑上分隔的设备，负责处理外围网络两个边界的负载）
- 其他常见做法与约束
    - 外围网络中的工作负荷不得存储业务关键型信息
    - 只有获授权的管理员才能访问和更新外围网络配置与部署

### 外围网络的要求
为了实现这些特征，以下列表提供了有关需要满足哪些虚拟网络要求才能实现成功外围网络的指导：

- 子网体系结构：指定虚拟网络，使整个子网专门作为外围网络，与相同虚拟网络中的其他子网分开。如此可确保外围网络与其他内部或专用子网层之间的流量，一定流经具有“用户定义路由”的子网边界上的防火墙或 IDS/IPS 虚拟设备。
- 外围网络的网络安全组 (NSG) –外围网络子网本身应该打开以允许外围网络与 Internet 之间的通信，但这不表示客户应该绕过 NSG。请遵循常用的安全实践，管制允许访问部署的远程地址范围，和/或特定应用程序协议和打开的端口，将曝露于 Internet 的网络接触面减到最小。请注意，这些不一定都能做得到。例如，如果客户在 Azure 中有外部网站，则外围网络应该允许从任何公共 IP 地址传入的 Web 请求，但只应该打开 Web 应用程序端口：TCP:80 和 TCP:443。
- 外围网络的路由表 – 外围网络子网本身必须能够直接与 Internet 通信，但不应该允许未通过防火墙或安全设备，就直接与后端或本地网络之间相互通信。
- 外围网络安全设备配置 - 为了路由和检查外围网络与受保护网络其余部分之间的数据包，外围网络中的安全设备（例如防火墙、IDS、IPS 设备）可以有多重主目录，但外围网络和后端子网有独立的 NIC。外围网络中的 NIC 将使用相应的 NSG 和路由表，直接与 Internet 相互通信。对于相应的后端子网，连接到后端子网的 NIC 有更受限制的 NSG 和路由表。
- 外围网络安全设备功能 – 部署在外围网络中的安全设备通常执行以下功能：
    - 防火墙 – 对传入请求实施防火墙规则或访问控制策略
    - 威胁检测和预防 – 检测并缓解来自 Internet 的恶意攻击
    - 审核和记录 – 维护详细记录供审核和分析
    - 反向代理 - 将外围网络前端设备（通常是防火墙）上的目标地址映射并转换成实际的后端服务器地址，以将传入请求重定向到相应的后端服务器。
    - 正向代理 – 提供 NAT，同时审核从虚拟网络内部发起的与 Internet 的通信
    - 路由器 – 作为路由器来转发虚拟网络内的传入流量和跨子网流量
    - VPN 设备 – 充当跨界 VPN 网关，用于在本地网络上的客户与 Azure 虚拟网络之间进行跨界 VPN 连接
    - VPN 服务器 – 充当 VPN 服务器，接受连接到 Azure 虚拟网络的 VPN 客户端

>[AZURE.TIP]将有权访问外围网络安全设备的个人与获授权成为应用程序开发/部署/操作管理员的个人完全分开维护。将这些组隔开可区分权责，防止个人绕过应用程序安全与网络安全控制机制。

### 构建网络边界时的疑问
对于本部分中所述的“Azure 网络”，除非专门说明，否则所有网络（包括网络、虚拟网络或子网）都是指订阅管理员创建的专用 Azure 虚拟网络，而不是 Azure 中的基础物理网络。

此外，Azure 虚拟网络通常用于扩展传统的内部网络。站点到站点或 ExpressRoute 混合网络解决方案可与外围网络体系结构合并。在构建网络安全边界时，这是一个很重要的考虑因素，下面的问题解答将对此进行讨论。

构建包含外围网络和多个安全边界的网络时，必须解决三个重要的问题。

#### 1) 需要多少个边界？
第一个决策点是确定在特定情况下需要多少个安全边界：

- 单个边界 - 虚拟网络与 Internet 之间的前端外围网络上有一个边界。
- 两个边界 - 一个在外围网络网络的 Internet 端，另一个在 Azure 虚拟网络中的外围网络子网与后端子网之间
- 三个边界 - 在外围网络的 Internet 端、在外围网络与后端子网之间，以及在后端子网与本地网络之间
- N 个边界 – 根据安全要求而定，在特定网络中可应用的安全边界数量没有限制。

需要的边界类型和数量根据公司的风险承受度和实施的特定方案而有所不同。这通常是组织内的多个团队达成的共同决策，通常包括风险和合规性小组、网络/平台小组和应用程序开发小组。了解安全、所涉及的数据及运用技术的人员，在此决策上应该有发言权，以确保对每种实施方案表达适当的安全立场。

>[AZURE.TIP]应该尽可能以最少的边界来满足特定情况下的安全要求。边界越多，操作和故障排除就越困难，后续管理多个边界策略时的管理开销也越大。不过，边界不足也会增大风险。获得平衡至关重要。

![包含三个安全边界的混合网络][6]

上图从较高层面显示了网络的三个安全边界：一个在外围网络与 Internet 之间，一个在 Azure 前端与后端专用子网之间，还有一个在 Azure 后端子网与本地企业网络之间。

#### 2) 边界位于何处？
确定边界数量后，下一个决策点就是在何处实施它们。通常有三种选择：1) 使用基于 Internet 的中介服务（例如基于云的 WAF，本文中未讨论），2) 使用 Azure 的本机功能和/或网络虚拟设备，3) 使用本地网络上的物理设备。

在纯粹的 Azure 网络上，选项包括本机 Azure 功能（例如 Azure 负载平衡器），或由 Azure 丰富的合作伙伴生态系统提供的网络虚拟设备（例如检查点防火墙）。

如果 Azure 与本地网络之间需要边界，安全设备可以位于连接的任何一端（或两端）。因此，必须确定安全设备的位置。

上图中，Internet 到外围网络及前端到后端的边界完全在 Azure 内，而且必须是本机 Azure 功能或网络虚拟设备。在 Azure（后端子网）与企业网络之间的边界上，安全设备可以在 Azure 端或本地端，甚至是两端的设备组合。任一选项都可能有明显的优点和缺点，必须慎重考虑。

例如，使用本地网络端现有的物理安全设备。优点：不需要新的设备，只需要新的配置。缺点：所有流量必须从 Azure 传回到本地网络才能被安全设备看到。因此，如果将 Azure 到 Azure 流量强制传回本地网络来实施安全策略，则可能会造成明显的延迟，影响应用程序性能和用户体验。

#### 3) 如何实施边界？
每个安全边界可能有不同的功能要求（例如，外围网络的 Internet 端需要 IDS 和防火墙规则，但外围网络与后端子网之间只需要 ACL）。决定使用哪些设备将取决于方案和安全要求。以下外围网络示例 1、2 和 3 讨论了一些可用的选项。了解 Azure 本机网络功能和 Azure 中由合作伙伴生态系统提供的设备可以找到大量的选项来满足几乎任何方案。

另一个重要的实施决策点就是如何连接本地网络与 Azure。通过 Azure 虚拟网关或网络虚拟设备。以下示例 4、5 和 6 详细描述和讨论了这些选项。

此外，Azure 内可能需要 VNet 到 VNet 流量，以下示例 7 和 8 详细讨论了这些方案。

解答上述问题后，可以参考前面的[快速开始](#fast-start)部分确定最适合特定方案的示例。

## 构建 Azure 虚拟网络的安全边界
### 示例 1 – 使用 NSG 构建简单的外围网络
[返回“快速开始”](#fast-start) <!-- | [此示例的详细构建说明][Example1] -->

![使用 NSG 的入站外围网络][7]

#### 环境描述
此示例中，有一个订阅包含以下项：

- 两个云服务：“FrontEnd001”和“BackEnd001”
- 一个虚拟网络“CorpNetwork”，其中包含两个子网：“FrontEnd”和“BackEnd”
- 应用到这两个子网的网络安全组
- 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
- 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
- 一个代表 DNS 服务器的 Windows Server（“DNS01”）

<!--[详细构建说明][Example1]页提供了构建此示例的更多详细信息（提供了脚本和 ARM 模板）。-->

#### 网络安全组 (NSG) 描述
此示例将构建一个 NSG 组，然后加载六个规则。

>[AZURE.TIP]一般而言，应该先创建特定的“允许”规则，最后再创建一般的“拒绝”规则。给定的优先级确定了要先评估哪些规则。发现要向流量应用哪个特定规则后，不需要评估后续规则。可以朝入站或出站方向（从子网的角度看）应用 NSG 规则。

以声明性的方式为入站流量构建以下规则：

1.	允许内部 DNS 流量（端口 53）
2.	允许从 Internet 到任何 VM 的 RDP 流量（端口 3389）
3.	允许从 Internet 到 Web 服务器 (IIS01) 的 HTTP 流量（端口 80）
4.	允许从 IIS01 到 AppVM1 的任何流量（所有端口）
5.	拒绝从 Internet 到整个 VNet（两个子网）的任何流量（所有端口）
6.	拒绝从前端子网到后端子网的任何流量（所有端口）

将这些规则绑定到每个子网后，如果有从 Internet 到 Web 服务器的入站 HTTP 请求，将应用规则 3（允许）和规则 5（拒绝），但由于规则 3 具有较高的优先级，因此只应用规则 3 并忽略规则 5。这样就会允许 HTTP 请求传往 Web 服务器。如果相同的流量尝试传往 DNS01 服务器，则会先应用规则 5（拒绝），因此不允许该流量传递到服务器。规则 6（拒绝）阻止前端子网与后端子网对话（规则 1 和 4 允许的流量除外），这可在攻击者入侵前端上的 Web 应用程序时保护后端网络，攻击者只能对后端的“受保护”网络进行有限度的访问（只能访问 AppVM01 服务器上公开的资源）。

有一个默认出站规则可允许流量外流到 Internet。在此示例中，我们允许出站流量，且未修改任何出站规则。如果两个方向的流量都要锁定，则需要用户定义的路由，下面的“示例 3”将对此进行介绍。

#### 结束语
这种隔离后端子网与输入流量的方式相当直截了当。有关以下操作的示例详细信息：

- 如何使用 PowerShell 脚本构建此外围网络
- 如何使用 ARM 模板构建此外围网络
- 每个 NSG 命令的详细描述
- 展示如何在每一层允许或拒绝流量的详细流量传送方案

<!-- 请参阅[详细构建说明][Example1]页。-->

### 示例 2 – 构建外围网络以通过防火墙和 NSG 保护应用程序
[返回“快速开始”](#fast-start) <!-- | [此示例的详细构建说明][Example2]-->

![使用 NVA 和 NSG 的入站外围网络][8]

#### 环境描述
此示例中，有一个订阅包含以下项：

- 两个云服务：“FrontEnd001”和“BackEnd001”
- 一个虚拟网络“CorpNetwork”，其中包含两个子网：“FrontEnd”和“BackEnd”
- 应用到这两个子网的单个网络安全组
- 一个与前端子网连接的网络虚拟设备（在本示例中为防火墙）
- 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
- 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
- 一个代表 DNS 服务器的 Windows Server（“DNS01”）

<!-- [详细构建说明][Example2]页提供了构建此示例的更多详细信息（提供了脚本和 ARM 模板）。-->

#### 网络安全组 (NSG) 描述
此示例将构建一个 NSG 组，然后加载六个规则。

>[AZURE.TIP]一般而言，应该先创建特定的“允许”规则，最后再创建一般的“拒绝”规则。给定的优先级确定了要先评估哪些规则。发现要向流量应用哪个特定规则后，不需要评估后续规则。可以朝入站或出站方向（从子网的角度看）应用 NSG 规则。

以声明性的方式为入站流量构建以下规则：

1.	允许内部 DNS 流量（端口 53）
2.	允许从 Internet 到任何 VM 的 RDP 流量（端口 3389）
3.	允许从 Internet 到 NVA（防火墙）的任何流量（所有端口）
4.	允许从 IIS01 到 AppVM1 的任何流量（所有端口）
5.	拒绝从 Internet 到整个 VNet（两个子网）的任何流量（所有端口）
6.	拒绝从前端子网到后端子网的任何流量（所有端口）

将这些规则绑定到每个子网后，如果有从 Internet 到防火墙的入站 HTTP 请求，将应用规则 3（允许）和规则 5（拒绝），但由于规则 3 具有较高的优先级，因此只应用规则 3 并忽略规则 5。这样就会允许 HTTP 请求传往防火墙。如果相同的流量尝试传往 IIS01 服务器，即使是在前端子网上，也会应用规则 5（拒绝），因此不允许该流量传递到服务器。规则 6（拒绝）阻止前端子网与后端子网对话（规则 1 和 4 允许的流量除外），这可在攻击者入侵前端上的 Web 应用程序时保护后端网络，攻击者只能对后端的“受保护”网络进行有限度的访问（只能访问 AppVM01 服务器上公开的资源）。

有一个默认出站规则可允许流量外流到 Internet。在此示例中，我们允许出站流量，且未修改任何出站规则。如果两个方向的流量都要锁定，则需要用户定义的路由，下面的“示例 3”将对此进行介绍。

#### 防火墙规则描述
需要在防火墙上创建转发规则。本示例只将 Internet 流量入站路由到防火墙，再传送到 Web 服务器，因此只需要一个转发 NAT 规则。

转发规则接受任何到达防火墙并尝试连接到 HTTP（端口 80，若为 HTTPS 则为 443）的入站源地址，此地址将从防火墙的本地接口发出，并重定向到 IP 地址为 10.0.1.5 的 Web 服务器。

#### 结束语
这种使用防火墙保护应用程序并隔离后端子网与输入流量的方式相当直截了当。有关以下操作的示例详细信息：

- 如何使用 PowerShell 脚本构建此示例外围网络
- 如何使用 ARM 模板构建此示例
- 每个 NSG 命令和防火墙规则的详细描述
- 展示如何在每一层允许或拒绝流量的详细流量传送方案

<!-- 请参阅[详细构建说明][Example2]页。-->

### 示例 3 – 构建外围网络以通过防火墙、UDR 和 NSG 保护网络
[返回“快速开始”](#fast-start) | [此示例的详细构建说明][Example3]

![使用 NVA、NSG 和 UDR 的双向外围网络][9]

#### 环境设置
此示例中，有一个订阅包含以下项：

- 三个云服务：“SecSvc001”、“FrontEnd001”和“BackEnd001”
- 一个虚拟网络“CorpNetwork”，其中包含三个子网：“SecNet”、“FrontEnd”和“BackEnd”
- 一个与 SecNet 子网连接的网络虚拟设备（在本示例中为防火墙）
- 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
- 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
- 一个代表 DNS 服务器的 Windows Server（“DNS01”）

[详细构建说明][Example3]页提供了构建此示例的更多详细信息（提供了脚本和 ARM 模板）。

#### 用户定义的路由 (UDR) 描述
默认情况下，以下系统路由定义为：

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.0.0/16}     VNETLocal                            Active   Default    
         {0.0.0.0/0}       Internet                             Active   Default    
         {10.0.0.0/8}      Null                                 Active   Default    
         {100.64.0.0/10}   Null                                 Active   Default    
         {172.16.0.0/12}   Null                                 Active   Default    
         {192.168.0.0/16}  Null                                 Active   Default

VNETLocal 始终是该特定网络的 VNet 的已定义地址前缀（也就是说，根据每个特定 VNet 的定义方式，不同的 VNet 有不同前缀）。其余系统路由是静态的，上面列出了其默认值。

此示例中创建了两个路由表，分别用于前端和后端子网。已在每个表中加载了适用于给定子网的静态路由。基于此示例的目的，每个表包含三个路由，它们负责通过防火墙定向所有流量 (0.0.0.0/0)（下一跃点 = 虚拟设备 IP 地址）：

1. 定义了不带下一跃点的本地子网流量以允许本地子网流量绕过防火墙
2. 下一跃点定义为防火墙的虚拟网络流量，这会覆盖允许本地 VNet 流量直接路由的默认规则
3. 将带有下一跃点的所有剩余流量 (0/0) 定义为防火墙

路由表在创建后会绑定到其子网。前端子网路由表在创建并绑定到子网后应如下所示：

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.1.0/24}     VNETLocal                            Active 
		 {10.0.0.0/16}     VirtualAppliance 10.0.0.4            Active    
         {0.0.0.0/0}       VirtualAppliance 10.0.0.4            Active

>[AZURE.NOTE]UDR 和混合网络目前有所限制。将来的版本中会解决这些限制。有关如何使用 ExpressRoute 或站点到站点网络启用外围网络的示例将在下面的示例 3 和 4 中讨论。

#### IP 转发描述
UDR 随附 IP 转发功能。这是虚拟设备上的一项设置，使虚拟设备能够接收不是要专门传送到该设备的流量，再将流量转发到其最终目标。

例如，如果来自 AppVM01 的流量对 DNS01 服务器发出请求，UDR 会将此流量路由到防火墙。在启用 IP 转发后，目标为 DNS01 (10.0.2.4) 的流量被设备 (10.0.0.4) 所接受，然后转发到其最终目标 (10.0.2.4)。如果防火墙上未启用 IP 转发，则即使路由表将防火墙用作下一跃点，流量也不会被设备所接受。若要使用虚拟设备，必须记得一同启用 IP 转发和用户定义的路由。

#### 网络安全组 (NSG) 描述
此示例将构建一个 NSG 组，然后加载单个规则。此组接着只绑定到前端和后端子网（不绑定到 SecNet）。以声明性的方式构建以下规则：

1.	拒绝从 Internet 到整个 VNet（所有子网）的任何流量（所有端口）

尽管本示例使用了 NSG，但它的主要用途是作为防止人为配置错误的第二道防线。我们想要阻止由 Internet 流往前端或后端子网的所有入站流量，流量只应流经 SecNet 子网前往防火墙（并于合适时流往前端或后端子网）。此外，在配置 UDR 规则后，确实能够流往前端或后端子网的流量都将被定向到防火墙（得益于 UDR）。防火墙将此流量视为非对称流量，并且会丢弃出站流量。因此，前端和后端子网共有三个安全防护层：1) FrontEnd001 和 BackEnd001 云服务上没有开放的终结点，2) NSG 拒绝来自 Internet 的流量，3) 防火墙丢弃非对称流量。

本示例中的网络安全组有一个有趣的特点，那就是它只包含一个规则，用于拒绝由 Internet 流往整个虚拟网络（包含安全子网）的流量。不过，由于 NSG 只绑定到前端和后端子网，因此不对流往安全子网的入站流量处理此规则。这样，即使 NSG 规则指出因为 NSG 永远不绑定到安全子网，所以没有由 Internet 流往 VNet 上任何地址的流量，流量也还是会流向安全子网。

#### 防火墙规则
需要在防火墙上创建转发规则。由于防火墙会阻止或转发所有入站、出站和 VNet 内部流量，因此需要配置许多防火墙规则。此外，所有入站流量都抵达安全服务的公共 IP 地址（在不同端口上），并由防火墙进行处理。最佳实践是先绘制逻辑流量图，再设置子网和防火墙规则，以避免事后进行修改。下图是此示例中防火墙规则的逻辑视图：
 
![防火墙规则的逻辑视图][10]

>[AZURE.NOTE]根据所用的网络虚拟设备，管理端口会有所不同。本示例中提到的 Barracuda NG 防火墙使用端口 22、801 和 807。请参阅设备供应商的文档来查找用于管理所用设备的确切端口。

#### 防火墙规则描述
安全子网上只有防火墙这个资源，因此上述逻辑视图未显示该子网，而且这个视图显示防火墙规则以及这些规则在逻辑上是如何允许或拒绝流量的流动，而不显示实际的路由路径。此外，针对 RDP 流量选择的外部端口均为较高范围的端口 (8014 – 8026)，并且选择时在一定程度上遵循了本地 IP 地址的最后两个八位字节以方便阅读（例如，本地服务器地址 10.0.1.4 与外部端口 8014 相关联），不过可以使用任何更高范围的端口，只要端口不冲突即可。

在此示例中，我们需要 7 种类型的规则，下面描述了这些规则类型：

- 外部规则（针对入站流量）：
  1.	防火墙管理规则：此应用重定向规则允许流量传递到网络虚拟设备的管理端口。
  2.	RDP 规则（针对每个 Windows 服务器）：这四个规则（每台服务器一个）允许通过 RDP 管理单个服务器。根据所用的网络虚拟设备功能，也可将其捆绑为一个规则。
  3.	应用程序流量规则：应用程序流量规则有两个，第一个针对前端 Web 流量，第二个针对后端流量（例如 Web 服务器流往数据层）。这些规则的配置取决于网络体系结构（服务器的放置位置）和流量流动行为（流量的流动方向，以及使用的端口）。
      - 第一个规则允许实际的应用程序流量抵达应用程序服务器。其他规则所考虑的是安全、管理等方面的事项，应用程序规则用于允许外部用户或服务访问应用程序。就本示例而言，端口 80 上有单个 Web 服务器，因此单个防火墙应用程序规则将流往外部 IP 的入站流量，并重定向到 Web 服务器的内部 IP 地址。重定向的流量会话经过 NAT 后流往内部服务器。
      - 第二个应用程序流量规则是后端规则，用于允许 Web 服务器通过任何端口与 AppVM01 服务器（而非 AppVM02）对话。
- 内部规则（针对 VNet 内部流量）
  4.	出站到 Internet 规则：此规则允许来自任何网络的流量传递到选定的网络。此规则通常是防火墙上已有的但处于禁用状态的默认规则。对于本示例，应启用此规则。
  5.	DNS 规则：此规则只允许 DNS（端口 53）流量传递到 DNS 服务器。在此环境中，阻止了大部分由前端流往后端的流量，而此规则专门允许来自任何本地子网的 DNS。
  6.	子网到子网规则：此规则允许后端子网上的任何服务器连接到前端子网上的任何服务器（但不允许反向连接）。
- 防故障规则（针对不符合上述任一规则的流量）：
  7.	拒绝所有流量规则：请始终将此规则用作最终规则（在优先级方面），这样，如果流量的流动行为无法符合上述任何规则，此规则就会将其丢弃。这是默认规则且通常已激活，一般而言并不需要修改。

>[AZURE.TIP]为简化本示例，允许第二个应用程序流量规则上的任何端口，但在真实方案中，则应使用具体的端口和地址范围，以降低此规则的攻击面。

创建好上述所有规则后，请务必检查每个规则的优先级，以确保可根据需要允许或拒绝流量。在本示例中，规则设置了优先顺序。

#### 结束语
这是以较复杂但更完整的方式来保护和隔离网络的方法（示例 2 只保护应用程序，示例 1 只隔离子网）。这项设计允许你同时监视两个方向的流量，而不只是保护入站应用程序服务器，同时还对此网络上的所有服务器实施网络安全策略。此外，根据使用的设备，还能实现全面的流量审核和感知。有关以下操作的示例详细信息：

- 如何使用 PowerShell 脚本构建此示例外围网络
- 如何使用 ARM 模板构建此示例
- 每个 UDR、NSG 命令和防火墙规则的详细描述
- 展示如何在每一层允许或拒绝流量的详细流量传送方案

请参阅[详细构建说明][Example3]页。

### 示例 4 – 使用站点到站点虚拟设备 VPN 添加混合连接
[返回“快速开始”](#fast-start) | 详细构建说明即将发布

![包含连接 NVA 的外围网络的混合网络][11]

#### 环境设置
可将使用网络虚拟设备 (NVA) 的混合网络添加到示例 1、2 或 3 中所述的任何外围网络类型。

如上图所示，通过 Internet 的 VPN 连接（站点到站点）用于通过网络虚拟设备将本地网络连接到 Azure 虚拟网络。

>[AZURE.NOTE]如果使用 ExpressRoute 并启用“Azure 公共对等”选项，则需要创建静态路由才能路由到企业 Internet 边缘外部的 NVA VPN IP 地址（无需通过 ExpressRoute WAN 边缘）。这是因为 ExpressRoute“Azure 公共对等”选项所需的 NAT 很可能会中断 VPN 会话（IPSec 通常不支持 NAT）。

配置好 VPN 之后，NVA 将成为所有网络和子网的“中心”。防火墙转发规则确定允许、NAT、重定向或丢弃哪些流量（即使是对于本地网络与 Azure 之间按此方式传送的流量）。

应该谨慎考虑流量传送，因为根据具体的用例，这种设计模式可能会优化或退化传送的性能。

使用示例 3“构建外围网络以通过防火墙、UDR 和 NSG 保护网络”中构建的环境，添加站点到站点 VPN 混合网络连接后会生成以下设计：

![使用站点到站点 VPN 连接 NVA 的外围网络][12]

在上面的设计中，本地路由器或与 VPN 网络虚拟设备兼容的任何其他网络设备是 VPN 客户端。此物理设备负责发起和保持与网络虚拟设备的 VPN 连接。

就 NVA 而言，网络在逻辑上就像四个不同的“安全区域”，NVA 上的规则统管这些区域之间的流量。逻辑示意图如下：

![从 NVA 角度观察的逻辑网络][13]

#### 结束语
将站点到站点 VPN 混合网络连接添加到 Azure 虚拟网络能够安全地将本地网络扩展到 Azure。使用 VPN 连接时，流量将会加密并通过 Internet 路由。如本示例中所示，使用 NVA 可以集中实施和管理安全策略。有关以下操作的示例详细信息：

- 如何使用 PowerShell 脚本构建此示例外围网络
- 如何使用 ARM 模板构建此示例
- 展示此设计中流量如何传送的详细流量方案

即将发布，并在此页上提供链接。

### 示例 5 – 使用站点到站点 Azure 网关 VPN 添加混合连接
[返回“快速开始”](#fast-start) | 详细构建说明即将发布

![包含连接网关的外围网络的混合网络][14]

#### 环境设置
可将使用 Azure VPN 网关的混合网络添加到示例 1 和 2 中所述的任何一种外围网络。

如上图所示，通过 Internet 的 VPN 连接（站点到站点）用于通过 Azure VPN 网关将本地网络连接到 Azure 虚拟网络。

>[AZURE.NOTE]如果使用 ExpressRoute 并启用“Azure 公共对等”选项，则需要创建静态路由才能路由到企业 Internet 边缘外部的 Azure VPN 网关 IP 地址（无需通过 ExpressRoute WAN 边缘）。这是因为 ExpressRoute“Azure 公共对等”选项所需的 NAT 很可能会中断 VPN 会话（IPSec 通常不支持 NAT）。

如下所示，使用此选项时，环境现在有两个网络边缘。在第一个边缘上，NVA 和 NSG 控制 Azure 网络内部及 Azure 与 Internet 之间的流量，第二个边缘是 Azure VPN 网关，它是本地与 Azure 之间的一个完全独立且隔离的网络边缘。

应该谨慎考虑流量传送，因为根据具体的用例，这种设计模式可能会优化或退化传送的性能。

使用示例 1“构建外围网络以通过 NSG 保护应用程序”中构建的环境，添加站点到站点 VPN 混合网络连接后会生成以下设计：

![使用 ExpressRoute 连接网关的外围网络][15]

#### 结束语
将站点到站点 VPN 混合网络连接添加到 Azure 虚拟网络能够安全地将本地网络扩展到 Azure。使用本机 Azure VPN 网关时，流量将由 IPSec 加密并通过 Internet 路由。此外，使用 Azure VPN 网关的成本也较低（不像第三方网络虚拟设备一样需要额外的许可成本）。在不使用任何网络虚拟设备的示例 1 中，这是最经济实惠的方案。有关以下操作的示例详细信息：

- 如何使用 PowerShell 脚本构建此示例外围网络
- 如何使用 ARM 模板构建此示例
- 展示此设计中流量如何传送的详细流量方案

即将发布，并在此页上提供链接。

### 示例 6 – 使用 ExpressRoute 添加混合连接
[返回“快速开始”](#fast-start) | 详细构建说明即将发布

![包含连接网关的外围网络的混合网络][16]

#### 环境设置
可将使用 ExpressRoute 专用对等连接的混合网络添加到示例 1 或 2 中所述的任一种外围网络。

如上图所示，ExpressRoute 专用对等在本地网络与 Azure 虚拟网络之间提供直接连接。流量只在服务提供商网络和 Microsoft/Azure 网络上流动，永远不会传到 Internet。

>[AZURE.NOTE]由于 Azure 虚拟网关上使用的动态路由相当复杂，因此用户定义的路由 (UDR) 和 ExpressRoute 在使用上存在限制。如果子网与提供 ExpressRoute 连接的 Azure 网关通信，则不应该应用 UDR。此外，Azure 网关不能是用于其他 UDR 绑定子网的 NextHop 设备。将来的 Azure 版本将启用完全集成 UDR 和 ExpressRoute 的功能。

</br>
>[AZURE.TIP]使用 ExpressRoute 可使企业网络流量脱离 Internet，以提高安全性，并大幅改进性能，同时符合 ExpressRoute 提供商的 SLA。在 ExpressRoute 性能方面，Azure 网关在 ExpressRoute 上的传递速度高达 2Gbps，而使用站点到站点 VPN 时，Azure 网关的最大吞吐量为 200Mbps。

如下图所示，由于使用了此选项，环境现在有两个网络边缘，NVA 和 NSG 控制 Azure 网络内部及 Azure 与 Internet 之间的流量，网关是本地与 Azure 之间的一个完全独立且隔离的网络边缘。

应该谨慎考虑流量传送，因为根据具体的用例，这种设计模式可能会优化或退化传送的性能。

使用示例 1“使用 NSG 构建简单的外围网络”中构建的环境，添加 ExpressRoute 混合网络连接后会生成以下设计：

![使用 ExpressRoute 连接网关的外围网络][17]

#### 结束语
添加 ExpressRoute 专用对等网络连接能够以安全、低延迟、高性能的方式将本地网络扩展到 Azure。此外，如本示例中所示使用本机 Azure 网关还能降低成本（不像第三方网络虚拟设备一样需要额外的许可）。有关以下操作的示例详细信息：

- 如何使用 PowerShell 脚本构建此示例外围网络
- 如何使用 ARM 模板构建此示例
- 展示此设计中流量如何传送的详细流量方案

即将发布，并在此页上提供链接。

## 参考
### 有用的网站和文档
- 使用 ARM 访问 Azure： 
- 使用 PowerShell 访问 Azure：[/documentation/articles/powershell-install-configure/](/documentation/articles/powershell-install-configure)
- 虚拟网络文档：[https://azure.microsoft.com/documentation/services/virtual-network/](/services/virtual-network/)
- 虚拟网络组文档：[/documentation/articles/virtual-networks-nsg/](/documentation/articles/virtual-networks-nsg)
- 用户定义的路由文档：[/documentation/articles/virtual-networks-udr-overview/](/documentation/articles/virtual-networks-udr-overview)
- 站点到站点 VPN：[/documentation/articles/vpn-gateway-site-to-site-create/](/documentation/articles/vpn-gateway-site-to-site-create)
- ExpressRoute 文档（请务必阅读“入门”和“操作指南”部分）：[https://azure.microsoft.com/documentation/services/expressroute/](/services/expressroute/)

<!--Image References-->
[0]: ./media/best-practices-network-security/flowchart.png "安全选项流程图"
[1]: ./media/best-practices-network-security/compliancebadges.png "Azure 合规性徽章"
[2]: ./media/best-practices-network-security/azuresecurityfeatures.png "Azure 安全功能"
[3]: ./media/best-practices-network-security/dmzcorporate.png "企业网络中的外围网络"
[4]: ./media/best-practices-network-security/azuresecurityarchitecture.png "Azure 安全体系结构"
[5]: ./media/best-practices-network-security/dmzazure.png "Azure 虚拟网络中的外围网络"
[6]: ./media/best-practices-network-security/dmzhybrid.png "包含三个安全边界的混合网络"
[7]: ./media/best-practices-network-security/example1design.png "使用 NSG 的入站外围网络"
[8]: ./media/best-practices-network-security/example2design.png "使用 NVA 和 NSG 的入站外围网络"
[9]: ./media/best-practices-network-security/example3design.png "使用 NVA、NSG 和 UDR 的双向外围网络"
[10]: ./media/best-practices-network-security/example3firewalllogical.png "防火墙规则的逻辑视图"
[11]: ./media/best-practices-network-security/example4designoptions.png "包含连接 NVA 的外围网络的混合网络"
[12]: ./media/best-practices-network-security/example4designs2s.png "使用站点到站点 VPN 连接 NVA 的外围网络"
[13]: ./media/best-practices-network-security/example4networklogical.png "从 NVA 角度观察的逻辑网络"
[14]: ./media/best-practices-network-security/example5designoptions.png "包含使用站点到站点连接来连接 Azure 网关的外围网络的混合网络"
[15]: ./media/best-practices-network-security/example5designs2s.png "包含使用站点到站点 VPN 的 Azure 网关的外围网络"
[16]: ./media/best-practices-network-security/example6designoptions.png "包含使用 ExpressRoute 连接 Azure 网关的外围网络的混合网络"
[17]: ./media/best-practices-network-security/example6designexpressroute.png "包含使用 ExpressRoute 连接的 Azure 网关的外围网络"

<!--Link References-->
[Example1]: /documentation/articles/virtual-networks-dmz-nsg-asm
[Example2]: /documentation/articles/virtual-networks-dmz-nsg-fw-asm
[Example3]: /documentation/articles/virtual-networks-dmz-nsg-fw-udr-asm
[Example4]: /documentation/articles/virtual-networks-hybrid-s2s-nva-asm
[Example5]: /documentation/articles/virtual-networks-hybrid-s2s-agw-asm
[Example6]: /documentation/articles/virtual-networks-hybrid-expressroute-asm
[Example7]: /documentation/articles/virtual-networks-vnet2vnet-direct-asm
[Example8]: /documentation/articles/virtual-networks-vnet2vnet-transit-asm

<!---HONumber=79-->