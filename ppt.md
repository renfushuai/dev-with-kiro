

---

## 封面

🤖
Team Sharing · 2026
Kiro + Skill
Java 后端开发实践分享
从 Skill 深度使用到 AI 辅助后端开发落地
结合 ai-crm 真实项目案例

🎯 寄语：
AI 进化速度太快，所有经验都只是暂时的
一切需要自己和 AI 磨合探索

Kiro IDE
Skills 生态
Java / Spring Boot
Spec 驱动开发

---

## 议程

Agenda
今天讲什么
三个板块
🧩
Part 1
Skill 快速回顾
简易温习核心概念与配置
⚡
Part 2
Skill 深度使用
Steering 文件 Spec 驱动开发 Hooks 自动化
☕
Part 3
Java 后端落地
ai-crm 真实案例 积分模块全流程 团队协作规范

---

## Part 1 分隔页

01
Part 1
Skill 快速回顾
简短温习 Skill 核心机制，我们快速过一遍

---

## Skill 快速温习

Part 1
温故知新：核心概念与最佳实践

什么是 Skill？
它是可安装的 AI 能力扩展包（SKILL.md），提供即插即用的模块化知识。

常用命令速记：
搜索：`npx skills find java`
安装到工作区：`npx skills add owner/repo@skill-name`
全局安装（个人工具）：`npx skills add owner/repo@skill-name -g`

🏅 **团队最佳实践复习**
业务和项目规范类的 Skill，务必要安装在工作区（`.agents/skills/`）并提交到 git！这样能保证全团队使用这套规范的拉齐。

---

## Part 2 分隔页

02
Part 2
Skill 深度使用
Steering · Spec 驱动开发 · Hooks 自动化

---

## Steering 文件

Part 2 · Steering
Steering：让 AI 了解你的项目
Steering 文件是放在  .kiro/steering/  下的 Markdown，
每次对话都会自动注入给 Kiro，相当于给 AI 的 永久上下文 。
📌
always（默认）
每次对话都注入，适合项目规范、架构说明
📂
fileMatch
匹配特定文件时注入，如编辑 *.sql 时注入 DB 规范
🔑
manual
用 # 手动引用，按需加载，节省上下文
你们项目的 Steering
# .kiro/steering/structure.md
---
inclusion: always
---
# 项目结构规范
- 所有枚举必须实现 BaseEnum
- InputDto 必须以 InputDto 结尾
- 所有表必须有 appid 字段（多租户）
- Session 上下文从 SessionHolder 读取
- 禁止在 Service 中写参数校验逻辑
💡 有了 Steering，Kiro 生成的每一行代码都自动遵守团队规范，
不需要每次对话都重复说明约定

---

## Steering 实战效果

Part 2 · Steering
Steering 实战：规范自动落地
❌ 没有 Steering
// AI 可能生成这样的代码
public class   UserDto  {
private  Integer userId;   // 没有 @ApiModelProperty
private  String name;      // 没有校验注解
}
public enum   StatusEnum  {   // 没实现 BaseEnum
ACTIVE, INACTIVE
}
// Service 里写了参数校验
if  (dto.getUserId() ==  null ) {
throw new   Exception ( "userId不能为空" );
}
✅ 有 Steering 自动生成
// Kiro 自动遵守所有规范
@Data
public class   UserInputDto  {
@ApiModelProperty ( "应用ID" )
@NotNull (message =  "appId不能为空" )
private  Integer appid;
@ApiModelProperty ( "用户ID" )
@NotNull (message =  "userId不能为空" )
private  Long userId;
}
public enum   StatusEnum   implements   BaseEnum  {
ACTIVE( 1 ,  "启用" ), INACTIVE( 0 ,  "禁用" );
// getValue() + getDesc() 自动实现
}

---

## Steering 实战对比 例2 枚举+Controller

Part 2 · Steering 实战
Steering 实战：枚举 &amp; Controller
❌ 没有 Steering
// 枚举：没实现 BaseEnum，用裸数字
public enum   WorkOrderStatusEnum  {
PENDING, PROCESSING, DONE
}
// Service 里直接比较数字
if  (workOrder.getStatus() ==  1 ) {
// 1 是什么？没人知道
}
// InputDto：枚举字段没有 @CheckEnum
@Data
public class   WorkOrderQueryInputDto  {
// 传入 status=999 也不会报错！
private  Integer status;
private  Integer workOrderType;
}
传入非法枚举值 status=999，Service 层拿到脏数据，查询结果为空却不报错，排查困难
✅ 有 Steering 自动生成
// 枚举：自动实现 BaseEnum
public enum   WorkOrderStatusEnum   implements   BaseEnum  {
PENDING( 0 ,  "待处理" ), PROCESSING( 1 ,  "处理中" ),
DONE( 2 ,  "已完成" );
}
// InputDto：枚举字段自动加 @CheckEnum
@Data
public class   WorkOrderQueryInputDto  {
@ApiModelProperty ( "工单状态，参见 WorkOrderStatusEnum" )
@CheckEnum (value = WorkOrderStatusEnum. class ,
message =  "工单状态不合法" )
private  Integer status;
@ApiModelProperty ( "工单类型，参见 WorkOrderTypeEnum" )
@CheckEnum (value = WorkOrderTypeEnum. class ,
message =  "工单类型不合法" )
private  Integer workOrderType;
}
传入 status=999 → 接口层直接返回参数错误，非法值在入口被拦截，Service 永远拿到合法数据

---



## Spec 驱动开发概念

Part 2 · Spec
Spec 驱动开发
从需求到代码的结构化工作流
📋
requirements.md
需求文档
🏗️
design.md
技术设计
✅
tasks.md
实现任务
⚡
Execute
自动执行
📋 requirements.md
用户故事 + 验收标准，用自然语言描述"做什么"，Kiro 帮你整理成结构化需求
🏗️ design.md
架构设计、数据模型、接口定义、正确性属性，Kiro 根据需求自动生成技术方案
✅ tasks.md
可执行的任务清单，每个任务对应具体代码文件，支持一键"Run All Tasks"
💡 Spec 文件存放在  .kiro/specs/{feature-name}/ ，提交 git 后全团队可见，是需求 + 设计 + 实现的完整记录

---

## Hooks 自动化

Part 2 · Hooks
Hooks：事件驱动的自动化
Hook 在 IDE 事件发生时自动触发 AI 动作， 把重复的检查工作变成 自动流程 。
常用触发事件
事件
触发时机
fileEdited
保存文件时
preToolUse
AI 执行工具前
postTaskExecution
Spec 任务完成后
userTriggered
手动点击触发
agentStop
AI 执行结束后
实用 Hook 示例
// 保存 Java 文件时自动 Code Review
{
"name" :  "Java Code Review" ,
"version" :  "1.0.0" ,
"when" : {
"type" :  "fileEdited" ,
"patterns" : [ "**/*.java" ]
},
"then" : {
"type" :  "askAgent" ,
"prompt" :  "检查是否遵守团队规范：
1. 枚举是否实现 BaseEnum
2. InputDto 是否有 @ApiModelProperty
3. 是否有 appid 字段
4. Service 是否有参数校验逻辑（应移到 DTO）"
}
}

---

## Part 3 分隔页

03
Part 3
Java 后端落地实践
以 ai-crm 学员积分模块为例
完整演示 Spec 驱动开发全流程

---



## Step 1 - 写需求

Part 3 · Step 1
Step 1：用自然语言描述需求
告诉 Kiro 你要做什么，它帮你整理成
结构化需求文档
💬 你对 Kiro 说：
"给学员积分系统写个 spec，需要三个接口：
积分记录分页查询、增加积分、消耗积分。
积分变动要记录流水，并发安全要保证。"
✅ Kiro 生成：
4 个结构化需求（含用户故事）
每个需求的验收标准（WHEN/THEN 格式）
术语表（ExpUser、SubId 等）
数据一致性需求
生成的需求示例
## 需求 2：增加积分
用户故事： 作为销售顾问，我希望能为指定
学员增加积分，以便通过积分激励学员参与
课程活动。
验收标准：
1. WHEN 调用增加积分接口时，THE Controller
SHALL 接收 userId（必填）、points（最小1）、
remark（可选，最大200字）
2. WHEN 请求合法时，THE Service SHALL 使用
分布式锁（以 userId 为锁 key）保证并发安全
3. WHILE 持有锁，THE Service SHALL 查询或
初始化积分账户，将余额加上增量
4. WHILE 持有锁，THE Service SHALL 新增
changeType=INCREASE 的流水记录

---

## Step 2 - 技术设计

Part 3 · Step 2
Step 2：自动生成技术设计
design.md 包含什么
🏗️
架构分层图
Controller → Service → Mapper 调用链，AOP 横切关注点
📊
数据模型 + DDL
实体类定义、建表 SQL、索引设计，自动遵守 appid 多租户规范
🔄
业务流程伪代码
增加/消耗积分的完整执行步骤，含锁和事务边界
✅
正确性属性（6条）
形式化描述系统不变式，作为属性测试的依据
自动生成的实体类
@Data
@TableName ( "exp_user_points" )
public class   ExpUserPoints   extends   EntityBase  {
@TableId (type = IdType.AUTO)
private  Long id;
@ApiModelProperty ( "租户ID" )
private  Long subId;
@ApiModelProperty ( "学员ID" )
private  Long userId;
@ApiModelProperty ( "当前余额 = totalIncrease - totalConsume" )
private  Integer balance;
@ApiModelProperty ( "累计增加积分" )
private  Integer totalIncrease;
@ApiModelProperty ( "累计消耗积分" )
private  Integer totalConsume;
}

---

## Step 3 - 任务执行

Part 3 · Step 3
Step 3：任务清单 + 一键执行
tasks.md 结构
## Tasks
- [x] 1. 数据库 DDL 脚本
- 创建 exp_user_points 表
- 创建 exp_user_points_record 表
- [x] 2. 枚举与错误码
- [x] 2.1 PointsChangeTypeEnum
- [x] 2.2 ErrorCode.POINTS_INSUFFICIENT
- [x] 3. 实体类
- [x] 3.1 ExpUserPoints
- [x] 3.2 ExpUserPointsRecord
- [x] 7. Service 实现
- [x] 7.2 increase 方法
- [x] 7.4 consume 方法
- [ ]* 7.3 属性测试（可选）
- [x] 9. Controller 实现
执行方式
1
Run All Tasks
Kiro 按顺序执行所有任务，自动写代码、建文件
2
Execute Task N
单独执行某个任务，适合增量开发或重新生成
3
状态追踪
[ ] 未开始 → [-] 进行中 → [x] 完成，全程可见
标注  *  的任务是可选任务（如属性测试），
MVP 阶段可跳过，不影响主流程执行

---

## 核心代码展示 - Service

Part 3 · 代码展示
Kiro 生成的 Service 代码
增加积分（含分布式锁）
@RedissonLock (key =  "#dto.userId" ,
waitTime =  3 , leaseTime =  10 )
@Transactional (rollbackFor = Exception. class )
public void  increase( PointsIncreaseInputDto  dto) {
Long subId = SessionHolder.getCurrentSubId();
String staffId = SessionHolder.getCurrentStaffId();
// 查询或初始化账户
ExpUserPoints  account = getOne(
new   LambdaQueryWrapper &lt;&gt;()
.eq( ExpUserPoints ::getSubId, subId)
.eq( ExpUserPoints ::getUserId, dto.getUserId())
);
if  (account ==  null ) {
account =  new   ExpUserPoints ()
.setSubId(subId).setUserId(dto.getUserId())
.setBalance( 0 ).setTotalIncrease( 0 )
.setTotalConsume( 0 );
}
int  before = account.getBalance();
account.setBalance(before + dto.getPoints());
account.setTotalIncrease(
account.getTotalIncrease() + dto.getPoints());
saveOrUpdate(account);
// 写流水记录
recordMapper.insert( new   ExpUserPointsRecord ()
.setSubId(subId).setUserId(dto.getUserId())
.setChangeType( PointsChangeTypeEnum .INCREASE.getValue())
.setPoints(dto.getPoints())
.setBeforeBalance(before)
.setAfterBalance(account.getBalance())
.setStaffId(staffId).setRemark(dto.getRemark()));
}
代码亮点
🔒
@RedissonLock 在事务外加锁
AOP Order(0) 保证锁在事务外，防止锁释放后数据未提交
🏦
账户自动初始化
首次增加积分时自动创建账户，无需提前初始化
📝
流水记录完整
记录变动前后余额、操作人、备注，完整审计链路
🔄
SessionHolder 获取上下文
subId、staffId 从 ThreadLocal 读取，不作为接口参数

---

## 正确性属性

Part 3 · 正确性保障
正确性属性：形式化验证
Spec 驱动开发的核心亮点：
用 属性（Property） 描述系统不变式，
用 属性测试（PBT） 自动验证
积分模块的 6 条属性
属性1 非法参数（points≤0）必须被拒绝
属性2 查询结果必须按 userId+subId 过滤且倒序
属性3 增加后 balance == before + points
属性4 余额不足时消耗被拒绝，状态不变
属性5 消耗后 balance == before - points
属性6 始终满足 balance == totalIncrease - totalConsume
jqwik 属性测试示例
// Feature: student-points, Property 3
// 增加积分后账户状态正确
@Property (tries =  100 )
void  increaseUpdatesBalanceCorrectly(
@ForAll @Positive   int  initialBalance,
@ForAll @Positive   int  delta
) {
// 准备账户
ExpUserPoints  account = mockAccount(initialBalance);
PointsIncreaseInputDto  dto =  new   PointsIncreaseInputDto ();
dto.setUserId( 1L );
dto.setPoints(delta);
service.increase(dto);
ExpUserPoints  updated = getAccount( 1L );
// 属性3：余额正确
assertThat(updated.getBalance())
.isEqualTo(initialBalance + delta);
// 属性6：不变式
assertThat(updated.getBalance())
.isEqualTo(updated.getTotalIncrease()
- updated.getTotalConsume());
}

---

## 团队协作规范

Part 3 · 团队协作
团队协作：让 AI 成为团队成员
📁 提交到 Git 的文件
.kiro/
├── steering/
│   ├── product.md      ← 产品背景
│   ├── structure.md    ← 代码规范
│   └── tech.md         ← 技术栈
├── specs/
│   └── student-points/
│       ├── requirements.md
│       ├── design.md
│       └── tasks.md
└── skills/             ← 工作区 skill
全团队共享同一套 AI 行为规范
🔄 推荐工作流
1
新功能 → 先写 Spec
需求 → 设计 → 任务，对齐后再执行
2
Code Review → Hook 自动触发
保存文件时自动检查规范合规性
3
Steering 持续维护
发现新规范 → 更新 steering → 全团队生效
4
Skill 统一安装
工作区 skill 提交 git，新人 clone 即可用

---

## 效率对比

Part 3 · 效率提升
效率对比：传统 vs Kiro
环节
传统方式
Kiro + Spec
节省
需求整理
手写文档 2-4h
对话生成 15min
~85%
技术设计
画图写文档 4-8h
自动生成 20min
~90%
CRUD 代码
手写 Entity/DTO/Mapper 2-3h
任务执行 10min
~85%
规范检查
Code Review 人工检查
Hook 自动触发
自动化
新人上手
口口相传 1-2 周
Steering 即时生效
知识沉淀
核心价值：把重复的、规范性的工作交给 AI，让开发者专注在业务逻辑和架构决策上

---

## 落地建议

Part 3 · 落地建议
团队落地三步走
🌱
第一步
完善 Steering
整理现有代码规范
写入 structure.md
补充 tech.md 技术栈
提交 git 全团队共享
本周可完成
⚡
第二步
新功能用 Spec
下个迭代选一个功能
用 Spec 驱动开发
对比传统方式效率
收集团队反馈
下个迭代
🚀
第三步
Hook + Skill 扩展
配置 Code Review Hook
安装 unit-testing skill
自定义团队专属 skill
持续迭代优化
持续演进

---

## Part 4 分隔页

04
Part 4
大预言：OPC时代与新职业
下一个十年，属于一人公司（OPC）的时代，已经彻底来了。

---

## 独立开发者必备的“免费”工具栈

Part 4 · 宝藏工具分享
- Vercel ｜ 一键部署上线（免费）
- Namecheap ｜ 域名持有（~1美元/月）
- Cloudflare ｜ DNS 解析与 CDN（免费）

🛡 后端与中间件
- Supabase ｜ 强大的后端服务（免费）
- Clerk    ｜ 身份与用户认证（免费）
- Sentry   ｜ 线上错误监控（免费）
- Resend   ｜ 触发邮件服务（免费）

💾 数据库与扩展
- Upstash  ｜ Serverless Redis 缓存（免费）
- Pinecone ｜ AI 应用的向量数据库（免费）


💡 思考：当技术基建和代码生产趋近于“0门槛”，纯堆代码的 CRUD 价值被无限压缩，未来有什么新岗位等待着我们？

---

## 新职业 (上)：后端的进阶之路

Part 4 · 未来展望
2027-2030 后端工程师的演变与进阶

1. AI编排师（AI Orchestration Designer）
核心转变：从“微服务网关架构师”变成“AI任务流架构师”。
核心能力：将复杂业务拆解为 Spring Boot 服务与 AI Agent 的协作流。例如定义系统边界，编排传统 API 节点与 AI 分析节点何时通过异步 MQ 协同。这需要极强的服务端拆结构想力。

2. 上下文架构师（Context Architect）
核心转变：从“数据库模型设计”演变为“AI全局语境设计专家（DDD）”。
工作内容：设计系统“应当投喂给 AI 什么”。统筹 Steering 文件规范、Redis 缓存上下文、RAG 向量库与 OpenAPI 调用隔离。这要求极强的防数据越权与多租户隔离安全意识。

3. 架构规范与输出审计员（AI Output Auditor）
核心转变：从“写 CRUD 的初级码农”升级为“资深 Code Reviewer 和安全卫士”。
核心职责：当 AI 大量生产 Java 代码或 SQL 增量脚本时，审查其是否漏了分布式事务锁（如 `@RedissonLock`）、是否发生数据越权。这唯有真正踩过线上坑的后端专家才能高效把关。

---

## 新职业 (下)：后端的基建与治理

Part 4 · 未来展望
2027-2030 后端工程师的演变与进阶

4. Skill与插件开发者（Skill Developer）
核心转变：从“被动实现 API 的人”变为“AI 内部基建的建设者”。
工作内容：把公司现有复杂且庞杂的遗留 Java 服务，用 OpenAPI 或 `.agents/skills` 封装成标准能力集，为团队沉淀研发套件库，享受内部基座开发的巨大红利。

5. AI工程化布道师（Human-AI Collaboration Trainer）
核心转变：从研发骨干成长为重塑组内“研发效能与工作流”的布道者。
职责：主导产研协作模式转型。例如推进 Spec 驱动开发，告诉团队哪些系统边界放心交给 AI 完成，哪些核心资产链路（如资金清结算）必须由有经验的人类强管控。

6. 数据追溯与治理调解员（AI Ethics Mediator）
核心转变：在 AI“黑盒调用”常态化下，成为负责兜底追踪与数据血缘的人。
工作内容：当 AI 自动决策链产生难以理解的脏数据（甚至引发线上事故）时，能从海量的日志（Log、RocketMQ 轨迹）中剥离业务幻觉与逻辑漏洞，制定针对性的补偿脚本。

---

## 结束页

🎯
Thanks
让 AI 成为你的
最强队友
Steering 沉淀规范 · Spec 驱动开发 · Skill 扩展能力 · Hook 自动化流程
🧩
skills.sh
Skill 生态
📋
.kiro/specs/
Spec 文件
📌
.kiro/steering/
团队规范
⚡
.kiro/hooks/
自动化
