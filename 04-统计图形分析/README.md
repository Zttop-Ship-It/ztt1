# 04 统计图形分析

这一节用于把 `02` 案例里的统计图形还原为**数据表、计算口径和视觉映射**。分析时先在 `01` 判断数据的主组织关系，在 `02` 记录案例表现标签，再在本节识别每个统计模块。一个案例内可以有地图、条形、图符或流程箭头等多个模块，各自核对证据、统计单位和读法。图像、来源及具体读图叙事见原案例报告。

## 1. 从 RAWGraphs 源码学到的分析顺序

本节读到的是 RAWGraphs App 提交 `48c3b2ee9ff4`。其锁文件解析出的版本为 `rawgraphs-charts 1.1.0`、`rawgraphs-core 1.0.0`。应用注册内置图形并处理导入、字段映射、预览和导出。各图的必填字段、可接收类型、聚合和绘制规则还要查对应图形的定义。以下图形说明针对这个源码版本。 [应用说明](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/README.md) · [依赖锁文件](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/package-lock.json) · [图形注册清单](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js)

| 步骤 | 分析时写什么 | 源码参照 |
| --- | --- | --- |
| 读表 | 一行代表什么，一列是什么类型。核查分隔符、千位和小数分隔符、日期区域设置、空值与零值。长表转换前先检查保留的标识列和待展开的度量列 | [数据读入](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoader.js) · [解析](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoaderUtils/parser.js) · [长表转换](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoaderUtils/stack.js) |
| 对字段 | 写下必填与可选维度。把分类、数值、日期分别映射到各自的角色，说明数据类型与字段的语义是否合适。邮编可以全是数字，作为地区键时仍按类别读 | [字段匹配](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/DataMapping/ChartDimensionCard.js) · [必填与类型检查](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartPreview/ChartPreview.js) |
| 核算 | 确定分组键与聚合方法，例如按地区求和、按类别计行或按样本位置取中位数。核对是否重复纳入小计，百分比重算前先确认分母 | [条形图的数据映射](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/barchart/mapping.js) |
| 读图 | 指出真正承担读数的通道，以及分组、强调和标注各自使用什么。任选一项用数据复算，再对照读图结果 | [气泡图尺寸绘制](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/bubblechart/render.js) · [箱线图绘制](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/boxplot/render.js) |
| 复核与记录 | 记下字段映射、聚合、排序、尺度和关键选项。需要留档时可同时保存可编辑的 SVG 和可回读的项目配置 | [导出实现](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/Exporter/Exporter.js) |

### 三个会改变读数的实际规则

- 普通条形图按 `Series` 和 `Bars` 分组。填了 `Size` 就用该维度的聚合配置，空着通常得到**组内记录行数**。已经汇总过的表一类往往只有一行，要比较汇总金额时把金额列映射到 `Size`。 [条形映射](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/barchart/mapping.js)
- 桑基图要求 `Source node`、`Target node`。在此版本，`Size` 留空时每组源—目标配对的权重是 **1**。同一对端点重复出现多行时，若要表达频数，先聚合出频数列并映射到 `Size`。 [桑基维度](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/sankeydiagram/dimensions.js) · [桑基映射](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/sankeydiagram/mapping.js)
- 多阶段流带图（Alluvial）至少要向 `Steps` 依次放入两列。它按**相邻阶段配对**生成流带，`Size` 留空时使用该配对下的记录条数，也可映射明确的量值。在中间阶段汇合后，图上能读到相邻两段关系，整条跨阶段路径仍需从有配对键的源表核定。 [流带维度](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/alluvialdiagram/dimensions.js) · [流带映射](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/alluvialdiagram/mapping.js)

## 2. 按问题和字段选统计图形

| 读者的问题 | 每行记录什么 | 可选图形及映射 | 读数前核对 |
| --- | --- | --- | --- |
| 各类别谁更多、差多少？ | 一个类别的汇总，或一条待聚合记录 | [条形图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/barchart/dimensions.js)，类别→`Bars`，数值→`Size`。同类下的多组并列可用[分组条形](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/barchartmultiset/dimensions.js) | 条长由共同基线比较。计行、求和和平均分别表达不同问题 |
| 同类内各部分占多少？ | 一个总体中的一个组成部分 | [堆叠条形](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/barchartstacked/dimensions.js)，总体→`X Axis`，分组→`Series`，部分数值→`Size` | 各部分采用相同单位与分母。内部色段起点不同，逐组精确比较时可用并列条形 |
| 随时间怎样变化？ | 一个时点和一个系列的观测 | [折线图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/linechart/dimensions.js)，日期→`X Axis`，度量→`Y Axis`，对象→`Lines` | 缺测和零值分别处理。横轴时间间距按真实日期判断，同一时点多条记录须说明聚合 |
| 两个变量怎样一起变化？ | 一个观测对象 | [气泡图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/bubblechart/dimensions.js)，数值或日期→`X Axis`、`Y Axis`，可选数值→`Size` | 两轴位置表达两个变量。此实现用平方根尺度映射半径，使面积随正值变化，像素取整会影响小圆。相关关系需保留证据边界 |
| 一组样本集中在哪里、分散多远？ | 一条原始样本观测 | [箱线图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/boxplot/dimensions.js)，测量值→`Y Axis`，类别→`Groups`。还可比较蜂群点图或小提琴图 | 箱体用四分位数，中央线是中位数。用原始样本核对样本量，另查须线及异常值的定义 |
| 两个字段交叉后的值是多少？ | 一个交叉组合，或一条可聚合记录 | [矩阵图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/matrixplot/dimensions.js)，行列字段→`Y Axis`、`X Axis`，度量→`Color` 或 `Size` | 区分缺格和实际为零。用色阶表达量级时说明范围，避免跨图比较不共用的色阶 |
| 哪个分支包含哪些成员、规模多大？ | 一个叶子条目及其上级路径 | [矩形树图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/treemap/dimensions.js)或[旭日图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/sunburst/dimensions.js)，从上到下的层级字段→`Hierarchy`，规模→`Size` | 叶子条目和父级小计应能区分。要看分叉关系可考虑树状图。面积或角度的精确读数优先看数值 |
| 从哪里流向哪里、流量多大？ | 一组源—目标的明细，或一条含各阶段字段的记录 | 桑基用 `Source node`、`Target node`、`Size`。多阶段流带用 `Steps`、`Size` | 先核对边的方向、量值和单位，再读线宽。注释引线和无权重的流程箭头只记录其关系 |
| 一项任务什么时候开始、持续多久？ | 一个任务区间 | [甘特图](https://github.com/rawgraphs/rawgraphs-charts/blob/v1.1.0/src/ganttChart/dimensions.js)，起止→`Start date`、`End Date`，任务名可用于分组 | 条形两端对应时间。端点精度要一致，只有步骤顺序的案例用顺序键记录 |

地图案例仍要检查地理键、几何和投影。这个应用的内置图形注册清单里未见专门的地理底图图形，带地图的案例可以先单独分析其中的统计模块，再处理空间位置。 [图形注册清单](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js)

## 3. 用仓库现有案例验证分析边界

| 案例 | 先还原的数据表 | 可以确认的统计读取 | 复建条件 |
| --- | --- | --- | --- |
| [第 29 页老龄化、肥胖与 NHS 人员](../02-%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%BD%A2%E7%B1%BB%E5%9E%8B%E6%A1%88%E4%BE%8B/08-%E5%9B%BE%E7%89%87%E5%BC%8F-Photo-graphic-Infographics/02-%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90/CASE-PDF3-29A-%E5%88%86%E6%9E%90%E6%8A%A5%E5%91%8A.md) | 国家、指标、年份、数值、单位分别列。三个模块的分母分开记录 | 照片填在柱内，实际读数依柱长或标值。等大的红圆通过文字提供两年人数 | 对老龄化和肥胖分别做条形映射，保留不同统计口径 |
| [第 67 页美国农产品出口](../02-%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%BD%A2%E7%B1%BB%E5%9E%8B%E6%A1%88%E4%BE%8B/01-%E6%B7%B7%E5%90%88%E5%9E%8B%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%BD%A2-Mixed-Charts/02-%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90/CASE-PDF3-67A-%E5%88%86%E6%9E%90%E6%8A%A5%E5%91%8A.md) | 已转录的是州、货品、市场各自的边际汇总。复建需要州、货品、市场和金额的配对明细，或至少各相邻层的边表 | 读端点汇总和流带的相对宽窄。三个层面的汇总对应同一出口总量 | 可用 Steps 或 Source／Target 映射明细。没有明细时保留边际汇总，各流带金额待核实 |
| [第 43 页 Fortune 500 营收中心](../02-%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%BD%A2%E7%B1%BB%E5%9E%8B%E6%A1%88%E4%BE%8B/07-%E5%9C%B0%E6%A0%87%E5%9E%8B-Location-Infographics/02-%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90/CASE-PDF3-43A-%E5%88%86%E6%9E%90%E6%8A%A5%E5%91%8A.md) | 都市圈、年度、营收、单位、地理键分别列 | 外移大小圆提示营收量级，标值给精确金额，引线把圆接回地图位置 | 要复建精确的面积比例圆，先确认可核验的半径或面积尺度。原图引线可作为定位关系转录 |
| [第 59 页分行业就业](../02-%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%BD%A2%E7%B1%BB%E5%9E%8B%E6%A1%88%E4%BE%8B/10-%E5%8D%95%E4%B8%80%E5%9B%BE%E8%A1%A8%E5%9E%8B-Single-chart/02-%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90/CASE-PDF3-59A-%E5%88%86%E6%9E%90%E6%8A%A5%E5%91%8A.md) | 一行业一状态一行，分别记图符数、单枚比例及分母口径 | 一枚等大图符代表本行业工作者的 5%。造船业 12 枚失业图符，对应 60% | 可另用条形呈现相同比例。若研究原图的 ISOTYPE 方法，保留图符重复及图例规则 |
| [第 32 页品牌建设流程](../02-%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%BD%A2%E7%B1%BB%E5%9E%8B%E6%A1%88%E4%BE%8B/05-%E6%B5%81%E7%A8%8B%E5%9B%BE%E5%9E%8B-Process-Infographics/02-%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90/CASE-PDF3-32B-%E5%88%86%E6%9E%90%E6%8A%A5%E5%91%8A.md) | 一条连接一行，起点、终点、关系类型分列 | 箭头表达连接与方向，节点大小用于强调。图面没有可确定的流量字段 | 可以复建连接关系，流量留缺 |

## 4. 可直接套用的案例分析记录

| 项目 | 要填入的内容 |
| --- | --- |
| 读者问题 | 要查值、比较、观察分布、追踪变化、读层级，还是看关系？ |
| 数据粒度 | 一行对应哪一个对象，主键或配对键是什么？ |
| 字段和口径 | 按列记录名称、类型、单位、分母、时间范围、来源。把推断和未知单列 |
| 统计处理 | 写出排序、过滤、分组和聚合规则，必要时给出算式及一条复算记录 |
| 视觉映射 | 写出字段 → 图形角色 → 读图方式。附尺度、图例、基线或状态标记 |
| 可复建程度 | 记录可复建的数据、需估读的结构和待补的原始明细，并链接原案例报告 |

### 快速自检示例

假设三条记录中，甲类金额分别为 20、30，乙类为 40。用条形图按类别分组。金额映射到 `Size` 且选择求和时，读到的是 **甲 50、乙 40**。留空 `Size` 时，读到的是 **甲 2 行、乙 1 行**。同一张表先定统计问题，再选择数值列和聚合方式。
