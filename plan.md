我整理成一个偏工程落地的 `plan.md`，重点补充：

- pnpm monorepo 架构
- core + adapter 模式
- H5 / 微信 / 支付宝差异
- Skill + MCP
- AI 分析闭环
- 小程序 Runtime 特性
- setData / Bridge / Canvas / 分包 / 插件 / 懒加载分析
- SDK 与生产隔离
- 不自动修改生产代码的原则
- 后续路线

```md
# AI Frontend Performance Engineer Plan

## 1. 项目定位

AI Frontend Performance Engineer 是一个面向企业前端团队的 AI 性能分析与优化 Agent。

目标不是替代传统 APM，而是利用 AI 结合：

- 运行时性能数据
- 自动化测试能力
- 项目源码分析
- 前端架构理解

自动定位性能问题，并生成可执行的优化建议。

核心理念：

> 从“发现性能指标异常”升级到“理解为什么慢，并给出架构级优化建议”。

---

# 2. 产品目标

## 2.1 支持平台

第一阶段：

- H5
- 微信小程序

重点建设：

- 支付宝小程序


原因：

企业业务大量运行在支付宝小程序，而支付宝缺少成熟自动化分析生态，需要提供 Runtime SDK 能力。


---

# 3. 总体架构

```
                        AI Agent

                 Claude Code / IDE Agent

                          |
                          |
                         MCP

                          |
                          |

              Frontend Performance Skill

                          |

        +-----------------+----------------+

        |                                  |

 Performance Analyzer              Report Generator


        |

        |

 Runtime Adapter Layer


 +-------------+--------------+-------------+

 |             |              |

 H5        微信小程序      支付宝小程序


Playwright   自动化SDK      Runtime SDK

CDP          wx API        my.performance


        |

        |

 Unified Performance Event Model


        |

        |

 Source Analyzer

        |

        |

 HTML Performance Report

```


---

# 4. Monorepo设计

采用 pnpm workspace。

目录：

```
ai-performance/

├── package.json
├── pnpm-workspace.yaml
├── turbo.json


├── packages/

│
├── core/
│
├── adapter-h5/
│
├── adapter-wechat/
│
├── adapter-alipay/
│
├── analyzers/
│
├── skill/
│
├── mcp-server/
│
├── report/
│
└── runtime-sdk/


├── examples/

├── h5-demo

├── wechat-demo

└── alipay-demo

```


---

# 5. Core设计


Core 不关心具体平台。

负责：

- 数据模型
- 事件协议
- Collector接口
- Analyzer接口
- Report接口


## Performance Event Model


统一：

```typescript
interface PerformanceEvent {

 type:
  | lifecycle
  | network
  | render
  | resource
  | error
  | bridge
  | component
  | package
  | plugin


 timestamp:number

 duration?:number

 metadata:any

}
```


不同平台转换：

```
H5

Performance API

      |

Performance Event



微信

wx.getPerformance

      |

Performance Event



支付宝

my.performance

      |

Performance Event

```


---

# 6. Adapter设计

不同平台能力不同。

不强行统一。


定义：

```typescript
interface AdapterCapability {

 lifecycle:boolean

 network:boolean

 screenshot:boolean

 recording:boolean

 sourceMap:boolean

 automation:boolean

}

```


例如：

## H5

```
{
 lifecycle:true,
 network:true,
 screenshot:true,
 recording:true,
 automation:true
}

```


## 微信

```
{
 lifecycle:true,
 network:true,
 automation:true
}

```


## 支付宝

```
{
 lifecycle:true,
 performance:true,
 runtime:true
}

```


---

# 7. H5 Adapter


## 技术方案

Playwright + CDP。


安装：

```
pnpm add playwright
```


能力：

## 页面自动化

- 打开页面
- 模拟用户操作
- 页面流程执行


## Performance API

采集：

- Navigation Timing
- Resource Timing
- Web Vitals


指标：

- FCP
- LCP
- CLS
- INP
- TTFB


## 网络分析

采集：

- 请求耗时
- 资源大小
- API错误


## Runtime

采集：

- JS Error
- Promise Error
- Long Task
- Memory


## 后续

支持：

- rrweb录屏
- 用户行为回放


---

# 8. 微信小程序 Adapter


利用官方能力：

- 微信自动化 SDK
- 微信性能 API


## 自动化

支持：

- 启动开发者工具
- 编译项目
- 页面操作
- 流程执行


## 性能采集


### App生命周期

采集：

- App Launch
- App Show


### Page生命周期

采集：

- onLoad
- onShow
- onReady
- onHide
- onUnload


### 页面跳转

分析：

- navigateTo
- redirectTo
- switchTab


### 性能API

利用：

- wx.getPerformance
- wx.reportPerformance


---

# 9. 支付宝小程序 Adapter（重点）


## 背景

支付宝小程序：

- 缺少微信类似自动化能力
- my.performance只能提供底层指标
- 需要运行时采集能力


因此设计：

支付宝 Runtime SDK。


---

# 10. Runtime SDK设计


目标：

不是长期埋点 SDK。

而是：

> 调试分析和灰度采集 SDK。


初始化：

```javascript
AIPerf.init({
 debug:true
})

```


自动采集：

## App

- onLaunch
- onShow


## Page

- onLoad
- onShow
- onReady
- onHide


## Performance

封装：

my.performance


采集：

- 页面耗时
- 启动耗时
- 渲染耗时


## Session

记录：

```
首页

↓

商品

↓

订单

↓

支付

```


形成：

Session Timeline。


---

# 11. 小程序专项分析能力


小程序不是H5。


原因：

- 无DOM
- 无BOM
- JS Runtime不同
- 双进程渲染
- JS-Native通信


因此需要独立 MiniApp Analyzer。


---

# 12. 小程序生命周期分析


分析：

```
App Launch

↓

Page Load

↓

Page Show

↓

Page Ready

↓

用户可交互

```


发现：

例如：

```
onShow:

1500ms

原因：

同步初始化:

用户信息

优惠查询

推荐接口

```


建议：

拆分首屏逻辑。


---

# 13. setData / Bridge分析


核心能力。


小程序：

```
JS

|

setData

|

Bridge

|

Native Render

```


## 采集：

- setData次数
- payload大小
- 更新字段
- 调用频率


发现：

## 数据过大

例如：

```
setData:

850KB

```


建议：

- 减少字段
- 分批更新


---

## 高频调用


发现：

```
1秒:

50次setData

```


建议：

合并更新。


---

## 无效更新


分析：

setData：

```
user对象 200KB

```


页面：

只使用：

```
user.name

```


建议：

精确更新字段。


---

# 14. Canvas分析


重点：

小程序Canvas容易出现性能问题。


采集：

- draw次数
- draw耗时
- FPS变化
- 内存变化


分析：

例如：

源码：

```
for(){

canvas.draw()

}

```


建议：

- 批量绘制
- 降低刷新频率
- 离屏Canvas


---

# 15. 分包分析


静态分析：

读取：

- app.json
- 页面配置
- 组件依赖


分析：

## 主包过大


输出：

```
主包:

1.8MB


主要来源:

order页面

editor组件

```


建议：

迁移分包。


注意：

只生成方案。

不自动修改。


---

# 16. 分包预下载分析


结合用户路径：

例如：

```
首页

↓

商品详情

↓

订单

```


发现：

订单分包首次进入加载。


建议：

增加 preloadRule。


---

# 17. 小程序插件分析


分析：

- plugin.json
- 插件加载耗时


采集：

```
plugin_start

plugin_ready

```


发现：

地图插件：

800ms。


建议：

- 延迟加载
- 空闲预加载


---

# 18. 组件懒加载分析


分析：

首页组件：

```
Banner

Goods

Coupon

Comment

Recommend

```


发现：

首屏不可见组件同步初始化。


建议：

- 异步组件
- 可见区域加载
- 延迟初始化


---

# 19. AI分析能力


## 指标分析

输入：

```
LCP 3s
```

输出：

```
banner图片过大

源码:

HomeBanner.vue
```


---

## 源码关联

关联：

```
性能事件

↓

源码位置

↓

优化建议

```


---

## 架构建议


例如：

```
主包过大

建议迁移order模块

风险:

高

影响:

路由
分享
权限

```


---

# 20. 不自动修改代码原则


小程序：

禁止自动修改：

- 分包
- 路由
- 插件
- 页面结构


原因：

生产风险高。


输出：

```
优化方案

+

影响范围

+

风险等级

+

实施步骤

```


---

# 21. SDK生产隔离


原则：

性能分析工具不能影响生产性能。


方案：


## 环境变量


开发：

```
PERF_ENABLE=true

```


生产：

```
false

```


---

## Tree Shaking


生产删除。


---

## Runtime Stub


开发：

```
真实SDK

```


生产：

```
空实现

```


---

## 灰度

支持：

- 指定用户
- 指定版本
- 指定设备


---

# 22. Skill设计


Skill负责：

任务编排。


例如：

用户：

```
分析支付宝小程序首页性能
```


Skill：

```
选择alipay adapter

↓

采集session

↓

读取源码

↓

AI分析

↓

生成报告

```


---

# 23. MCP接口


## analyze_performance


输入：

```json
{
 platform:"alipay",
 target:"home"
}

```


返回：

```
Performance Report
```


---

## analyze_architecture


分析：

- 分包
- 组件
- 插件
- 包结构


---

## generate_report


生成：

```
performance-report.html
```


---

# 24. HTML Report


包含：

## 性能总览


## 时间线


## 问题列表


## 源码定位


## 优化建议


## 风险评估


---

# 25. 开发路线


## Phase 1

MVP

支持：

- core
- H5 Adapter
- 微信 Adapter
- Skill
- MCP
- HTML Report


验证：

AI分析能力。


---

## Phase 2

支付宝重点建设：

- Runtime SDK
- Session采集
- MiniApp Analyzer


---

## Phase 3

企业能力：

- Web Console
- 性能趋势
- 版本对比
- 灰度分析


---

## Phase 4

AI优化闭环：

```
发现问题

↓

定位源码

↓

生成Patch建议

↓

人工确认

↓

验证效果

```


---

# 总结

项目不是传统性能监控工具。

定位：

> AI 驱动的前端性能工程平台。

核心竞争力：

1. AI理解源码和业务
2. 多平台Adapter架构
3. 小程序Runtime深度分析
4. 支付宝小程序性能分析能力

其中：

- H5：Playwright + CDP
- 微信：自动化SDK + 性能API
- 支付宝：Runtime SDK + AI分析

最终目标：

让 AI 成为一个懂 Web、小程序 Runtime 和前端架构的性能工程师。
```

这版已经可以作为后续 Claude Code / Cursor / Teamlink 开发的 `plan.md` 基础。你后续如果继续细化，我建议下一步拆两个文档：

1. `architecture.md`：详细定义 monorepo、接口、Event Model  
2. `miniapp-performance-rules.md`：沉淀微信/支付宝小程序性能规则库（这个会是核心壁垒）
