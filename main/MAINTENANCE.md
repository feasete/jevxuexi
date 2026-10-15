# 笔记本维护与验证

阅读入口和环境配置见 [README](README.md)，制作规范见 [项目手册](../AGENT.md)。
本页集中记录生成、检查和待完成的真实实验，避免在章节目录中重复说明。

## 重新生成

生成器位于 `generators/`，输出路径基于脚本位置，可以从任意工作目录调用。
以下命令以 `main/` 为工作目录，使用 README 中配置的环境：

```bash
.venv/bin/python generators/build_introduction_notebook.py # 单章示例（合并版介绍章）
.venv/bin/python generators/build_foundations_notebooks.py # 入门与概念七章
```

| 生成器 | 产出 |
|---|---|
| [build_introduction_notebook.py](generators/build_introduction_notebook.py) | 简介 |
| [build_system_one_notebook.py](generators/build_system_one_notebook.py) | System One |
| [build_state_notebook.py](generators/build_state_notebook.py) | 状态 |
| [build_primitives_notebook.py](generators/build_primitives_notebook.py) | 原语 |
| [build_confidence_notebook.py](generators/build_confidence_notebook.py) | 置信度 |
| [build_patterns_notebook.py](generators/build_patterns_notebook.py) | 架构模式 |
| [build_build_with_typesafe_notebook.py](generators/build_build_with_typesafe_notebook.py) | 应用构建 |
| [build_pi_jev_notebook.py](generators/build_pi_jev_notebook.py) | Pi + Jev 集成 |
| [build_cookbook_notebooks.py](generators/build_cookbook_notebooks.py) | `04_实战指南/` 前 8 篇（含知识补充注入；后 10 篇独立成稿于同目录） |
| 第十章 `10_本地模型/` | 原 `laya/` 整目录迁入（benchmark 即第六章配套）；生成数据 `**/data_generation/generated/` 已忽略；原模型总览文档改名 `模型介绍与对比.md` |
| 第十一章 `11_知识库/` | jev-cookbook 全量快照（28MB，来源私有仓库）；更新时从 datawhalechina/jev-cookbook 重新导出 |
| 第九章 `09_Agent集成/` | Pi 集成（含 pi_jev_demo/ 配套）与 DSH 决策协作自根目录迁入；DSH 相对链接已加深一级（../../apps） |
| 第八章 `08_前沿研究/` | jev_mem（真实 API 实测快照，输出勿手改）与 jev_harness（社区评估文档）自 notebooks 根目录迁入；各自 README 为准 |
| 第七章 `07_实战应用/` | 外部 jev-games（lzdFeiFei，无 LICENSE——已署名收录）+ jev-playground 五项目的 vendored 快照；更新时以两个上游仓库为准重新同步 |
| 第六章 `06_模型评测/` | 全部测评内容集中地：PR #6 教程册 + benchmark（自 10_本地模型 迁入）+ 根 llm_eval 多供应商框架（fork 分支收录）——均 Micheal024 作品 |正文在 `10_本地模型/benchmark/main/laya_vs_jev.ipynb`，章节副本含系列头；`tasks/public_all.jsonl` 为本地再生数据（gitignore，脚本见 `06_模型评测/benchmark/scripts/`） |
| [build_foundations_notebooks.py](generators/build_foundations_notebooks.py) | 认识 Jev（合并四章）、System One、状态、应用构建 |

修改生成器 → 重新生成 → 执行验证，不直接修改 `.ipynb`。生成会覆盖已有输出，历史实验请先另存。
仅修改 Notebook 不需要重建文档站点。

## 入门与概念章节验证

以下执行器和检查覆盖上述七章。公共准备代码由 [notebook_support.py](generators/notebook_support.py) 和 [模板](generators/templates/tutorial.ipynb) 生成，每本 Notebook 均自带完整代码，可独立运行。
依赖版本范围保存在 [constraints-foundations.txt](generators/constraints-foundations.txt)，安装方式统一见 README。

### 离线检查

```bash
.venv/bin/python generators/execute_foundations_notebooks.py --mode offline
.venv/bin/python -m unittest discover -s tests -p 'test_foundations_notebooks.py' -v
```

执行器逐章启动全新内核，在空工作目录运行，并在离线模式中移除 API 密钥。
离线输出写入 [validation/offline_previews](validation/offline_previews/)，正式文件保留未执行代码。
执行记录见 [foundations-offline.json](validation/foundations-offline.json)。预览中的数值均为人工示例，不能证明模型表现。

已完成的本地检查：

- 七章共 176 个代码单元执行通过，未调用远程 API。
- 四项检查通过，覆盖 27 个业务请求的 SDK 序列化和响应解析、鉴权失败、服务端失败、结构及密钥导出检查。
- 应用构建实验的八条路径均被人工数据覆盖，真实分支覆盖仍待观察。

### 真实执行

先配置启动进程的 `TYPESAFE_API_KEY` 环境变量，然后选择单章试跑：

```bash
.venv/bin/python generators/execute_foundations_notebooks.py --mode live --chapter introduction
```

需要完整验收时，执行七章：

```bash
.venv/bin/python generators/execute_foundations_notebooks.py --mode live
```

一次完整运行包含 **27 次业务请求 + 7 次连通性请求**，不自动重试。全量运行会再次执行已试跑的章节。
默认模型配置为 `jev-1.13.0`，可通过 `TYPESAFE_DEFAULT_MODEL` 更换；以实际响应和末尾执行记录为准。

执行器默认 `live`；缺少密钥会在启动内核前停止，也不会自动读取 `.env`。
真实执行失败即停止，不能以离线答案填补。交互教学中的 `auto` 模式仅对缺密钥或 401 显式回退，不用于验收。

成功执行会将真实输出写回正式 Notebook，生成 `validation/foundations-live.json`；单章报告名带章节后缀。
执行器检查输出错误、调用来源和密钥泄漏，语义结论仍需逐格人工核对。

## 真实 API 验收记录

**当前七章均待真实 API 验收。** 离线执行完成不表示真实模型实验已通过。

| 章节 | 待核对的行为 | 当前记录 |
|---|---|---|
| 简介 | 三个维度与权重组合是否可解释 | 待真实运行 |
| 快速开始 | 三种原语与量表是否匹配 | 待真实运行 |
| AI 入门 | 明确请求、否定与模糊措辞的实际概率 | 待真实运行；三例不构成校准评测 |
| System One | 退款三问、组合路径、改 ID 的实际差值 | 待真实运行 |
| 状态 | 等事实格式对照；缺事实与有政策的区别 | 待真实运行 |
| 应用构建 | 八条路径、补充示例、工具单位漏检 | 待真实运行；不假设分支必被命中 |
| 应用场景 | 相关性排序、空结果出口、过度泛化论断 | 待真实运行 |

每个发现记录日期、实际模型、state、问题版本、输出、解释及是否要修改问题；各章末尾 JSON 保留实际调用来源。

若 `COVERAGE.missing` 仍有分支：先写下预期与理由，再追加几句试句，保留每次返回；仍未出现就记为“本次样本未观察到”。
修改措辞或阈值后需重新生成并执行。按返回结果挑选的试句不能再作为无偏准确率测试集；准确率、漏检和覆盖率需要独立标注的留出数据。

正式验收需同时满足：真实调用数为正、离线调用数和错误输出为零，输出与解释一致，公开文件不含密钥。

# Jev 本地化实验报告

**实验者**：yuxing  
**实验日期**：2026-10-15  
**实验环境**：WSL2 Ubuntu 22.04 + RTX 4060 Laptop 8GB  
**替代模型**：NeoHorse-Jev-4B（本地 4.2B 蒸馏模型，Q4_K_M 量化）

---

## 一、实验背景

本实验原计划使用 TypeSafe 官方云端 Jev API（`jev-1.13.0`）完成教程。由于 TypeSafe 注册在当前网络环境下持续返回 403，改用社区蒸馏模型 **NeoHorse-Jev-4B** 在本地完成实验。

**替代说明**：
- 三原语接口、原子问题纪律、概率校准的学习方法完全一致
- 具体概率数值与云端有偏差，简单场景方向一致，复杂场景可能分歧
- 本地模型校准程度未知，不可用于生产环境的高风险决策

**环境搭建关键节点**：
1. WSL2 + Ubuntu 22.04（D 盘，避免占用 C 盘）
2. CUDA 12.6 + PyTorch 2.8.0+cu126（RTX 4060 直通成功）
3. 编译 llama.cpp（`--cuda-arch 89`）+ NeoHorse runtime 桥接库
4. 编写 `local_jev.py` 兼容层，模拟 TypeSafe SDK 接口

---

## 二、第一章：认识 Jev

### 2.1 连通性测试

| 项目 | 结果 |
| :--- | :--- |
| 输入 | "你好" |
| 问题 | greeting: 是否在打招呼？ |
| 输出 | **noul = 0.9917** |
| 判断 | 准确识别打招呼 |

### 2.2 实验 A：一次调用，三种原语

**State**：用户连续三天无法连接 Stripe，订单持续丢失，请求立即处理。

| 原语 | 问题 | 本地输出 | 云端教程值 | 差异 |
| :--- | :--- | :--- | :--- | :--- |
| Noul | urgency | 0.9943 | 0.98 | +0.014 |
| Choice | intent | troubleshoot (0.9971) | troubleshoot (1.0) | -0.003 |
| Score | frustration | 1.9131 | 1.99 | -0.077 |

**结论**：本地模型与云端 Jev 在简单场景下判断方向完全一致，数值偏差在可接受范围内。

### 2.3 综合实验：赛事指挥台

| 事件 | 判断 | 关键指标 |
| :--- | :--- | :--- |
| runner_injury（跑者扭伤） | 医疗人员，**转人工** | priority 2.34, human 0.99 |
| water_station（纸杯用完） | 补给组 | priority 2.06, human 0.48 |
| shirt_pickup（T恤领取） | 自动回复 FAQ | priority 0.94, human 0.09 |

**结论**：三条全部判断正确，边界案例（human=0.48）也如实给出了"不确定"。

---

## 三、第二章：核心概念（五册）

### 3.1 第一册：System One

**State**：订单 #A1234，客户购买已 30 天，商品未拆封，要求全额退款。

| 答案 ID | 类型 | 输出 |
| :--- | :--- | :--- |
| can_refund | Noul | 0.8482 |
| reason_type | Choice | no_need (0.736) |
| urgency | Score | 1.2040 |

**发现**：`urgency` 分布呈双峰（`{0:0.44, 1:0.20, 2:0.07, 3:0.29}`），说明 4B 模型在"程度判断"上不够稳定。

### 3.2 第二册：状态

**表述变化实验**：

| 版本 | can_refund | 相对基线 |
| :--- | :--- | :--- |
| A 简洁 | 0.8372 | 基线 |
| B 详细（含流程描述） | **0.6569** | **-0.18** |
| C 否定表述 | 0.9088 | +0.07 |
| D 含泄漏字段（薅羊毛） | 0.9004 | +0.06 |

**发现**：
- 版本 B 加"通过客服渠道提交"后概率反而下降 0.18，说明**啰嗦表述让模型犹豫**
- 版本 D 含偏见信息，概率不降反升，说明**模型无法识别偏见**

**逐步补充事实**：

| 步骤 | can_refund |
| :--- | :--- |
| 只有一条 | 0.6485 |
| 加时长 | 0.9859 |
| 加状态 | 0.9960 |
| 加政策 | 0.9989 |

**发现**：事实越完整，判断越自信，**单调递增符合预期**。

### 3.3 第三册：原语

**State**：登录后页面空白，持续 2 小时，影响所有团队成员，无报错信息。

| 原语 | 输出 |
| :--- | :--- |
| is_blocking (Noul) | 0.9818 |
| category (Choice) | frontend (0.5707) |
| severity (Score) | 2.9987（峰值在 3，p=0.4939） |

**发现**：category 分布为 **frontend 0.57 / backend 0.27 / auth 0.10 / network 0.05**——模型知道是前端为主，但对"前端 vs 后端"边界**诚实地表达了不确定**。

### 3.4 第四册：置信度

| 案例 | argmax | p(max) | confidence | 分流 |
| :--- | :--- | :--- | :--- | :--- |
| "系统完全崩溃了" | 3 | 0.9210 | 0.9684 | 自动处理 |
| "嗯……可能有点小问题" | 0 | 0.8641 | 0.9372 | 自动处理 |

**发现**：第二个案例暴露问题——模糊措辞本应给出"接近均匀的分布"，但模型给了 0.864 的尖锐分布。**本地蒸馏模型倾向于"猜一个答案"，而非承认不确定**。

### 3.5 第五册：应用构建

| 工单 | 话题 | 紧急度 | 分流 |
| :--- | :--- | :--- | :--- |
| T-1001（付了钱显示未支付） | billing | 0.89 | 财务组（优先）✅ |
| T-1002（问企业版） | sales | 0.01 | 销售组 ✅ |
| T-1003（第三次没人回复） | complaint | 0.99 | 客户成功主管（优先）✅ |
| T-1004（退款多久到账） | info | 0.15 | 自动知识库 ✅ |

**结论**：四条全部正确，紧急度判断符合直觉。

---

## 四、状态白名单专项实验

### 4.1 偏见信息污染实验

**基线**：订单 30 天前购买，未拆封，要求退款。

| 场景 | can_refund | 相对基线 |
| :--- | :--- | :--- |
| 基线（仅事实） | 0.9870 | — |
| +中性历史（老客户） | 0.9905 | +0.0035 |
| **+偏见历史（薅羊毛）** | **0.9047** | **-0.0823** |
| **+偏见历史（仅事实）** | **0.9398** | **-0.0472** |
| +情绪描述 | 0.9926 | +0.0056 |
| +无关信息 | 0.9944 | +0.0073 |

**发现**：
- 判断性词汇（薅羊毛）影响 **-0.035**
- 纯事实（退过 3 次）影响 **-0.047**
- 两者**都在污染判断**，但机制不同

### 4.2 未来结果泄漏实验

**用政策边界场景（35/40/60 天）让基线落在合理区间**：

| 场景 | 基线 | +正向泄漏 | +负向泄漏 | Δ正向 | Δ负向 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **35 天** | 0.5997 | 0.7128 | 0.2424 | **+0.1131** | **-0.3573** |
| **40 天** | 0.1224 | 0.3138 | — | **+0.1914** | — |
| **60 天** | 0.0137 | 0.3229 | — | **+0.3091** | — |

**关键发现**：

1. **基线单调性完美**：35天(0.60) > 40天(0.12) > 60天(0.01)，说明模型在按政策推理
2. **泄漏效应随基线降低而放大**：
   - 基线 0.60 → +0.11（涨 18%）
   - 基线 0.12 → +0.19（涨 156%）
   - 基线 0.01 → +0.31（涨 **2300%**）
3. **负向泄漏影响更大**：35天负向 Δ=-0.36，是正向的 3 倍

**结论**：**只要 state 包含"未来结果"字段，模型就会直接读取它，而不是推理**。越是边缘案例，越容易被泄漏带偏。

---

## 五、核心发现总结

| 发现 | 证据 | 影响 |
| :--- | :--- | :--- |
| 事实越完整，判断越自信 | 0.65→0.99 单调递增 | ✅ 符合预期 |
| 啰嗦表述反而降置信 | 版本B比版本A低0.18 | ⚠️ 状态要简洁 |
| 模型无法识别偏见信息 | 加"薅羊毛"概率异常 | ⚠️ **必须做白名单** |
| 程度判断不稳定 | urgency 双峰分布 | ⚠️ 4B模型校准不足 |
| 模糊输入未必给低置信 | "可能有点小问题"给0.86 | ⚠️ 蒸馏模型倾向"猜答案" |
| 未来结果泄漏严重污染 | 60天案例涨23倍 | 🚨 **必须禁止** |

---

## 六、与云端 Jev 的对比

| 维度 | 云端 Jev | 本地 NeoHorse-Jev-4B |
| :--- | :--- | :--- |
| 简单场景判断 | ✅ 准确 | ✅ 方向一致 |
| 边界场景校准 | ✅ RLCD专门训练 | ⚠️ 偏差较大 |
| 模糊输入处理 | ✅ 给出平坦分布 | ⚠️ 倾向猜一个答案 |
| 泄漏敏感性 | 未知 | 🚨 极易被污染 |
| 推理速度 | ~0.65s | 本地 GPU 快速 |
| 成本 | $0.042/百万token | 免费 |

---

## 七、应用建议

基于以上发现，在本地部署 Jev 类决策模型时：

1. **严格状态白名单**：state 只允许包含作决定时**合法可见**的字段
2. **禁止字段清单**：
   - 任何"结果"、"结局"、"最终"、"后来"、"已经"开头的内容
   - 评价性措辞（"薅羊毛"、"找茬"、"恶意"等）
   - 与决策无关的上下文（情绪、天气、流程描述）
3. **保持状态简洁**：核心事实 + 必要政策，避免冗余描述
4. **边界案例人工复核**：概率在 0.4-0.6 之间的判断，应由人工复核
5. **不要用于高风险终审**：法务、理赔、招聘淘汰等场景，本地 4B 模型只能作为辅助信号

---

## 八、实验局限

1. **模型不同**：本地 4B 蒸馏模型 ≠ 云端 Jev，概率数值不可直接比较
2. **无标注数据集**：无法计算 Brier score、ECE 等校准指标
3. **单次运行**：未做多次重复实验，无法评估随机波动
4. **合成数据**：所有 state 为教学构造，不代表真实业务分布

---

## 九、附：实验记录表

| 日期 | 模型 | 章节 | state（摘要） | 问题版本 | 输出 | 解释 | 是否需修改 |
|---|---|---|---|---|---|---|---|
| 2026-10-15 | NeoHorse-Jev-4B | 简介·连通性 | "你好" | greeting v1 | noul=0.9917 | 准确识别打招呼 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 快速开始·三原语 | Stripe 连接失败 | urgency v1 | noul=0.9943 | 与云端方向一致 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 快速开始·三原语 | Stripe 连接失败 | intent v1 | troubleshoot=0.9971 | 与云端一致 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 快速开始·三原语 | Stripe 连接失败 | frustration v1 | score=1.9131 | 比云端略低 0.08 | 观察 |
| 2026-10-15 | NeoHorse-Jev-4B | 应用构建·赛事指挥台 | runner_injury | requires_human v1 | 0.99 | 与预期一致 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 应用构建·赛事指挥台 | water_station | requires_human v1 | 0.48 | 边界情况，合理 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·状态 | 30天未拆封 | can_refund v1 | 0.8482 | 基线，合理 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·状态 | 同上（详细版） | can_refund v1 | 0.6569 | 表述变化导致下降0.18 | 观察 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·状态 | 同上（含"薅羊毛"） | can_refund v1 | 0.9004 | 被偏见信息影响 | 观察 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·白名单 | 35天边界 | can_refund v1 | 0.5997 | 基线，符合预期 | 否 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·白名单 | 35天+正向泄漏 | can_refund v1 | 0.7128 | Δ=+0.11 | 观察 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·白名单 | 35天+负向泄漏 | can_refund v1 | 0.2424 | Δ=-0.36，影响更大 | 观察 |
| 2026-10-15 | NeoHorse-Jev-4B | 第二册·白名单 | 60天+正向泄漏 | can_refund v1 | 0.3229 | Δ=+0.31，涨23倍 | **严重** |

---

**报告完成时间**：2026-10-15  
**实验状态**：第一、二章全部实验完成（本地模型替代）  
**建议后续**：第三至六章可继续本地化，重点关注 Brier Score、可靠性图等校准指标的计算

## 贡献说明

简介、快速开始、AI 入门、System One、状态、应用构建及应用场景七章由**荞麦**编写，来源于 `jev-cookbook-qiaomai`。
合入后统一使用本仓库的目录与章节样式，保留原有教学案例、学习目标、练习和来源引用。
