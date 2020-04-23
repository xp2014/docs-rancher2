---
title: 配置参考
description: 本部分旨在在 Rancher 中设置 OpenLDAP 身份验证时用作参考。在开始之前，请阅读并了解外部身份验证配置和用户主体。

keywords:
  - rancher 2.0中文文档
  - rancher 2.x 中文文档
  - rancher中文
  - rancher 2.0中文
  - rancher2
  - rancher教程
  - rancher中国
  - rancher 2.0
  - rancher2.0 中文教程
  - 系统管理员指南
  - 登录认证
  - 对接 OpenLDAP
  - 配置参考
---

本部分旨在在 Rancher 中设置 OpenLDAP 身份验证时用作参考。

有关配置 OpenLDAP 的更多详细信息，请参阅[官方文档](https://www.openldap.org/doc/)。

> 在开始之前，请阅读并了解[外部身份验证配置和用户主体](/docs/admin-settings/authentication/_index)。

## 背景：OpenLDAP 认证流程

1. 当用户尝试使用其 LDAP 凭据登录时，Rancher 会使用具有搜索目录和读取用户/组属性权限的服务帐户，创建与 LDAP 服务器的初始绑定。
2. 然后，Rancher 使用搜索过滤器根据提供的用户名和配置的属性映射为用户搜索目录。
3. 找到用户后，将使用用户的 DN 和提供的密码通过另一个 LDAP 绑定请求对他进行身份验证。
4. 身份验证成功后，Rancher 将从用户对象的成员资格属性以及通过基于配置的用户映射属性执行组搜索来解析组成员资格。

## 配置 OpenLDAP 服务器设置

您将需要输入地址，端口和协议以连接到 OpenLDAP 服务器。不安全流量的标准端口为`389`，TLS 流量的标准端口为`636`。

> **使用 TLS?**
>
> 如果 OpenLDAP 服务器使用的证书是自签名的，或者不是来自公认的证书颁发机构的，则请确保手头有 PEM 格式的 CA 证书（包括任何中间证书）。在配置过程中，您将必须粘贴此证书，以便 Rancher 能够验证证书链。

如果您不确定在用户/组 **Search Base** 配置字段中输入的正确值，请咨询 LDAP 管理员，或参阅 Active Directory 身份验证文档中的[使用 ldapsearch 识别 Search Base 和架构部分](/docs/admin-settings/authentication/ad/#annex-identify-search-base-and-schema-using-ldapsearch)。

**OpenLDAP 服务器参数**

| 参数             | 描述                                                                                                                                         |
| :--------------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| 主机名           | 指定 OpenLDAP 服务器的主机名或 IP 地址。                                                                                                     |
| 端口             | 指定 OpenLDAP 服务器侦听连接的端口。未加密的 LDAP 通常使用标准端口 389，而 LDAPS 使用端口 636。                                              |
| TLS              | 选中此框以启用基于 SSL / TLS 的 LDAP（通常称为 LDAPS）。如果服务器使用自签名/企业签名的证书，则还需要粘贴 CA 证书。                          |
| 服务器连接超时   | Rancher 在考虑服务器不可达之前等待的持续时间（以秒为单位）。                                                                                 |
| 服务帐户专有名称 | 输入应用于绑定，搜索和检索 LDAP 条目的用户的专有名称（DN）。 (查看[参考](#先决条件))。                                                       |
| 服务帐号密码     | 服务帐户的密码。                                                                                                                             |
| 用户 Search Base | 输入目录树中节点的专有名称，从该节点开始搜索用户对象。所有用户都必须是此基本 DN 的后代。例如：`ou=people,dc=acme,dc=com`。                   |
| 组 Search Base   | 如果您的组所在的节点与在其下配置的`User Search Base`节点不同，则需要在此处提供专有名称。否则将此字段留空。例如：`ou=groups,dc=acme,dc=com`。 |

## 配置用户/组架构

如果您的 OpenLDAP 目录不同于标准的 OpenLDAP 架构，则必须完成**自定义架构**部分以使其匹配。请注意，Rancher 使用本节中配置的属性映射来构造搜索过滤器并解析组成员身份。因此，始终建议您验证此处的配置是否与您的 OpenLDAP 中使用的架构匹配。

如果您不熟悉 OpenLDAP 服务器中使用的用户/组架构，请咨询 LDAP 管理员，或参阅 Active Directory 身份验证文档中的[使用 ldapsearch 识别 Search Base 和架构部分](/docs/admin-settings/authentication/ad/#annex-identify-search-base-and-schema-using-ldapsearch)。

### 用户架构配置

下表详细介绍了用户架构配置的参数。

**用户架构配置参数**

| 参数           | 描述                                                                                                                                                                   |
| :------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 对象类         | 域中用于用户对象的对象类的名称。如果定义，则仅指定对象类的名称 - 请勿将其放在 LDAP 包装器中，例如`&(objectClass=xxxx)`                                                 |
| 用户名属性     | 用户属性，其值适合作为显示名称。                                                                                                                                       |
| 登录属性       | 该属性的值与用户登录 Rancher 时输入的凭据的用户名部分匹配。通常是这样`uid`。                                                                                           |
| 用户成员属性   | 包含用户所属组的专有名称的用户属性。通常这是`memberOf`或之一`isMemberOf`。                                                                                             |
| 搜索属性       | 当用户输入文本以在 UI 中添加用户或组时，Rancher 会查询 LDAP 服务器并尝试通过此设置中提供的属性来匹配用户。可以通过使用竖线(`|`)分隔多个属性来指定多个属性。            |
| 用户启用的属性 | 如果您的 OpenLDAP 服务器的架构支持用户属性，可以对其值进行评估以确定该帐户是禁用还是锁定，请输入该属性的名称。默认的 OpenLDAP 模式不支持此功能，并且该字段通常应留空。 |
| 禁用状态位掩码 | 这是禁用/锁定的用户帐户的值。如果`用户启用的属性`为空，则忽略该参数。                                                                                                  |

### 组架构配置

下表详细介绍了组架构配置的参数。

**组架构配置参数**

| 参数           | 描述                                                                                                                          |
| :------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| 对象类别       | 域中用于组对象的对象类的名称。如果定义，则仅指定对象类的名称 - 请勿将其放在 LDAP 包装器中，例如`&(objectClass=xxxx)`          |
| 名称属性       | 其值适合于显示名称的组属性。                                                                                                  |
| 组成员用户属性 | **用户属性的名称**，其格式与中的组成员匹配`组成员映射属性`。                                                                  |
| 组成员映射属性 | 包含组成员的组属性的名称。                                                                                                    |
| 搜索属性       | 在向 UI 中的集群或项目添加组时用于构造搜索过滤器的属性。请参阅用户架构说明`Search Attribute`。                                |
| 组 DN 属性     | 组属性的名称，其格式与用户的组成员资格属性中的值匹配。请参阅`User Member Attribute`。                                         |
| 嵌套组成员     | 此设置定义 Rancher 是否应解析嵌套的组成员身份。仅当您的组织使用这些嵌套成员身份时才使用（即，您具有包含其他组作为成员的组）。 |