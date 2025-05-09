# 数据科学导论实验

## 实验要求

### 评分点

实验部分重点是为了让大家在实践中熟悉数据科学知识，锻炼团队合作能力，只要在报告中叙述清楚、内容合理即可。

评分标准：

1. 认真度
2. 工作量
3. 思路是否清晰完备
4. 个人收获
5. 实际结果
6. 报告的格式是否专业
7. 能否提交源码
8. 项目组成员
9. 任务分工和组织
10. 个人总结收获

### 数据集和问题

新西兰交通事故分析数据集（**New Zealand Crash Analysis System，简称 CAS 数据集**）是由新西兰交通局（Waka Kotahi NZ Transport Agency）管理并公开的一套详细交通事故记录数据，广泛用于道路安全研究、城市规划与政策制定。

问题：
1.雨天夜间无路灯的事故是否更容易导致重伤或死亡？
2.某些地区是否在特定年份事故数量激增？可能与政策、道路施工等有关。
3.随着时间的推移，某些类型的车辆是否更频繁地参与事故？

### 实验报告内容

根据问题和数据集，在该数据集上进行实验并对结果进行评价，将所得结果以报告形式提交。内容包括：

1. 问题定义 
2. 数据集介绍与预处理 
3. 模型的设计与实现 
4. 结果评价与展示 

## 数据集简介

### 数据来源

* 数据来自**新西兰交通事故分析系统（Crash Analysis System, CAS）**
* 该系统记录了**1980年代至今所有由警方正式报告的交通事故**
* 数据由\*\*新西兰交通局（Waka Kotahi NZTA）\*\*管理，并对公众和研究人员开放历史数据

### 数据粒度

* 每条记录对应一起**警方正式记录的交通事故**
* 事故发生的**时间、地点、道路条件、涉及车辆、人员伤亡信息**都被详细记录

---

## 数据内容结构

### 1. **Crash（事故信息）**

* `CrashID`：事故编号（唯一键）
* `Crash_Year`：事故发生年份
* `Date/Time`：具体时间戳
* `Location`：坐标、道路名称、城市
* `Weather`：晴天、雨天、雾天等
* `Light`：白天、夜晚、有灯/无灯
* `Road_Surface`：干燥、湿滑、积雪
* `Speed_Limit`：限速值
* `Crash_Severity`：无伤（Non-injury）、轻伤、重伤、死亡

### 2. **Vehicle（涉及车辆）**

* `Vehicle_Type`：轿车、卡车、摩托车、自行车等
* `Vehicle_Movement`：直行、转弯、变道等
* `Vehicle_Age`：车辆年龄
* `Vehicle_Registration`：是否注册、是否为非法车辆

### 3. **Person（相关人员）**

* `Role`：驾驶员、乘客、行人、自行车骑手等
* `Age/Sex`：年龄和性别
* `Injury_Severity`：无伤、轻伤、重伤、死亡
* `Seatbelt/Helmet`：是否使用安全带/头盔

---

## 空间与时间信息

* **地理信息完整**：包括事故具体经纬度坐标、道路名称、行政区划（城市、地区）
* **可用于地图可视化与空间分析（GIS）**
* **时间跨度大**：可以分析长期趋势，如 2000–现在的事故变化

---

## 数据用途与研究方向

* **交通安全分析**：高发路段识别、天气和时间对事故的影响
* **政策效果评估**：例如限速调整、道路照明、道路整修后事故率变化
* **城市交通规划**：识别设计问题或事故热点
* **机器学习建模**：预测事故严重性、分析事故模式、生成安全评分模型

---

## 获取方式

* **开放版本**可从 Waka Kotahi 官网或 [NZ Open Data Portal](https://opendata-nzta.opendata.arcgis.com/) 免费下载，通常以 CSV 或 GIS 格式提供
* **完整数据集（带详细坐标和个人信息）** 需通过研究申请获得授权使用

---



## 概述

针对新西兰交通事故分析数据集，我们从以下三方面提出数学建模思路：

1. **雨天夜间无路灯事故的严重性分析**：采用二元或有序逻辑回归、泊松回归，以及配对病例–对照等方法，评估“雨天”“夜间”“无路灯”对重伤或死亡的影响；
2. **区域年度事故激增检测**：通过时序聚合与可视化、CUSUM/贝叶斯突变点检测、中断时间序列分析，以及时空扫描统计，寻找特定地区在特定年份事故数显著上升的原因；
3. **车辆类型参与事故的时间趋势**：计算各类车辆每年的参与比例，用Mann–Kendall检验、ARIMA/泊松回归和Joinpoint回归来识别长期趋势与结构性变化。

---

## 1. 雨天夜间无路灯事故与重伤/死亡

### 1.1 数据准备

* **筛选条件**：提取道路表面“湿滑”（Rain）、事故时间为夜间（Night）、道路照明为“无灯”（NoLight）的记录。
* **因变量**：设定二元变量

$$
Y = 
\begin{cases}
1, & \text{重伤或死亡（serious/fatal）} \\
0, & \text{轻伤或未伤（minor/no injury）}
\end{cases}
$$

或使用有序因变量分级（轻伤、重伤、死亡）。

### 1.2 统计建模

1. **二元逻辑回归（Logistic Regression）**

   $$
   \log\frac{P(Y=1)}{1 - P(Y=1)}
   = \beta_0 + \beta_1\,\text{Rain} + \beta_2\,\text{Night} + \beta_3\,\text{NoLight}
   + \beta_4\,(\text{Rain}\times\text{NoLight}) + \dots
   $$

   用以估计各因素及其交互项对严重后果的边际影响。
2. **有序逻辑回归（Ordinal Logistic Regression）**
   针对三级或更多严重性等级，建模累积对数几率（cumulative log‑odds）。
3. **泊松／负二项回归（Poisson/Negative Binomial）**
   对道路路段（或时间段）上严重事故次数建模，并纳入交通流量等暴露变量作控制。

### 1.3 因果推断

* **配对病例–对照设计**：将“雨天夜间无灯”事故与“其他条件相似”（限速、路段曲率、流量等）的对照事故配对，估计严重伤亡的比值比（odds ratio）。

---

## 2. 区域年度事故数量激增检测

### 2.1 探索性聚合与可视化

* **区域–年度汇总**：将每条事故记录按区域（行政区或Meshblock）和年份分组计数。
* **可视化**：热力图或折线图展示各区域随年份的事故变化，初步发现潜在激增。

### 2.2 突变点检测

* **CUSUM图（累积和控制图）**：监测时间序列累计偏差，发现显著漂移时点。
* **贝叶斯突变点模型**：通过贝叶斯框架自动识别多个潜在分段点。

### 2.3 中断时间序列分析

* **分段回归（Segmented Regression）**：

  $$
  Y_t = \alpha + \beta_1\,t + \beta_2\,D_t + \beta_3\,(t - T_0)\,D_t + \varepsilon_t
  $$

  其中 $D_t$ 指示某政策或施工开始后的时间段，检验干预前后趋势与水平的变化。

### 2.4 时空热点扫描

* **Kulldorff时空扫描统计（SaTScan）**：检测在特定区域–年份组合中，事故数是否显著超出预期。
* **广义线性混合模型（GLMM）**：以区域和年份为随机效应，量化区域差异及年度波动。

---

## 3. 车辆类型参与事故的时间趋势

### 3.1 年度参与比例与非参数检验

* **计算比例**：对每一年，每种车辆类型（轿车、SUV、摩托车等）参与事故的占比。
* **Mann–Kendall检验**：检验各类型比例随年份的单调上升或下降趋势。

### 3.2 ARIMA 与泊松时间序列模型

1. **ARIMA模型**：对各类型事故总数构建自回归积分滑动平均模型，分析周期性与趋势。
2. **泊松回归**：

   $$
   \log(\lambda_{v,t}) = \alpha_v + \beta_v \, t
   $$

   其中 $\lambda_{v,t}$ 为第 $t$ 年类型 $v$ 的预期事故数，$\beta_v$ 描述年趋势。

### 3.3 Joinpoint 回归断点检测

* **结构性变化识别**：通过Joinpoint方法自动寻找趋势中显著变化的年份，揭示如法规变更或车辆市场结构转折点。

---

**后续工作建议**

1. 编写数据清洗与预处理脚本，生成分析所需子集与特征；
2. 进行初步可视化与描述性统计，验证模型假设；
3. 依上述思路拟合模型，并用交叉验证、敏感性分析评估稳健性；
4. 将结果与新西兰相关政策、路网改造等背景结合，开展进一步解释与建议。
