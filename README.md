Todo：借鉴实现原理，移植至Docker容器；增加自动判断域名使用Cloudflare CDN并完成Ech配置注入
![DoHflare Banner](./assets/banner.jpg)
<p align="right">
  <b>简体中文</b> | <a href="./README.en.md">English</a>
</p>

# DoHflare

![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-F38020?logo=cloudflare&logoColor=white) ![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)

一款专为 Cloudflare Workers 部署而设计的 DNS over HTTPS (DoH) 代理。 

DoHflare 将客户端 DNS 查询代理到上游 DoH 解析器，同时强制遵循 TTL 限制、保留 EDNS0 客户端子网（ECS）数据，并通过双层缓存机制（隔离内存 + Cloudflare 缓存 API）管理流量。

## 📑 目录

- [📁 项目结构](#-项目结构)
- [📦️ 部署](#%EF%B8%8F-部署)
  - [方法一：Cloudflare 仪表板（快速部署）](#方法一cloudflare-仪表板快速部署)
  - [方法二：GitHub 集成（CI/CD）](#方法二github-集成cicd)
  - [方法三：Wrangler CLI（本地开发）](#方法三wrangler-cli本地开发)
  - [方法四：Cloudflare Pages（直接上传）](#方法四cloudflare-pages直接上传)
- [⚙️ 配置](#%EF%B8%8F-配置)
  - [环境变量参考表](#环境变量参考表)
- [📖 客户端设置与使用示例](#-客户端设置与使用示例)
- [🌏 中国大陆网络环境特别优化指南](#-中国大陆网络环境特别优化指南)
- [🚨 错误代码与状态映射](#-错误代码与状态映射)
- [⚖️ 平台限制与约束](#%EF%B8%8F-平台限制与约束)
- [📜 许可证与合规性](#-许可证与合规性)

---

## 📁 项目结构

```text
.
├── assets/
│   └── banner.jpg         # 项目主页横幅图片
├── src/
│   └── index.js           # Worker 核心脚本代码
├── .gitignore             # Git 版本控制忽略规则
├── LICENSE                # GNU AGPLv3 开源许可证
├── NOTICE                 # 附加条款与代码归属声明
├── package.json           # 项目元数据与 npm 部署脚本
├── README.md              # 项目主文档（中文）
├── README.en.md           # 项目主文档（英文）
├── SECURITY.md            # 安全策略与漏洞报告指南
└── wrangler.jsonc         # Cloudflare Workers 配置文件与环境变量
```

## 📦️ 部署

根据您的基础设施偏好和 CI/CD 需求，可以通过多种方式部署 DoHflare。

*（注：Cloudflare 仪表板会频繁更新。以下仪表板方法中描述的菜单路径和按钮名称仅供参考，可能随时间略有变化。）*

### 方法一：Cloudflare 仪表板（快速部署）

适合无需本地开发环境的快速部署。

1. 登录  [Cloudflare 仪表板](https://dash.cloudflare.com/)，导航至 **Workers 和 Pages**。
2. 点击 **创建应用程序**。
3. 在 “Ship something new” 菜单下，选择 **从 Hello World! 开始**，为您的 Worker 命名，然后点击 **部署**。
4. 模板部署完成后，点击 **编辑代码**，将默认脚本替换为 `src/index.js` 的原始内容，然后点击 **部署**。
5. 进入 Worker 的 **设置** -> **变量和机密**，配置您的环境变量（参考下文“配置”部分）。
6. 导航至 **设置** -> **域和路由**，点击 **添加** 将代理绑定到特定的自定义域名。

### 方法二：GitHub 集成（CI/CD）

推荐用于自动化部署和版本控制追踪。

1. 将本仓库 **Fork** 到您的个人 GitHub 账户。
2. 在 Cloudflare 仪表板中，导航至  **Workers 和 Pages** -> **创建应用程序**。
3. 选择 **Continue with GitHub**，授权 Cloudflare 访问您 fork 的仓库。
4. 按照提示配置构建设置。确保在 Cloudflare 仪表板设置中定义所有必要的环境变量，以覆盖默认的运行时回退值。

### 方法三：Wrangler CLI（本地开发）

偏好终端操作和本地配置的开发者采用的标准方法。本项目需要 Cloudflare Workers CLI（`wrangler`）。

1. 使用您的 Cloudflare 账户验证本地环境：
```bash
wrangler login
```
2. 使用 `wrangler.jsonc` 中定义的配置部署 Worker：
```bash
wrangler deploy
```

### 方法四：Cloudflare Pages（直接上传）

如果您希望绕过 Workers 流程，而使用 Cloudflare Pages UI，您可以通过单个脚本的高级模式进行部署。

1. 在本地机器上创建一个新的空目录。
2. 将 `src/index.js` 复制到此新目录中，并严格重命名为 `_worker.js`。
3. 在 Cloudflare 仪表板中，导航至 **Workers 和 Pages** -> **创建应用程序**。
4. 点击页面底部“想要部署 Pages？”旁边的 **开始使用**。
5. 在 **拖放文件** 部分下，点击 **开始使用**。
6. 在“通过上传项目部署站点”界面上，在文本框中输入所需项目名称，点击 **创建项目**。
7. 在“上传您的项目资产”下，选择上传一个 **文件夹**，并选择仅包含您的 `_worker.js` 文件的目录。
8. 上传完成后，点击 **部署站点**。
9. 部署成功后，点击 **继续处理项目** 进入项目仪表板。
10. 导航到 **自定义域** 选项卡，将代理绑定到特定的自定义域名，并按照提供的 DNS 设置说明操作。
11. 配置警告：此方法完全忽略 `wrangler.jsonc` 文件。您必须进入 **设置** -> **变量和机密**，手动配置所有必要的环境变量以覆盖硬编码的运行时默认值。

## ⚙️ 配置

所有运行时行为都通过 `wrangler.jsonc` 的 `vars` 块中定义的环境变量或通过 Cloudflare 仪表板进行控制。脚本包含了硬编码的回退默认值；只有当您打算覆盖默认值时，才需要设置变量。

### 环境变量参考表

| 变量                            | 默认值                                                       | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `UPSTREAM_URLS`                 | `["https://dns.google/dns-query", "https://dns11.quad9.net/dns-query"]` | 上游 DoH 解析器的 JSON 数组。                                |
| `DOH_PATH`                      | `/dns-query`                                                 | DoH 请求的监听 URI 路径。                                    |
| `DOH_CONTENT_TYPE`              | `application/dns-message`                                    | DNS 负载的预期 MIME 类型。                                   |
| `ROOT_CONTENT`                  | *（生成的 HTML）*                                            | 在 `/` 路径提供服务的自定义 HTML 内容。                      |
| `ROOT_CONTENT_TYPE`             | `text/html; charset=utf-8`                                   | 根路径响应的 MIME 类型。                                     |
| `ROOT_CACHE_TTL`                | `86400`                                                      | 根页面的生存时间（秒）。                                     |
| `MAX_CACHEABLE_BYTES`           | `65536`                                                      | 缓存 DNS 响应的最大字节数。                                  |
| `MAX_POST_BODY_BYTES`           | `8192`                                                       | 允许的传入 POST 请求的最大字节数。                           |
| `FETCH_TIMEOUT_MS`              | `2500`                                                       | 上游 fetch 操作的超时限制（毫秒）。                          |
| `MAX_RETRIES`                   | `2`                                                          | 上游请求失败时的最大重试次数。                               |
| `DEFAULT_POSITIVE_TTL`          | `60`                                                         | 成功解析但未提供 TTL 时应用的默认 TTL（秒）。                |
| `DEFAULT_NEGATIVE_TTL`          | `15`                                                         | NXDOMAIN 或解析失败时应用的默认 TTL（秒）。                  |
| `FALLBACK_ECS_IP`               | `119.29.29.0`                                                | 客户端 IP 无法解析时为 EDNS 客户端子网注入的默认 IP 地址。   |
| `CF_CACHE_WRITE_THRESHOLD`      | `500`                                                        | 将响应写入 Cloudflare 全局缓存的概率阈值（分母为 10000）。   |
| `GLOBAL_WRITE_COOLDOWN_MS`      | `300000`                                                     | 同一键的全局缓存写入所需的最小间隔（毫秒）。                 |
| `GLOBAL_WRITE_PER_MINUTE_LIMIT` | `200`                                                        | 隔离环境中每分钟允许的全局缓存写入最大次数。                 |
| `GLOBAL_CACHE_NAMESPACE`        | `https://dohflare.local/cache/`                              | 用于在 Cloudflare 缓存 API 中隔离对象的虚拟 URL 命名空间。   |
| `HOT_WINDOW_MS`                 | `60000`                                                      | 跟踪 L1 热缓存请求频率的时间窗口（毫秒）。                   |
| `HOT_HIT_THRESHOLD`             | `20`                                                         | 在热窗口内触发立即全局缓存写入所需的最小命中次数。           |
| `STALE_WINDOW_FACTOR`           | `0.5`                                                        | 应用于 TTL 的乘数，用于确定可接受的“过时-同时-重新验证”窗口。 |
| `EXTREME_STALE_FALLBACK_MS`     | `86400000`                                                   | 所有上游请求失败时，提供过时缓存数据的最大时间（毫秒）。     |
| `JITTER_PCT`                    | `10`                                                         | 应用于 TTL 的确定性抖动百分比，用于减轻缓存雪崩效应。        |

## 📖 客户端设置与使用示例

部署后，您的 DoHflare 代理完全符合现代安全 DNS 标准。您可以配置浏览器通过您的边缘节点路由 DNS 流量，或通过 CLI 工具进行测试。

### 一、浏览器配置

现代网页浏览器原生支持 DNS over HTTPS。以下是设置私有 DoHflare 实例的方法：

* **Microsoft Edge**:
  1. 导航至 `edge://settings/privacy/security`。
  2. 向下滚动，找到并开启 **使用安全的 DNS**。
  3. 选择 **请选择服务提供商**，然后在文本框中输入您的端点 URL。
* **Google Chrome**:
  1. 导航至 `chrome://settings/security`。
  2. 打开 **使用安全 DNS**。
  3. 在 **选择 DNS 提供商** 下拉菜单中，选择 **添加自定义 DNS 服务提供商**，然后在文本框中输入您的端点 URL。

*（注：上述步骤仅供参考，因为浏览器界面可能会随更新而变化。对于其他浏览器，如 Firefox、Safari 或 Brave，请在线搜索具体的 DoH 配置指南。）*

### 二、使用 [natesales/q](https://github.com/natesales/q)

推荐使用 `q` 测试 DoH 端点。它原生处理 DoH 封装，并产生类似 `dig` 的输出。

```bash
# 测试标准 A 记录查询
q A cloudflare.com @https://<YOUR_WORKER_DOMAIN>/dns-query
```

## 🌏 中国大陆网络环境特别优化指南

鉴于中国大陆特殊的网络环境，直接连接 Cloudflare 的 Anycast 节点可能导致高延迟甚至连接重置。为获得极致的 DoH 解析体验，强烈建议中国大陆用户采用 **“自定义域名 + 特征伪装 + CNAME 优选”** 的进阶部署方案。下文将详细介绍各步骤的实施方法。

> [!TIP]
> **随机域名生成建议**：为最大程度规避特征检测，建议使用无连字符的 UUID 作为子域名，例如 `8dc76a899f244efa89793c55367b0930.example.com`。您可以通过 [uuid.rpnet.cc](https://uuid.rpnet.cc) 快速生成此类随机字符串。本文所有示例均基于此随机域名展开。

### 一、主动防御与特征伪装

GFW 会对域名中包含 `doh` 等敏感关键词的子域进行 SNI 嗅探与主动阻断。在绑定自定义域名前，请务必完成以下伪装工作：

1. **规避敏感域名**：切勿使用如 `doh.example.com` 这类包含 `doh` 的子域名。请使用随机生成的字符串或无特定含义的词汇（例如上述 UUID 示例）。
2. **修改默认路由**：在 Cloudflare 环境变量中，将默认的 `DOH_PATH`（`/dns-query`）更改为无特征路径（例如 `/auth-verify` 或 `/query`）。
3. **注入 Nginx 伪装页面**：为防止 GFW 的主动探测爬虫通过访问根目录暴露 DoH 代理特征，请在环境变量中添加 `ROOT_CONTENT`，并填入以下标准的 Nginx 默认欢迎页 HTML 代码，使根目录呈现为一个普通的 Nginx 未配置站点：

```html
<!DOCTYPE html><html><head><title>Welcome to nginx!</title><style>html { color-scheme: light dark; } body { width: 35em; margin: 0 auto; font-family: Tahoma, Verdana, Arial, sans-serif; }</style></head><body><h1>Welcome to nginx!</h1><p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p><p>For online documentation and support please refer to <a href="http://nginx.org/">nginx.org</a>.<br/>Commercial support is available at <a href="http://nginx.com/">nginx.com</a>.</p><p><em>Thank you for using nginx.</em></p></body></html>
```

### 二、将子域名托管至华为云 DNS

为实现境内外流量的分线路解析，需要将您选定的子域名（例如 `8dc76a899f244efa89793c55367b0930.example.com`）的 DNS 解析权完全交给支持地域解析的第三方平台（以华为云 DNS 为例）。

1. **在华为云添加子域名作为独立 DNS 区域** 
   注册并登录华为云，进入**云解析服务 DNS**，点击“添加公网域名”，输入您的完整子域名（如 `8dc76a899f244efa89793c55367b0930.example.com`）。添加成功后，华为云将自动为该子域名分配一组 DNS 服务器地址（通常为 `ns1.huaweicloud-dns.com`、`ns1.huaweicloud-dns.cn` 等）。

2. **在上级 DNS 服务商处设置 NS 记录**
   前往您主域名（`example.com`）当前使用的 DNS 服务商控制台（可能是域名注册商，也可能是 Cloudflare DNS），为您的子域名添加以下四条 NS 记录，将解析权委派给华为云：
   
   - 主机记录：填写您的子域名前缀（例如 `8dc76a899f244efa89793c55367b0930`）  
   - 记录类型：`NS`  
   - 记录值：依次为华为云分配的四条 DNS 服务器地址：
     - `ns1.huaweicloud-dns.com`
     - `ns1.huaweicloud-dns.cn`
     - `ns1.huaweicloud-dns.net`
     - `ns1.huaweicloud-dns.org`
   
   保存后，等待全球 DNS 生效。此后，该子域名及其所有子记录的解析将完全由华为云控制。

### 三、平台回源绑定（建立默认解析线路）

在华为云 DNS 中，我们需要先为子域名添加一条 **“全网默认”线路的 CNAME 记录**，将请求指向 Cloudflare 分配的默认域名。这条记录的作用是让 Cloudflare 能够验证域名的所有权，并建立基础的请求路由，确保后续优选配置能够生效。

#### 方案 A：Workers 部署
- **华为云侧配置**：在子域名的 DNS 记录中，添加一条 CNAME 记录：
  - 主机记录：**留空**（或填写 `@`，表示该子域名本身）
  - 记录类型：`CNAME`
  - 线路类型：**全网默认**
  - 记录值：您的 Worker 默认域名（例如 `your-worker.your-subdomain.workers.dev`）
- **Cloudflare 侧配置**：在 Cloudflare 仪表板中，进入 **Workers 和 Pages**，选择您的 DoHflare Worker，点击“触发器”->“添加路由”，输入 `8dc76a899f244efa89793c55367b0930.example.com/*`，保存。

#### 方案 B：Pages 部署
- **华为云侧配置**：同样添加一条 **全网默认** 线路的 CNAME 记录，记录值填写您的 Pages 默认域名（例如 `your-project.pages.dev`），主机记录留空。
- **Cloudflare 侧配置**：进入 Pages 项目详情页的 **自定义域** 选项卡，点击“设置自定义域”，输入 `8dc76a899f244efa89793c55367b0930.example.com`。
   - *注：若您的主域名 `example.com` 仍托管在 Cloudflare，Cloudflare 可能会自动在其自身 DNS 中添加一条 CNAME 记录以激活域名。由于解析权已委派给华为云，该记录并无实际作用，建议在自定义域状态变为“活动”后，前往 Cloudflare DNS 面板中将其删除。*

### 四、注入 CNAME 优选线路（针对中国大陆加速）

当“全网默认”线路配置完成且功能正常后，即可针对中国大陆流量进行优选加速。在华为云 DNS 控制台中，为同一子域名添加第二条 CNAME 记录，操作如下：

- **主机记录**：留空（表示该子域名本身）
- **记录类型**：`CNAME`
- **线路类型**：选择 **地域解析**，并在展开的下拉框中选中 **中国大陆**
- **记录值**：填写您选择的 Cloudflare 优选域名，例如 `cdn.rpnet.cc` 或 `cloudflare.182682.xyz`
- 点击“确定”保存，等待 DNS 生效。

### 最终效果

通过上述配置，您的 DoH 服务将实现智能分流：
- **境外请求**：命中“全网默认”线路，请求被指向 Cloudflare 默认边缘节点。
- **中国大陆请求**：通过华为云的地域解析精准调度至优选节点，显著降低解析延迟，获得稳定、快速的 DoHflare 体验。

本方案已在实践中验证可行，推荐中国大陆用户采用。

## 🚨 错误代码与状态映射

DoHflare 依赖于标准 HTTP 状态码，并结合自定义头部（`X-DOHFLARE-Code`）进行精确调试，同时不破坏 DNS 负载格式。

| HTTP 状态码                  | 内部代码         | 描述                                                      |
| ---------------------------- | ---------------- | --------------------------------------------------------- |
| `200 OK`                     | 无               | 成功解析（缓存命中或未命中）。                            |
| `400 Bad Request`            | 无               | 缺少 `dns` 参数、无效的 Base64URL 或格式错误的 DNS 查询。 |
| `405 Method Not Allowed`     | 无               | 请求方法不是 `GET`、`POST` 或 `OPTIONS`。                 |
| `413 Payload Too Large`      | 无               | POST 请求体超过 `MAX_POST_BODY_BYTES`。                   |
| `415 Unsupported Media Type` | 无               | POST 请求中缺少或无效的 `Content-Type` 头部。             |
| `502 Bad Gateway`            | `UPSTREAM_ERR`   | 上游解析器无法访问或返回了无效响应。                      |
| `500 Internal Server Error`  | `INTERNAL_FATAL` | 在负载解析或缓存写入期间发生未处理的异常。                |

**内部解析错误（记录在日志中）：**

* `OOB_READ`: 尝试在 ArrayBuffer 中进行越界读取。
* `MALFORMED_LOOP`: 在 DNS 名称压缩方案中检测到无限指针循环。
* `PAYLOAD_TOO_LARGE`: 上游响应超过 `MAX_CACHEABLE_BYTES`。

## ⚖️ 平台限制与约束

DoHflare 的性能受 Cloudflare Workers 平台执行限制的约束。请确保您的部署层级与预期流量规模相匹配。

| 资源                    | Workers 免费版 | Workers 付费版    | 对 DoHflare 的影响                              |
| ----------------------- | -------------- | ----------------- | ----------------------------------------------- |
| **每日请求数**          | 100,000        | 无限制            | 一旦达到限制，代理将返回 HTTP 429。             |
| **每个请求的 CPU 时间** | 10 毫秒        | 5 分钟            | DNS 解析是轻量级的；10 毫秒通常绰绰有余。       |
| **子请求（fetch）**     | 每个请求 50 次 | 每个请求 10000 次 | DoHflare 严格管理上游重试，远低于限制。         |
| **缓存 API 对象大小**   | 512 MB         | 512 MB            | 由 `MAX_CACHEABLE_BYTES`（默认 64KB）安全限制。 |

*有关完整和最新的限制信息，请参考 [Cloudflare Workers 限制文档](https://developers.cloudflare.com/workers/platform/limits)。*

## 📜 许可证与合规性

本软件遵循 **GNU Affero 通用公共许可证 v3.0 (AGPL-3.0)** 进行许可。

**附加声明**：根据 GNU AGPLv3 第 7 条的规定，本作品适用附加条款。有关具体的归属要求和合规政策，请参阅存储库中的 `NOTICE` 文件。

---

*版权所有 © 2026 Racpast。保留所有权利。*
