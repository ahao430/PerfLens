好的。我会整理成一个**完整单文件版 `plan.md`**，定位是：

- 可以直接交给 Claude Code 阅读；
- 不展开到每个代码接口细节（避免过早约束实现）；
- 但是把产品边界、架构、模块、技术路线、关键设计决策说明清楚；
- 后续再让 Claude Code 拆成 `architecture.md / event-schema.md / runtime-sdk.md / roadmap.md` 等。

由于完整版本较长，我会分几次输出，避免中间截断。

第一部分包含：

1. 项目概述
2. 产品目标
3. 核心理念
4. 总体架构
5. Monorepo设计
6. 技术选型
7. Session模型

下一条继续补：

8. Adapter设计  
9. H5方案  
10. 微信方案  
11. 支付宝Runtime SDK方案  
12. 小程序性能分析体系  
13. AI Analyzer  
14. MCP/Skill  
15. Dashboard  
16. 导出  
17. 开发路线

---

# PerfLens Plan

```md
# PerfLens

> AI-powered Frontend Performance Engineering Platform

## 1. 项目介绍

PerfLens 是一个 AI 驱动的前端性能工程平台。

目标：

通过 Runtime 数据采集、自动化测试、源码分析和 AI 推理能力，帮助开发者定位 Web 和 MiniApp 应用中的性能问题。

PerfLens 不只是性能监控工具，而是：

> 一个理解前端 Runtime、源码和业务流程的 AI 性能工程师。


---

# 2. 产品定位

传统性能工具：

```
采集指标

↓

展示数据

↓

人工分析
```


PerfLens：

```
采集 Runtime 数据

+

分析项目源码

+

理解业务流程

+

AI 推理


↓

定位性能根因

↓

生成优化建议
```


---

# 3. 支持平台


第一阶段：

## H5

基于：

- Playwright
- Chrome DevTools Protocol
- Browser Performance API


---

## 微信小程序

基于：

- 微信自动化 SDK
- 微信性能 API
- Runtime Hook


---

## 支付宝小程序（重点）

基于：

- Runtime SDK
- my.performance
- 真机调试采集


原因：

支付宝小程序大量用于企业业务，但是：

- 自动化能力不足
- 缺少成熟性能分析方案
- Web 性能工具无法覆盖


---

# 4. 核心设计原则


## 4.1 AI优先

系统不仅输出：

```
首页加载慢
```

而应该输出：

```
首页加载慢

原因：

onShow阶段同步执行三个接口

位置：

pages/home/index.js

影响：

首次进入页面

建议：

拆分首屏逻辑

风险：

低
```


---

## 4.2 Runtime优先

不同平台拥有不同 Runtime。

不能使用统一 Web 性能模型。


例如：

H5：

```
DOM
CSS
Layout
Paint
Browser Runtime
```


小程序：

```
JS Runtime

↓

Bridge

↓

Native Render

```


因此：

H5 Adapter 和 MiniApp Adapter 使用不同分析规则。


---

## 4.3 不自动修改生产代码


尤其小程序。


禁止：

- 自动调整分包
- 自动修改路由
- 自动迁移页面
- 自动修改业务逻辑


原因：

线上业务风险高。


输出：

```
问题

原因

影响范围

优化方案

风险等级

实施步骤
```


---

# 5. 总体架构


```
                       AI Agent

                Claude Code / IDE Agent

                            |

                           MCP

                            |

                      PerfLens Skill

                            |

                    PerfLens Server

                            |

        +-------------------+-------------------+

        |                                       |

 Runtime Collector                       Automation


        |                                       |

 H5 Runtime                            Playwright/CDP

 微信 Runtime                          微信自动化SDK

 支付宝 Runtime                       人工真机操作


        |

        |

              Performance Session


        |

        |

             Dashboard


```


---

# 6. Monorepo设计


采用 pnpm workspace。


目录：

```
perflens/


packages/


├── core

│
│   核心数据模型
│   Event协议
│   Session模型


├── server

│
│   本地服务
│   WebSocket
│   数据管理


├── dashboard

│
│   React Dashboard


├── adapter-h5

│
│   H5采集


├── adapter-wechat

│
│   微信小程序


├── adapter-alipay

│
│   支付宝小程序


├── runtime


│
├── runtime-wechat
│
└── runtime-alipay


├── analyzer


│
├── web-analyzer
│
└── miniapp-analyzer


├── skill


├── mcp-server


├── report


└── export


examples/


├── h5-demo

├── wechat-demo

└── alipay-demo

```


---

# 7. 技术选型


## Dashboard

推荐：

- React
- Vite
- TypeScript
- Tailwind
- ECharts


负责：

- Timeline
- Charts
- AI分析展示
- 导出


---

## Server

MVP：

- Node.js
- Fastify
- WebSocket
- SQLite


后续：

- PostgreSQL
- ClickHouse


原因：

性能事件属于：

- 时间序列
- 大量Event
- Session查询


---

## MCP

TypeScript MCP SDK。


提供：

- Session管理
- 数据查询
- AI分析
- 自动化调用


---

## Runtime SDK


要求：

- TypeScript
- 轻量
- 零业务依赖
- 可Tree Shaking


---

# 8. Performance Session


所有性能分析基于 Session。


原因：

避免：

Dashboard人工操作

和

Agent自动分析

互相干扰。


---

Session模型：

```json
{
 id:"session-001",

 platform:"alipay",

 source:"agent",

 project:"demo",

 startTime:123456,

 events:[]
}
```


---

# 9. Session来源


## 9.1 Agent Session


来源：

MCP。


流程：

```
用户请求分析

↓

创建Session

↓

自动化执行

↓

采集数据

↓

AI分析
```


适合：

- H5
- 微信


---

## 9.2 Developer Session


来源：

Dashboard。


流程：

```
启动录制

↓

开发者操作

↓

Runtime采集

↓

AI分析
```


适合：

支付宝真机。


---

## 9.3 Runtime Session


来源：

SDK连接。


例如：

```
小程序启动

↓

Runtime SDK

↓

WebSocket

↓

PerfLens Server
```


---

# 10. Event模型


所有平台统一：

```typescript

interface PerformanceEvent {


 id:string


 sessionId:string


 platform:string


 type:string


 timestamp:number


 duration?:number


 metadata:any


}

```


事件类型：


```
lifecycle

network

resource

render

bridge

setData

component

canvas

package

plugin

error

```


---

# 11. 数据流


```
Runtime

↓

Event

↓

Server

↓

Session Storage

↓

Analyzer

↓

Dashboard


```


---

# 12. AI分析输入


AI分析使用：

```
Performance Events

+

项目源码

+

项目结构

+

配置文件

+

性能规则库

```


输出：

```
问题

根因

源码位置

优化建议

风险等级

```

# 13. Adapter设计


Adapter负责平台差异。


统一接口：

```
Adapter

├── Collector

├── Capability

├── Analyzer Rule

└── Export Data

```


Capability：

```typescript

interface Capability {

 lifecycle:boolean

 network:boolean

 automation:boolean

 runtime:boolean

 screenshot:boolean

 recording:boolean

}

```


---

# 14. H5 Adapter


## 技术方案


采用：

- Playwright
- Chrome DevTools Protocol


安装：

```
playwright
```


---

## 自动化能力


支持：

- 打开页面
- 用户流程模拟
- 页面交互
- 性能测试


例如：

```
打开首页

点击商品

进入详情

提交订单

```


---

## Performance API


采集：

### Navigation Timing

- DNS
- TCP
- TTFB
- DOM Ready
- Load


### Resource Timing

- JS
- CSS
- Image
- API


### Web Vitals

- LCP
- CLS
- INP
- FCP


---

## Runtime分析


采集：

- JS Error
- Promise Error
- Long Task
- Memory


---

## 扩展能力


后续支持：

- rrweb录屏
- 用户行为回放
- DOM分析


---

# 15. 微信小程序 Adapter


微信小程序能力较完整。


利用：

- 微信自动化 SDK
- 微信性能 API


---

# 15.1 自动化


支持：

- 启动小程序
- 页面跳转
- 点击操作
- 流程复现


适合：

Agent模式。


例如：

```
打开首页

↓

点击商品

↓

进入详情页

↓

采集性能

```


---

# 15.2 生命周期采集


App：

```
onLaunch

onShow

onHide
```


Page：

```
onLoad

onShow

onReady

onHide

onUnload
```


Component：

```
created

attached

ready

detached
```


---

# 15.3 性能API


使用：

- wx.getPerformance
- wx.reportPerformance


采集：

- 启动耗时
- 页面耗时
- 渲染耗时


---

# 16. 支付宝小程序 Adapter


## 定位


支付宝是 PerfLens 的核心竞争力。


原因：

- 缺少自动化能力
- 企业应用大量使用
- Runtime与Web差异大


方案：

```
Runtime SDK

+

真机操作

+

AI源码分析

```


---

# 17. 小程序 Runtime SDK


SDK目标：

提供开发调试阶段的 Runtime Instrumentation。


不是传统线上 APM。


---

# 17.1 初始化


例如：

```javascript

import '@perflens/alipay/runtime'


PerfLens.init({

 server:"ws://localhost:3000",

 debug:true

})

```


---

# 17.2 自动采集


无需业务修改。


包括：


## App生命周期

```
onLaunch

onShow

```


---

## Page生命周期

```
onLoad

onShow

onReady

onHide

onUnload

```


---

## Component生命周期


```
created

attached

ready

```


---

## 网络


包装：

```
my.request

```


采集：

- URL
- duration
- status
- size


---

## setData


包装：

```
this.setData()

```


采集：

- 调用次数
- 数据大小
- 更新字段
- 调用频率


---

# 18. Hook机制


SDK采用：

Proxy Wrapper。


不是覆盖业务代码。


例如：

业务：

```javascript

Page({

 onLoad(){

 }

})

```


SDK：

```
Page

↓

Wrapper

↓

Original Page

```


执行：

```
PerfLens记录

↓

业务生命周期

↓

PerfLens结束记录

```


---

# 19. 手动业务埋点


自动采集无法理解业务。


提供：

```javascript

PerfLens.mark(
 "product_ready"
)

```


适用于：


- 首屏完成
- 商品可购买
- 页面可交互
- 关键流程完成


---

# 20. 小程序专项分析体系


## 20.1 setData分析


小程序核心问题。


模型：

```
JS Runtime

↓

setData

↓

Bridge

↓

Native Render

```


---

分析：

### 大数据量


例如：

```
setData 500KB
```


建议：

- 减少字段
- 拆分数据


---

### 高频更新


例如：

```
1秒50次setData
```


建议：

- 合并更新
- 降低刷新频率


---

### 无效更新


结合源码：

发现：

```
更新整个list对象

实际只使用name字段

```


---

# 20.2 分包分析


读取：

```
app.json

page.json

package.json

```


分析：

- 主包大小
- 分包大小
- 页面依赖
- 重复依赖


输出：

例如：

```
主包过大

原因：

components/editor

占用800KB


建议：

迁移subpackage

```


注意：

只生成建议。

不自动修改。


---

# 20.3 分包预加载分析


结合用户路径：

```
首页

↓

商品

↓

订单

```


发现：

```
进入订单首次下载资源

```


建议：

增加：

```
preloadRule
```


---

# 20.4 组件分析


分析：

- 初始化时间
- ready时间
- 重复创建


例如：

```
Recommend组件

ready:

900ms

```


建议：

- 延迟加载
- 懒初始化


---

# 20.5 Canvas分析


重点关注：

小程序Canvas容易产生性能问题。


采集：

- draw次数
- FPS
- 内存
- 更新频率


分析：

例如：

```
循环draw导致FPS下降
```


建议：

- 批量绘制
- 离屏Canvas


---

# 21. AI Analyzer


## 输入


```
Performance Session

+

Source Code

+

Project Structure

+

Rule Engine Result

```


---

## 输出结构


```json

{

 issue:"首页加载慢",

 severity:"high",

 location:"pages/home/index.js",

 reason:"onShow同步请求多个接口",

 suggestion:"拆分首屏逻辑",

 risk:"low"

}

```


---

# 22. Source Analyzer


读取：


## 页面


```
pages/

```


分析：

- 生命周期
- 数据流


---

## Component


分析：

- 初始化
- 依赖


---

## 配置


读取：

```
app.json

project.config.json

package.json

```


---

# 23. Performance Rule Engine


规则独立。


目录：

```
rules/

├── lifecycle

├── setData

├── network

├── package

├── canvas

└── component

```


规则：

```json

{

id:"SETDATA_BIG",

condition:"size>200KB",

level:"warning",

suggestion:"reduce payload"

}

```


---

# 24. Skill设计


Skill负责：

流程编排。


例如：

用户：

```
分析支付宝首页性能
```


执行：


```
启动Server

↓

创建Session

↓

等待SDK连接

↓

采集数据

↓

读取源码

↓

AI分析

↓

Dashboard展示

```


---

# 25. MCP设计


MCP提供Agent能力。


## Session


```
create_session

get_session

close_session

```


---

## 数据


```
query_events

get_timeline

get_metrics

```


---

## 分析


```
analyze_session

find_performance_issue

```


---

## 自动化


```
run_flow

```


例如：

```
open page

click button

wait

collect

```


---

# 26. Dashboard设计


定位：

AI增强版DevTools。


---

## Overview


展示：

- 启动时间
- 页面耗时
- 网络
- setData
- Canvas
- FPS


---

## Timeline


类似：

Chrome Performance。


展示：

```
Lifecycle

Network

Bridge

Render

```


---

## AI Panel


实时：

```
发现问题：

setData过大

位置：

pages/home/index.js

建议：

拆分数据

```


---

# 27. Dashboard交互原则


Dashboard负责：

- 查看
- 筛选
- 标记
- 分析
- 导出


不负责：

复杂自动化。


自动化由：

- Agent
- Playwright
- 微信自动化


负责。


---

# 28. 导出能力


支持：


## HTML Report


生成：

```
performance-report.html
```


包含：

- Summary
- Timeline
- Charts
- AI分析
- 优化建议


可独立打开。


---

## JSON


用于：

二次分析。


---

## CSV


用于：

Excel分析。


字段：

```
timestamp

type

page

duration

size

metadata

```


---

## Excel


生成：

```
performance-report.xlsx
```


Sheet：


Summary

Timeline

Network

Bridge

Issues


---

# 29. 数据存储


MVP：

SQLite。


原因：

- 本地运行
- 简单
- 支持查询


未来：

ClickHouse。


适合：

- 大量Event
- 时间序列
- 长期趋势


---

# 30. 安全设计


默认：

本地分析。


不上传：

- 源码
- Token
- Cookie


网络数据脱敏：

过滤：

```
Authorization

Cookie

Token

Password

```


---

# 31. 开发路线


## Phase 0 基础设施


完成：

- Monorepo
- Core
- Event Schema
- Server


---

## Phase 1 Dashboard MVP


完成：

- WebSocket
- Timeline
- Session
- Export


---

## Phase 2 H5


完成：

- Playwright
- CDP
- WebVitals


---

## Phase 3 微信


完成：

- 自动化
- 生命周期
- Performance API


---

## Phase 4 支付宝


重点：

- Runtime SDK
- Hook
- setData分析
- request分析
- 真机采集


---

## Phase 5 AI能力


完成：

- Rule Engine
- Source Analyzer
- MCP
- Skill


---

# 32. 长期方向


未来支持：

- Flutter
- React Native
- Native App


通过：

统一 Event Model。


---

# 总结


PerfLens：

> AI增强版 DevTools + MiniApp Runtime Performance Platform


核心能力：

1. Runtime采集

2. AI源码分析

3. 小程序专项性能模型

4. MCP Agent能力

5. 实时Dashboard

6. HTML/CSV/Excel报告


核心壁垒：

支付宝小程序 Runtime 性能分析。
```

这版作为 `plan.md` 已经可以作为项目总纲。

后续让 Claude Code 拆分时，我建议拆成：

```
docs/

├── architecture.md
├── event-schema.md
├── runtime-sdk.md
├── miniapp-performance-rules.md
├── mcp-design.md
├── dashboard-design.md
└── roadmap.md
```

其中最值得优先细化的是：

1. `event-schema.md`（决定整个系统能不能扩展）
2. `runtime-sdk.md`（支付宝壁垒）
3. `miniapp-performance-rules.md`（AI价值核心）

这三个确定后，代码实现会顺很多。
