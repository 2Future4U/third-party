---


copyright:
  years: 2018
lastupdated: "2018-07-20"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# 概述：开发 Integrated Billing 服务
{: #overview}

本主题介绍了在 {{site.data.keyword.Bluemix}} 中开发和发布第三方 Integrated Billing 服务所需的步骤。
{:shortdesc}

您获得批准在 {{site.data.keyword.Bluemix_notm}}“目录”中交付产品后，将在资源管理控制台中开始开发产品，这是指导式 UI 体验。您将设计目录元数据，设置服务价格套餐，以及与 {{site.data.keyword.Bluemix_notm}} Identity and Access Management (IAM) 相集成。 

接下来，在资源管理控制台外部，您将执行代码开发以构建和托管新的 Open Service Broker（将提供入门模板代理程序样本和 API 文档），并且将使用 IAM 来开发认证流程。完成这些步骤后，您将返回到资源管理控制台，以有限可视性方式将服务部署到目录中，这将允许您测试和验证多个可交付产品需求。准备就绪并且产品满足所有需求时，服务将在 {{site.data.keyword.Bluemix_notm}}“目录”中完全可视！


## 过程工作方式
{: #process}

要交付 Integrated Billing 服务，您可使用 Provider Workbench、资源管理控制台和所选开发环境。请参阅[核对表](/docs/third-party/checklist.html#checklist)以帮助跟踪这些步骤。

1. [创建文档和市场营销公告](/docs/third-party/cis1-docs-marketing.html)。
2. [在资源管理控制台中定义产品{{site.data.keyword.Bluemix_notm}}](/docs/third-party/cis2-rmc-define.html)。
3. [开发和托管服务代理程序](/docs/third-party/cis3-broker.html)。
4. [开发认证流程](/docs/third-party/cis5-iam.html)。
5. [测试服务](/docs/third-party/cis4-rmc-publish.html)。
6. [公开发布服务](/docs/third-party/cis6-ga.html)。

这些步骤假定您已获得批准交付 Integrated Billing 服务。如果您尚未完成 Provider Workbench 中的初始注册和批准过程，请参阅[入门教程](/docs/third-party/index.md)。
{: tip}

## 支持
{: #support}

Integrated Billing 服务需要第三方产品提供者提供自己的支持模型。IBM 代表可帮助您启用支持模型，并确保如果用户向 {{site.data.keyword.Bluemix_notm}} 支持开具凭单，该凭单会路由到第三方支持站点。

## 持续更新
{: #maintain}

发布服务后，可以通过直接与 {{site.data.keyword.Bluemix_notm}} 代表合作来继续对服务进行迭代和更新。


