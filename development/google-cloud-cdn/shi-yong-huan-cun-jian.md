---
description: 'https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1'
---

# 使用缓存键



本页面介绍如何自定义 Cloud CDN 缓存键。

如果新请求开始使用与旧缓存键条目不同的缓存键，则更改缓存键配置后，可能会导致缓存命中率突然下降。同样，如果新请求使用与旧缓存键相同的键，则更改缓存键创建者后，并不一定会使缓存条目失效。要使现有缓存条目失效，请参阅[使缓存内容失效](https://cloud.google.com/cdn/docs/invalidating-cached-content?authuser=1)。**注意**：从缓存键中排除某些组成部分时，可能会导致 Cloud CDN 将面向一位用户的内容传送给另一位用户。在排除其中的组成部分之前，请确保您的后端服务的响应不会因该组成部分而发生变动。

### 准备工作 <a id="before_you_begin"></a>

本页面假定您了解 [Cloud CDN](https://cloud.google.com/cdn/docs/overview?authuser=1)、[Cloud CDN 缓存键](https://cloud.google.com/cdn/docs/caching?authuser=1#cache-keys)和具有负载平衡的[后端服务](https://cloud.google.com/load-balancing/docs/backend-service?authuser=1)。我们建议您在继续之前参阅这些页面。

### 为后端服务启用 Cloud CDN 并自定义缓存键 <a id="enable-cdn"></a>

以下操作说明为具有负载平衡的后端服务激活 CDN，并通过排除一个或多个组件来自定义缓存键。如果您还没有用作源站的负载平衡器，请参阅[外部 HTTP\(S\) 负载平衡](https://cloud.google.com/load-balancing/docs/https?authuser=1)文档，了解如何创建负载平衡器。[控制台](https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list?authuser=1)

2. 点击**添加来源**。
3. 在**来源**下拉菜单中，点击**选择来源**。
4. 选择要为其启用 CDN 的源站。
5. 在源站所在的行中，点击**配置**。
6. 取消选中您想要从此后端服务的缓存键中省略的任何字段的复选框。
7. 点击**保存**。
8. 点击**添加**。

### 更新缓存键以重新添加协议、主机和查询字符串 <a id="updating_cache_keys_to_re-add_protocol_host_and_query_string"></a>

默认情况下，配置为使用 Cloud CDN 的后端服务会在缓存键中包含请求 URI 的所有组成部分。如果您之前已指明应排除一个或多个组件，则可以使用以下步骤再次包含这些组件。**注意**：后端存储分区不支持修改缓存键，并且不在缓存键中包含协议或主机，因为它们不会影响对象在 Cloud Storage 存储分区中的引用方式。如需详细了解来自后端存储分区的响应是如何缓存的，请参阅[缓存概览](https://cloud.google.com/cdn/docs/caching?authuser=1)。

以下操作说明将协议、主机和查询字符串重新添加到已启用 CDN 的现有后端服务的缓存键中。[控制台](https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list?authuser=1)

2. 在您的负载平衡器所在的行中，点击**菜单** more\_vert，然后点击**修改**。
3. 在您要修改的后端服务所在的行中，点击**配置**。
4. 在**缓存键**下，选择**自定义**。
5. 选中**协议**、**主机**和**查询字符串**复选框。
6. 将**查询字符串参数**字段留空。
7. 点击**保存**。
8. 点击**更新**。

### 更新缓存键以使用查询字符串包含列表或排除列表 <a id="updating_cache_keys_to_use_an_include_or_exclude_list_of_query_strings"></a>

以下操作说明将 CDN 缓存键设置为使用查询字符串参数的包含列表或排除列表。[控制台](https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/using-cache-keys?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list?authuser=1)

2. 在您的负载平衡器所在的行中，点击**菜单** more\_vert，然后点击**修改**。
3. 在您要修改的后端服务所在的行中，点击**配置**。
4. 在**缓存键**下，选择**自定义**。
5. 确认您已选中**查询字符串**复选框。
6. 如果您想指定应包括在缓存键中的查询字符串参数，请选择**只包括所选项 \(`whitelist`\)**。

   如果您想指定在缓存键中包括您列出的参数以外的所有查询字符串参数，请选择**包括除所选项之外的所有项 \(`blacklist`\)**。

7. 在**查询字符串参数**字段中输入字符串的逗号分隔列表。
8. 点击**保存**。
9. 点击**更新**。

### 后续步骤 <a id="whats_next"></a>

* 如需检查 Cloud CDN 是否正在从缓存传送响应，请参阅[查看日志](https://cloud.google.com/cdn/docs/logging?authuser=1)。

