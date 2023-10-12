---
title: 单一事件
description: 这是用于模拟“[!UICONTROL 单一事件]”类型的历程验证的说明页面。
exl-id: 314f967c-e10f-4832-bdba-901424dc2eeb
source-git-commit: 194667c26ed002be166ab91cc778594dc1f09238
workflow-type: tm+mt
source-wordcount: '889'
ht-degree: 37%

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

* 使用战术手册创建实例资产，例如&#x200B;**[!UICONTROL 历程]**、**[!UICONTROL 架构]**、**[!UICONTROL 区段]**、**[!UICONTROL 消息]**&#x200B;等。

* 创建的资产将显示在`Bill Of Material`页面上

<!-- TODO: attached image needs to change once postman is removed from UI -->
![物料清单页面](../assets/bom-page.png)

>[!TIP]
>
>如果您使用终端来运行CURL，则可以在运行CURL之前设置变量值，这样便无需在单个CURL中替换这些值。
>例如：如果您设置 `ORG_ID=************@AdobeOrg`，shell将自动替换每次出现 `$ORG_ID` ，以便您无需进行任何修改即可复制、粘贴和执行下面的曲线。
>
> 本文档中使用以下变量
>
> ACCESS_TOKEN
>
> API_KEY
>
> ORG_ID
>
> 沙盒名称
>
> PROFILE_SCHEMA_REF
>
> PROFILE_DATASET_NAME
>
> PROFILE_DATASET_ID
>
> 历程ID
>
> PROFILE_BASE_CONNECTION_ID
>
> PROFILE_SOURCE_CONNECTION_ID
>
> PROFILE_TARGET_CONNECTION_ID
>
> PROFILE_INLET_URL
>
> CUSTOMER_MOBILE_NUMBER
>
> CUSTOMER_FIRSTNAME
>
> CUSTOMER_LAST_NAME
>
> 电子邮件
>
> EVENT_SCHEMA_REF
>
> EVENT_DATASET_NAME
>
> EVENT_DATASET_ID
>
> EVENT_BASE_CONNECTION_ID
>
> EVENT_SOURCE_CONNECTION_ID
>
> EVENT_TARGET_CONNECTION_ID
>
> EVENT_INLET_URL
>
> 时间戳
>
> UNIQUE_EVENT_ID

## 提取 IMS 令牌

1. 请按照[验证和访问 Experience Platform API](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) 文档中的说明操作以生成访问令牌。

## 发布由战术手册创建的历程

有 2 种发布历程的方式；您可以选择其中任一种方式：

1. **使用 AJO UI** - 单击`Bill Of Material Page`上的历程链接；这会将您重定向到历程页面，可单击该页面上的&#x200B;**[!UICONTROL 发布]**&#x200B;按钮来发布历程。

   ![历程对象](../assets/journey-object.png)

1. **使用cURL**

   1. 发布历程。 响应将包含提取历程发布状态时下一步所需的作业ID。

      ```bash
      curl --location --request POST "https://journey-private.adobe.io/authoring/jobs/journeyVersions/$JOURNEY_ID/deploy" \
      --header "Accept: */*" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-api-key: $API_KEY" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" 
      ```

   1. 历程发布可能需要一些时间，因此要检查状态，请在cURL下方执行，直到 `response.status` 是 `SUCCESS`，请确保等待10-15秒（如果历程发布需要时间）。

      ```bash
      curl --location "https://journey-private.adobe.io/authoring/jobs/$JOB_ID" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-api-key: $API_KEY" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json"
      ```

## 摄取客户配置文件

>[!TIP]
>
>如果您的电子邮件提供商支持加电子邮件，您可以通过附加来重复使用相同的电子邮件地址 `+<variable>` 放入您的电子邮件中，例如 `usertest@email.com` 可以恢复为 `usertest+v1@email.com` 或 `usertest+24jul@email.com`. 每次都使用新的配置文件，但仍使用同一个电子邮件 ID 将很有帮助。
>
>P.S： Plus电子邮件是一项可配置的功能，需要电子邮件提供商提供支持。 在测试中使用此类地址之前，请检查您是否能够接收此类地址的电子邮件。

1. 初始用户需要创建&#x200B;**[!DNL customer dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection]**。
1. 如果您已创建&#x200B;**[!DNL customer dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection]**，请跳至步骤 `5`。
1. 通过执行以下cURL创建客户用户档案数据集。

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/catalog/dataSet" \
   --header "Authorization: Bearer $ACCESS_TOKEN" \
   --header "x-gw-ims-org-id: $ORG_ID" \
   --header "x-sandbox-name: $SANDBOX_NAME" \
   --header "x-api-key: $API_KEY" \
   --header "Content-Type: application/json" \
   --data '{
       "name": "'$PROFILE_DATASET_NAME'",
       "schemaRef": {
           "id": "'$PROFILE_SCHEMA_REF'",
           "contentType": "application/vnd.adobe.xed-full-notext+json; version=1"
       },
       "tags": {
           "unifiedProfile": [
           "enabled:true"
           ],
           "unifiedIdentity": [
           "enabled:true"
           ]
       },
       "fileDescription": {
           "persisted": true,
           "containerFormat": "parquet",
           "format": "parquet"
       }
   }'
   ```

   响应将采用格式 `"@/dataSets/<PROFILE_DATASET_ID>"`.

1. 创建 **[!DNL HTTP Streaming Inlet Connection]** 并通过以下步骤帮助完成这些任务。
   1. 创建基本连接。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Base_ConnectionForCustomerProfile_1694458293",
          "description": "Marketer Playground Playbook-Validation Customer Profile Base Connection 1",
          "auth": {
              "specName": "Streaming Connection",
              "params": {
                  "dataType": "xdm"
              }
          },
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      从响应中获取基本连接ID并使用它代替 `PROFILE_BASE_CONNECTION_ID` 在以下cURL中

   1. 创建源连接。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Source_ConnectionForCustomerProfile_1694458318",
          "description": "Marketer Playground Playbook-Validation Customer Profile Source Connection 1",
          "baseConnectionId": "'$PROFILE_BASE_CONNECTION_ID'",
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      从响应中获取源连接ID并使用它代替 `PROFILE_SOURCE_CONNECTION_ID`

   1. 创建目标连接。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/targetConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Target_ConnectionForCustomerProfile_1694458407",
          "description": "Marketer Playground Playbook-Validation Customer Profile Target Connection 1",
          "data": {
              "format": "parquet_xdm",
              "schema": {
                  "version": "application/vnd.adobe.xed-full+json;version=1",
                  "id": "'$PROFILE_SCHEMA_REF'"
              },
              "properties": null
          },
          "connectionSpec": {
              "id": "c604ff05-7f1a-43c0-8e18-33bf874cb11c",
              "version": "1.0"
          },
          "params": {
              "dataSetId": "'$PROFILE_DATASET_ID'"
          }
      }'
      ```

      从响应中获取目标连接ID并使用它代替 `PROFILE_TARGET_CONNECTION_ID`

   1. 创建数据流。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/flows" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Dataflow_ForCustomerCustomerProfile_1694460528",
          "description": "Marketer Playground Playbook-Validation Customer Profile Dataflow 1",
          "flowSpec": {
              "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
              "version": "1.0"
          },
          "sourceConnectionIds": [
              "'$PROFILE_SOURCE_CONNECTION_ID'"
          ],
          "targetConnectionIds": [
              "'$PROFILE_TARGET_CONNECTION_ID'"
          ]
      }'
      ```

   1. 获取基本连接。 结果将包含发送配置文件数据所需的inletUrl。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections/$PROFILE_BASE_CONNECTION_ID" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY"
      ```

      从响应中获取inletUrl并使用它来代替 `PROFILE_INLET_URL`

1. 在此步骤中，用户必须具有值 `PROFILE_DATASET_ID` 和 `PROFILE_INLET_URL`；如果不能，请参阅步骤 `3` 或 `4` 的量度。
1. 要摄取客户，用户需要替换 `CUSTOMER_MOBILE_NUMBER`， `CUSTOMER_FIRST_NAME`， `CUSTOMER_LAST_NAME` 和 `EMAIL` 在以下cURL中。

   1. `CUSTOMER_MOBILE_NUMBER` 将是手机号码，例如 `+1 000-000-0000`
   1. `CUSTOMER_FIRST_NAME` 将是用户的名字
   1. `CUSTOMER_LAST_NAME` 将是用户的姓氏
   1. `EMAIL` 将是用户的电子邮件地址，这对于使用不同的电子邮件 ID 至关重要，以便获取新的配置文件。

1. 最后，执行curl以摄取客户资料。 更新 `body.xdmEntity.consents.marketing.preferred` 到 `email`， `sms`，或 `push` 基于要验证的渠道。 同时设置相应的 `val` 到 `y`.

   ```bash
   curl --location "$PROFILE_INLET_URL?synchronousValidation=true" \
   --header 'Content-Type: application/json' \
   --data-raw '{
       "header": {
           "schemaRef": {
               "id": "'$PROFILE_SCHEMA_REF'",
               "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
           },
           "imsOrgId": "'$ORG_ID'",
           "datasetId": "'$PROFILE_DATASET_ID'",
           "source": {
               "name": "Streaming dataflow - 1694460605"
           }
       },
       "body": {
           "xdmMeta": {
               "schemaRef": {
                   "id": "'$PROFILE_SCHEMA_REF'",
                   "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
               }
           },
           "xdmEntity": {
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
           },
           "mobilePhone": {
               "number": "'$CUSTOMER_MOBILE_NUMBER'",
               "status": "active"
           },
           "person": {
               "name": {
               "firstName": "'$CUSTOMER_FIRST_NAME'",
               "lastName": "'$CUSTOMER_LAST_NAME'"
               }
           },
           "personalEmail": {
               "address": "'$EMAIL'"
           },
           "testProfile": false
           }
       }
   }'
   ```

## 摄取历程触发事件

1. 初始用户需要创建&#x200B;**[!DNL event dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection for events]**
1. 如果您已创建&#x200B;**[!DNL event dataset]**&#x200B;和 **[!DNL HTTP Streaming Inlet Connection for events]**，请跳至步骤 `5`。
1. 通过执行以下cURL创建事件数据集。

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/catalog/dataSet" \
   --header "Authorization: Bearer $ACCESS_TOKEN" \
   --header "x-gw-ims-org-id: $ORG_ID" \
   --header "x-sandbox-name: $SANDBOX_NAME" \
   --header "x-api-key: $API_KEY" \
   --header "Content-Type: application/json" \
   --data '{
       "name": "'$EVENT_DATASET_NAME'",
       "schemaRef": {
           "id": "'$EVENT_SCHEMA_REF'",
           "contentType": "application/vnd.adobe.xed-full-notext+json; version=1"
       },
       "tags": {
           "unifiedProfile": [
               "enabled:true"
           ],
           "unifiedIdentity": [
               "enabled:true"
           ]
       },
       "fileDescription": {
           "persisted": true,
           "containerFormat": "parquet",
           "format": "parquet"
       }
   }'
   ```

   响应将采用格式 `"@/dataSets/<EVENT_DATASET_ID>"`

1. 创建 **[!DNL HTTP Streaming Inlet Connection for events]**  并通过以下步骤帮助完成这些任务。
   <!-- TODO: Is the name unique? If so, we may need to generate and provide in variables.txt-->
   1. 创建基本连接。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Base_ConnectionForAEPDemoSchema_1694461448",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Base Connection 1",
          "auth": {
              "specName": "Streaming Connection",
              "params": {
                  "dataType": "xdm"
              }
          },
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      从响应中获取基本连接ID并使用它代替 `EVENT_BASE_CONNECTION_ID`

   1. 创建源连接。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Source_ConnectionForAEPDemoSchema_1694461464",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Source Connection 1",
          "baseConnectionId": "'$EVENT_BASE_CONNECTION_ID'",
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      从响应中获取源连接ID并使用它代替 `EVENT_SOURCE_CONNECTION_ID`

   1. 创建目标连接。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Target_ConnectionForAEPDemoSchema_1694802667",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Target Connection 1",
          "data": {
              "format": "parquet_xdm",
              "schema": {
                  "version": "application/vnd.adobe.xed-full+json;version=1",
                  "id": "'$EVENT_SCHEMA_REF'"
              },
              "properties": null
          },
          "connectionSpec": {
              "id": "c604ff05-7f1a-43c0-8e18-33bf874cb11c",
              "version": "1.0"
          },
          "params": {
              "dataSetId": "'$EVENT_DATASET_ID'"
          }
      }'
      ```

      从响应中获取目标连接ID并使用它代替 `EVENT_TARGET_CONNECTION_ID`

   1. 创建数据流。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/flows" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Dataflow_ForCustomerAEPDemoSchema_1694461564",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Dataflow 1",
          "flowSpec": {
              "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
              "version": "1.0"
          },
          "sourceConnectionIds": [
              "'$EVENT_SOURCE_CONNECTION_ID'"
          ],
          "targetConnectionIds": [
              "'$EVENT_TARGET_CONNECTION_ID'"
          ]
      }'
      ```

   1. 获取基本连接。 结果将包含发送配置文件数据所需的inletUrl。

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/flowservice/connections/$EVENT_BASE_CONNECTION_ID" \
       --header "Authorization: Bearer $ACCESS_TOKEN" \
       --header "x-gw-ims-org-id: $ORG_ID" \
       --header "x-sandbox-name: $SANDBOX_NAME" \
       --header "x-api-key: $API_KEY" \
       --header "Content-Type: application/json" 
   ```

   从响应中获取inletUrl并使用它来代替 `EVENT_INLET_URL`

1. 在此步骤中，用户必须具有值 `EVENT_DATASET_ID` 和 `EVENT_INLET_URL`；如果不能，请参阅步骤 `3` 或 `4` 的量度。
1. 要摄取事件，用户需要更改时间变量 `TIMESTAMP` cURL的请求正文中。

   1. 替换 `body.xdmEntity` 包含下载的事件json的内容。
   1. `TIMESTAMP` 是事件发生的时间，请使用UTC时区的时间戳，例如 `2023-09-05T23:57:00.071+00:00`.
   1. 为变量设置唯一值 `UNIQUE_EVENT_ID`.

   ```bash
   curl --location "$EVENT_INLET_URL?synchronousValidation=true" \
   --header 'Content-Type: application/json' \
   --data-raw '{
       "header": {
           "schemaRef": {
               "id": "'$EVENT_SCHEMA_REF'",
               "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
           },
           "imsOrgId": "'$ORG_ID'",
           "datasetId": "'$EVENT_DATASET_ID'",
           "source": {
               "name": "Streaming dataflow - 8/31/2023 9:04:25 PM"
           }
       },
       "body": {
           "xdmMeta": {
               "schemaRef": {
                   "id": "'$EVENT_SCHEMA_REF'",
                   "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
               }
           },
           "xdmEntity": {
               "endUserIDs": {
                   "_experience": {
                       "aaid": {
                           "id": "'$EMAIL'"
                       },
                       "emailid": {
                           "id": "'$EMAIL'"
                       }
                   }
               },
               "_experience": {
                   "analytics": {
                       "customDimensions": {
                           "eVars": {
                           "eVar235": "AC11147"
                           }
                       }
                   }
               },
               "_id": "'$UNIQUE_EVENT_ID'",
               "commerce": {
                   "productListAdds": {
                       "value": 11498
                   }
               },
               "eventType": "commerce.productListAdds",
               "productListItems": [
                   {
                       "_id": "ACS1620",
                       "SKU": "P1",
                       "_experience": {
                           "analytics": {
                           "customDimensions": {
                               "eVars": {
                                   "eVar1": "Pants"
                               }
                           }
                           }
                       },
                       "currencyCode": "USD",
                       "name": "Sample value",
                       "priceTotal": 30841.13,
                       "product": "https://ns.adobe.com/xdm/common/uri",
                       "productAddMethod": "Sample value",
                       "quantity": 1
                   },
                   {
                       "_id": "ACS1729",
                       "SKU": "P2",
                       "_experience": {
                           "analytics": {
                               "customDimensions": {
                                   "eVars": {
                                       "eVar1": "Galliano"
                                   }
                               }
                           }
                       },
                       "currencyCode": "USD",
                       "name": "Sample value",
                       "priceTotal": 20841.13,
                       "product": "https://ns.adobe.com/xdm/common/uri",
                       "productAddMethod": "Sample value",
                       "quantity": 2
                   }
               ],
               "timestamp": "'$TIMESTAMP'",
               "web": {
                   "webInteraction": {
                       "URL": "https://experienceleague.adobe.com/docs/experience-platform/edge/data-collection/collect-commerce-data.html?lang=en",
                       "name": "Sample value",
                       "region": "Sample value"
                   },
                   "webPageDetails": {
                       "URL": "https://experienceleague.adobe.com/docs/experience-platform/edge/data-collection/collect-commerce-data.html?lang=en",
                       "isErrorPage": false,
                       "isHomePage": false,
                       "name": "Sample value",
                       "pageViews": {
                           "id": "Sample value",
                           "value": 1
                       },
                       "server": "Sample value",
                       "siteSection": "Sample value",
                       "viewName": "Sample value"
                   },
                   "webReferrer": {
                   "URL": "Sample value",
                   "type": "internal"
                   }
               }
           }
       }
   }'
   ```

## 最终验证

您必须通过在&#x200B;**[!DNL Ingest the Customer Profile]**&#x200B;步骤 `8` 中使用的首选渠道接收消息

* `SMS`，如果首选渠道是 `customer_country_code` 和 `customer_mobile_no` 上的 `sms`
* `Email`，如果首选渠道是 `email` 上的 `email`

或者，您可以查看`Journey Report`，为此，请单击`Bill of Materials page`上的`Journey Object`，这会将您重定向到`Journey Details page`。

对于任何已发布的历程，用户必须获得&#x200B;**[!UICONTROL 查看报告]**按钮
![历程报告页面](../assets/journey-report-page.png)


## 清理

请不要同时运行多个`Journey`实例，如果历程仅用于验证，请在验证完成后停止历程。
