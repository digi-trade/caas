---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='mail:support@cabital.com'>申请使用</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - changelog

search: true

code_clipboard: true
---

# 简介

欢迎使用Cabital API，Cabital API 使开发人员能够将cabital专业、安全、可靠的筛查服务集成到现有工作流中，以帮助简化KYC的流程。

API 功能简介：

- 提供个人用户或企业用户的名单筛查能力
- 根据合作伙伴的合规策略，进行筛查结果排查，提供建议
- 从行业领先的数据库持续检索筛查存量客户的能力

您可以根据本文档提供的API来使用服务，支持的全部API请参见KYC Case

# API 认证

API Key是客户访问CaaS认证的唯一方法，所有的API都需要签名并包含以下HTTP Header:

- ACCESS-KEY 客户的Access Key，请联系服务顾问获取
- ACCESS-SIGN 在客户系统中根据下面的规则生成的消息签名
- ACCESS-TIMESTAMP API访问的时间戳

所有的访问必须设置 `Content-Type` 为 `application/json`，并且内容为正规的的JSON。

ACCESS-SIGN Header是通过sha256 HMAC的算法以Secret Key计算`时间戳 + HTTP方法 + 访问路径 + 请求体`。

- 时间戳应该与ACCESS-TIMESTAMP相同
- 请求体为String形式，或者视为空尤其在通过GET方式访问API时
- HTTP方法必须为大写英文

ACCESS-TIMESTAMP 以秒为单位从Unix Epoch到当前的数字，客户请求的时间戳必须与API服务的时间差别30秒以内，否则将被认为是过期或者直接拒绝。


<aside class="warning">请不要分享你的API Key与Secrets，否则会造成信息泄漏以及不必要的费用。</aside>


# KYC Case

## 资源描述

### KYC Create Case

> Sample

```json
{
    "externalCaseId": "243d19cf-562f-4060-89fa-1d35a7723c3e",
    "screenType": "INDIVIDUAL",
    "fullName": "John Doe",
    "individualInfo": {
        "gender": "MALE",
        "dob": "2002-02-02",
        "nationality": "JPN",
        "residentialCountry": "HKG"
    },
    "organizationInfo": {
        "registeredCountry": "HKG"
    },
    "memo": "case memo"
}
```

客户创建KYC Case，包含有用户个人或业务实体的基本信息。

字段 | 类型 | 必须       | 描述
--------- | ------- | ---------------| -----------
externalCaseId | string | true| 客户自己可以选择的外部Id，字符长度最大36位Unicode
screenType | string(ENUM) | true| Case的扫描类型，为枚举字符串，选择为`INDIVIDUAL, ORGANISATION`
fullName | string | true| 扫描对象的全称，个体名字或者实体名字,字符长度最大250位Unicode
individualInfo | object | false| 用户个体的信息, 二选一
organizationInfo | object | false| 业务实体的信息, 二选一
memo | string | false | 客户case的备注信息，字符长度最大255位Unicode


### KYC Get Case

> Sample

```json
{
    "externalCaseId": "243d19cf-562f-4060-89fa-1d35a7723c3e",
    "screenType": "INDIVIDUAL", // INDIVIDUAL, ORGANISATION
    "fullName": "John Doe",
    "individualInfo": {
        "gender": "MALE",
        "dob": "2002-02-02",
        "nationality": "JPN",
        "residentialCountry": "HKG"
    },
    "organizationInfo": {
        "registeredCountry": "HKG"
    },
    "memo": "case memo",
    "suggestion": "SUGGEST_TO_ACCEPT",
    "suggestionComment": "resolve by operator",
    "decision": "ACCEPT",
    "decisionComment": "default comment",
    "status": "COMPLETE"
}
```

客户查询KYC Case，包含case的处理意见。

字段 | 类型 | 描述
--------- | ------- | ---------------
externalCaseId | string | 客户自己可以选择的外部Id，字符长度最大36位Unicode
screenType | string(ENUM) | Case的扫描类型，为枚举字符串，选择为`INDIVIDUAL, ORGANISATION`
fullName | string | 扫描对象的全称，个体名字或者实体名字
individualInfo | object | 用户个体的信息, 二选一
organizationInfo | object | 业务实体的信息, 二选一
memo | string | 客户case的备注信息，字符长度最大255位Unicode
suggestion | string(ENUM) | 系统建议 `SUGGEST_TO_ACCEPT,SUGGEST_TO_REJECT,NO_SUGGESTION`
suggestionComment | string | 系统建议的人工备注
decision | string(ENUM) | 客户决定 `REJECT, ACCEPT`
decisionComment | string | 客户决定的人工备注
status | string(ENUM) | case状态 `NEW,PENDING,WAITING_APPROVAL,COMPLETE`
ogs | bool | 持续性扫描状态



### IndividualInfo

客户创建客户的个人用户基本信息。

字段 | 类型 | 必须 | 描述
--------- | --------- | ------- | -----------
gender | string | false| 性别，为枚举字符串，选择为`MALE, FEMALE`
dob | string | false| Date of Birth 生日， yyyy-MM-dd
nationality | string (alpha-3) | false| 用户的国籍 (ISO-3166-1 alpha-3)
residentialCountry | string (alpha-3) | false| 用户的居住国家 (ISO-3166-1 alpha-3)


### OrganizationInfo

客户创建客户的企业用户基本信息。

字段 | 类型 | 必须 | 描述
--------- | --------- | ------- | -----------
registeredCountry | string (alpha-3) | false | 企业注册国家 (ISO-3166-1 alpha-3)



## 创建 KYC Case

```shell
curl "http://caas.cabital.com/api/v1/cases" \
  -H "ACCESS-KEY: YOUR_KEY" \
  -H "ACCESS-SIGN: SIGN_KEY" \
  -H "ACCESS-TIMESTAMP: 1623828308" \
  -X POST
```

> HTTP返回 (200)

> The above command returns JSON structured like this:

```json
{
  "caseSystemId": "243d19cf-562f-4060-89fa-1d35a7723c3e", // UUID
  "status": "NEW"
}
```

创建 KYC Case

### HTTP Request

`POST https://caas.cabital.com/api/v1/cases`

[RequestBody](#kyc-create-case)

### HTTP Response

字段 | 类型 | 描述
--------- | ------- | ---------------
caseSystemId | string | KYC Case system UUID
status | string(ENUM) | case状态 `NEW,PENDING,WAITING_APPROVAL,COMPLETE`


<aside class="success">
客户在打开<i> Zero Footprint</i> 的配置下，创建 KYC Case 将自动触发一次性扫描!
</aside>

## 获取特定的 KYC Case


```shell
curl "http://caas.cabital.com/api/v1/cases/2" \
  -H "ACCESS-KEY: YOUR_KEY" \
  -H "ACCESS-SIGN: SIGN_KEY" \
  -H "ACCESS-TIMESTAMP: 1623828308"
```


> The above command returns JSON structured like this:

```json
{
    "externalCaseId": "243d19cf-562f-4060-89fa-1d35a7723c3e",
    "screenType": "INDIVIDUAL", // INDIVIDUAL, ORGANISATION
    "fullName": "John Doe",
    "individualInfo": {
        "gender": "MALE",
        "dob": "2002-02-02",
        "nationality": "JPN",
        "residentialCountry": "HKG"
    },
    "organizationInfo": {
        "registeredCountry": "HKG"
    },
    "memo": "case memo",
    "suggestion": "SUGGEST_TO_ACCEPT",
    "suggestionComment": "resolve by operator",
    "decision": "ACCEPT",
    "decisionComment": "default comment",
    "status": "COMPLETE"
}
```


### HTTP Request

`GET http://caas.cabital.com/api/v1/cases/{caseSystemId}`


### URL Parameters

参数 | 类型 | 描述
--------- | ----- |-----------
caseSystemId |string|  KYC Case system UUID

### HTTP Response

[ResponseBody](#kyc-get-case)

## 打开特定 KYC Case 的持续性扫描


```shell
curl "https://caas.cabital.com/api/v1/cases/6d92e7b4-715c-4ce3-a028-19f1c8c9fa6c/ogs" \
  -X POST \
  -H "ACCESS-KEY: YOUR_KEY" \
  -H "ACCESS-SIGN: SIGN_KEY" \
  -H "ACCESS-TIMESTAMP: 1623828308" 
```

> HTTP返回 (201)

针对特定KYC Case打开持续扫描

### HTTP Request

`POST https://caas.cabital.com/api/v1/cases/{caseSystemId}/ogs`

### URL Parameters

参数 | 描述
--------- | -----------
caseSystemId | KYC Case system UUID


## 关闭特定 KYC Case 的持续性扫描


```shell
curl "https://caas.cabital.com/api/v1/cases/6d92e7b4-715c-4ce3-a028-19f1c8c9fa6c/ogs" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

> HTTP返回 (204)

针对特定KYC Case关闭持续扫描

### HTTP Request

`DELETE https://caas.cabital.com/api/v1/cases/{caseSystemId}/ogs`

### URL Parameters

参数 | 描述
--------- | -----------
caseSystemId | KYC Case system UUID


## 提供特定 KYC Case 的最终决定


```shell
curl "https://caas.cabital.com/api/v1/cases/6d92e7b4-715c-4ce3-a028-19f1c8c9fa6c/decision" \
  -X POST \
  -H "Authorization: meowmeowmeow"
```

> HTTP返回 (201)

针对特定KYC Case提供最终KYC决定，caseResult为枚举型String：

- ACCEPT
- REJECT

### HTTP Request

`POST https://caas.cabital.com/api/v1/cases/{caseSystemId}/decision`

> Request Body

```json
{
  "decision" : "ACCEPT",
  "comment" : "决策备注"
}
```
### URL Parameters

参数 | 描述
--------- | -----------
caseSystemId | KYC Case system UUID

# Webhook

webhook 预计平均会在 3-5 分钟后到达，但理论上可能需要长达 24 小时。如果由于某种原因您错过了 webhook，请不要担心：我们会记录尝试发送的所有内容，并且可以随时重新发送失败的 webhook。如果 webhook 请求失败，我们会尝试重新发送四次：5 分钟、1 小时、5 小时和 18 小时后，直到请求成功获得 2XX 返回码的。

我们建议您等待 webhook 不超过一天，然后向我们的服务器发送请求以获取有关资源的状态信息。

## Webhooks types

类型 | 描述
--------- | -----------
CaseCreated | case成功创建后通知,`Zero Footprint` model not send
CasePending | case成功扫描后通知
CaseReviewed | case产生扫描建议后通知
CaseCompleted | case获取客户结果后通知

当系统给出的建议为`NO_SUGGESTION`时，需要由具体的operator进行处理. 处理完毕，我们会将结果通过 `CaseCompleted` webhook进行通知

## Webhook Payload

### Normal Payload

> HTTP Payload
 
```json
{
    "caseSystemId": "6d92e7b4-715c-4ce3-a028-19f1c8c9fa6c",
    "externalCaseId": "243d19cf-562f-4060-89fa-1d35a7723c3e"
}
```

`CaseCreated` `CasePending` webhook payload

字段 | 类型 | 必须 | 描述
--------- | ------- | ------------|-----------
externalCaseId | string | true | 客户填写的外部Id
caseSystemId | string | true | Case的system uuid

### Reviewed Payload

> HTTP Payload
 
```json
{
    "caseSystemId": "6d92e7b4-715c-4ce3-a028-19f1c8c9fa6c",
    "externalCaseId": "243d19cf-562f-4060-89fa-1d35a7723c3e",
    "suggestion": "SUGGEST_TO_REJECT",
    "comment": "User is okay to me."
}
```

字段 | 类型 | 必须 | 描述
--------- | ------- | ------------|-----------
externalCaseId | string | true | 客户填写的外部Id
caseSystemId | string | true | Case的system uuid
suggestion | string(ENUM) | true | 系统建议 `SUGGEST_TO_ACCEPT,SUGGEST_TO_REJECT,NO_SUGGESTION`
comment | string | false| 系统建议的人工备注

### Completed Payload

> HTTP Payload
 
```json
{
    "caseSystemId": "6d92e7b4-715c-4ce3-a028-19f1c8c9fa6c",
    "externalCaseId": "243d19cf-562f-4060-89fa-1d35a7723c3e",
    "decision": "ACCEPT",
    "comment": "User is okay to me."
}
```

字段 | 类型 | 必须 | 描述
--------- | ------- | ------------|-----------
externalCaseId | string | true | 客户填写的外部Id
caseSystemId | string | true | Case的system uuid
decision | string(ENUM) | true | 决策 `ACCEPT,REJECT`
comment | string | false| 决策备注

case的状态发生变化时，我们将向您发送带有 JSON 负载的 POST 请求到集成时提供给我们的 URL。

## Webhook Signature

Webhook 签名规则与API认证一致，请参考[API认证](#api)
