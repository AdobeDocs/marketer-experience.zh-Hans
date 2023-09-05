---
title: 单一事件
description: 这是用于模拟“[!UICONTROL 单一事件]”类型的历程验证的说明页面。
source-git-commit: 0a52611530513fc036ef56999fde36a32ca0f482
workflow-type: ht
source-wordcount: '749'
ht-degree: 100%

---


# 单一事件

## 遵循的步骤 {#steps-to-follow}

>[!CONTEXTUALHELP]
>id="marketerexp_sampledata_unitaryevent"
>title="使用方法"
>abstract="请访问该链接以了解更多详情"

>[!IMPORTANT]
>
>这些说明可能会在&#x200B;**[!UICONTROL 战术手册]**&#x200B;中发生变化，因此，请务必参考相应的&#x200B;**[!UICONTROL 战术手册]**&#x200B;的“示例数据”部分。

## 先决条件

* 您必须已安装 Postman 软件
* 使用战术手册创建实例资产，例如&#x200B;**[!UICONTROL 历程]**、**[!UICONTROL 架构]**、**[!UICONTROL 区段]**、**[!UICONTROL 消息]**&#x200B;等。

创建的资产将显示在`Bill Of Material`页面上

![物料清单页面](../assets/bom-page.png)

## 为 Postman 准备必要的集合

1. 访问&#x200B;**[!UICONTROL 用例战术手册]**&#x200B;应用程序。
1. 单击相应的&#x200B;**[!UICONTROL 战术手册]**&#x200B;信息卡以访问&#x200B;**[!UICONTROL 战术手册]**&#x200B;详细信息页面。
1. 访问&#x200B;**[!UICONTROL 物料清单]**&#x200B;页面，并查找&#x200B;**[!UICONTROL 示例数据]**&#x200B;部分。
1. 单击 UI 上的相应按钮来下载 `postman.json`。
1. 在 **[!DNL Postman Software]** 中导入 `postman.json`。
1. 为此验证创建专用的 Postman 环境（例如 `Adobe <PLAYBOOK_NAME>`）。

## 提取 IMS 令牌

>[!NOTE]
>
>所有环境变量都区分大小写，因此，请务必使用准确的变量名称。

1. 请按照[验证和访问 Experience Platform API](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) 文档中的说明操作以生成访问令牌。
1. 将访问令牌值存储在名为 `ACCESS_TOKEN` 的环境变量中。
1. 将其他与身份验证相关的值（例如 `API_KEY`、`IMS_ORG` 和 `SANDBOX_NAME`）存储在环境变量中。

>[!IMPORTANT]
>
>在从 Postman 执行任何 API 之前，请务必添加所有必需的环境变量。

## 发布由战术手册创建的历程

有 2 种发布历程的方式；您可以选择其中任一种方式：

1. **使用 AJO UI** - 单击`Bill Of Material Page`上的历程链接；这会将您重定向到历程页面，可单击该页面上的&#x200B;**[!UICONTROL 发布]**&#x200B;按钮来发布历程。

   ![历程对象](../assets/journey-object.png)

1. **使用 Postman API**

   1. 从&#x200B;**[!DNL Journey Publish]** > **[!DNL Queue journey publish job]**&#x200B;中触发&#x200B;**[!DNL Publish Journey]**&#x200B;请求。
   1. 历程发布可能需要一些时间，为了检查状态，请执行检查历程发布状态 API，直到 `response.status` 为 `SUCCESS`，如果历程发布需要一些时间，请务必等待 10-15 秒。

   >[!NOTE]
   >
   >所有环境变量都区分大小写，因此，请始终使用准确的变量名称。

## 摄取客户配置文件

>[!TIP]
>
>可以通过将 `+<variable>` 附加到电子邮件来重用同一电子邮件地址，例如，`usertest@email.com` 可以 `usertest+v1@email.com` 或 `usertest+24jul@email.com` 形式重用。每次都使用新的配置文件，但仍使用同一个电子邮件 ID 将很有帮助。

1. 初始用户需要创建&#x200B;**[!DNL customer dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection]**。
1. 如果您已创建&#x200B;**[!DNL customer dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection]**，请跳至步骤 `5`。
1. 触发&#x200B;**[!DNL Customer Profile Ingestion]** > **[!DNL Create Customer Profile InletId]** > **[!DNL Create Dataset]**&#x200B;以创建&#x200B;**[!DNL customer dataset]**；这将在 Postman 环境变量中存储 `CustomerProfile_dataset_id`。
1. 创建 **[!DNL HTTP Streaming Inlet Connection]**，使用&#x200B;**[!DNL Customer Profile Ingestion > Create Customer Profile InletId]** 下的 Postman API。

   1. `CustomerProfile_dataset_id` 必须在 Postman 环境变量中可用，如果不可用，请参考步骤 `3`。
   1. 触发 **[!DNL `CREATE Base Connection`]** 至[!DNL create base connection]。
   1. 触发 **[!DNL `CREATE Source Connection`]** 至[!DNL create source connection]。
   1. 触发 **[!DNL `CREATE Target Connection`]** 至[!DNL create target connection]。
   1. 触发 **[!DNL `CREATE Dataflow`]** 至[!DNL create dataflow]。
   1. 触发 **[!DNL `GET Base Connection`]** - 这会自动在 Postman 环境变量中存储 `CustomerProfile_inlet_id`。

1. 在此步骤中，您的 Postman 环境变量中必须有 `CustomerProfile_dataset_id` 和 `CustomerProfile_inlet_id`；否则，请分别参考步骤 `3` 或 `4`。
1. 要摄取客户，用户需要在 Postman 环境变量中存储 `customer_country_code`、`customer_mobile_no`、`customer_first_name`、`customer_last_name` 和 `email`。

   1. `customer_country_code` 将是手机号码的国家/地区代码，例如 `91` 或 `1`
   1. `customer_mobile_no` 将是手机号码，例如 `9987654321`
   1. `customer_first_name` 将是用户的名字
   1. `customer_last_name` 将是用户的姓氏
   1. `email` 将是用户的电子邮件地址，这对于使用不同的电子邮件 ID 至关重要，以便获取新的配置文件。

1. 更新 Postman 请求&#x200B;**[!DNL Customer Ingestion]** > **[!DNL Customer Streaming Ingestion]**&#x200B;以更改客户的首选渠道；默认情况下，在请求中配置 [!DNL `email`]。

   ```js
   "consents": {
       "marketing": {
           "preferred": "email",
           "email": {
               "val": "y"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "n"
           }
       }
   }
   ```

1. 将首选渠道更改为 `sms` 或 `push`，并相应的渠道值 `y` 和 `n` 更改为其他值。

   ```js
   "consents": {
       "marketing": {
           "preferred": "sms",
           "email": {
               "val": "n"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "y"
           }
       }
   }
   ```

1. 最后，触发&#x200B;**[!DNL `Customer Profile Ingestion > Customer Profile Streaming Ingestion`]**&#x200B;以摄取客户配置文件。

## 摄取事件

1. 初始用户需要创建&#x200B;**[!DNL event dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection for events]**
1. 如果您已创建&#x200B;**[!DNL event dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection for events]**，请跳至步骤 `5`。
1. 触发&#x200B;**[!DNL `Schemas Data Ingestion > AEP Demo Schema Ingestion > Create AEP Demo Schema InletId > Create Dataset`]**&#x200B;以创建&#x200B;**[!DNL event dataset]**，这将在 Postman 环境变量中存储 `AEPDemoSchema_dataset_id`
1. 创建 **[!DNL HTTP Streaming Inlet Connection for events]**，使用&#x200B;**[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL Create AEP Demo Schema InletId]** 下的 Postman API。

   1. `AEPDemoSchema_dataset_id` 必须在 Postman 环境变量中可用，如果不可用，请参考步骤 `3`
   1. 触发 **[!DNL `CREATE Base Connection`]** 至[!DNL create base connection]
   1. 触发 **[!DNL `CREATE Source Connection`]** 至[!DNL create source connection]
   1. 触发 **[!DNL `CREATE Target Connection`]** 至[!DNL create target connection]
   1. 触发 **[!DNL `CREATE Dataflow`]** 至[!DNL create dataflow]
   1. 触发 **[!DNL `GET Base Connection`]** - 这会自动在 Postman 环境变量中存储 `AEPDemoSchema_inlet_id`

1. 在此步骤中，您的 Postman 环境变量中必须有 `AEPDemoSchema_dataset_id` 和 `AEPDemoSchema_inlet_id`，否则，请分别参考步骤 `3` 或 `4`
1. 要摄取事件，用户需要在 Postman 中更改&#x200B;**[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL AEP Demo Schema Streaming Ingestion]**&#x200B;的请求正文的时间变量 `timestamp`。

   1. `timestamp` 将是事件的发生时间，使用当前时间戳，例如 `2023-07-21T16:37:52+05:30`，根据需要调整时区。

1. 触发 **[!DNL Schemas Data Ingestion > AEP Demo Schema Ingestion > AEP Demo Schema Streaming Ingestion]** 以摄取事件，以便触发历程

## 最终验证

您必须通过在&#x200B;**[!DNL Ingest the Customer Profile]**&#x200B;步骤 `8` 中使用的首选渠道接收消息

* `SMS`，如果首选渠道是 `customer_country_code` 和 `customer_mobile_no` 上的 `sms`
* `Email`，如果首选渠道是 `email` 上的 `email`

或者，您可以查看`Journey Report`，为此，请单击`Bill of Materials page`上的`Journey Object`，这会将您重定向到`Journey Details page`。

对于任何已发布的历程，用户必须获得&#x200B;**[!UICONTROL 查看报告]**按钮
![历程报告页面](../assets/journey-report-page.png)


## 清理

请不要同时运行多个`Journey`实例，如果历程仅用于验证，请在验证完成后停止历程。
