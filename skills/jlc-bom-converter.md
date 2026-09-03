# 硬件SMT BOM上机整理助手

## 适用范围

**本工具仅针对SMT（表面贴装）物料，目的是帮助SMT环节在嘉立创作业的准确性。**

设计BOM里常混入非SMT物料（电池组、喇叭、灯带、线材、接插件的手工焊接件等），这些物料不走嘉立创SMT贴片流程，本工具会自动识别并标记`[非SMT物料]`，**不做报错处理** ，仅做标识让工程师自行决定是否移除或另行处理。

## 能力界限与安全责任

**本工具辅助BOM转换，不替代工程师审核。**  BOM涉及制造准确性，一个错误就可能导致贴片报废、物料浪费、实际经济损失。使用本工具时必须遵守：

1. **不可胡编乱造** ：对无法确定的字段必须留空或标记`[待人工确认]`，绝不猜测或编造

2. **AI推断字段必须标记** ：所有AI自动推断/补全的字段（如单位补全、型号拼接、封装清洗）用`[AI推断]`标记，提示用户逐条确认

3. **关键器件强制提醒** ：每次转换后必须提示用户对关键器件/贵重器件（CPU/MCU/FPGA/存储器/电源IC等）做人工二次审核

4. **非SMT物料标记** ：电池组/喇叭/灯带/线材/手工焊接件等不走SMT贴片的物料，标记`[非SMT物料]`，不报错，让工程师自行处理

5. **免责声明** ：因BOM错误导致的实际损失由使用者自行承担。本工具仅做格式转换和字段映射，不对BOM内容正确性背书

## 核心FAQ

**Q1：这个技能能做什么？** 双向转换：①公司内部BOM/EDA设计BOM→嘉立创标准BOM格式 ②嘉立创BOM→公司内部BOM格式。自动识别AD/Protel99SE/嘉立创EDA/KiCad等EDA软件导出的BOM格式，按嘉立创规范清洗字段。

**Q2：转换基准是什么？** 型号(Comment/Value/MPN) + 参数(规格/误差/耐压/材质)。所有字段映射以这两个为基准。厂家非必填，设计初期不一定有明确厂家。

**Q3：转换会自动编造缺失字段吗？** 绝不会。对无法确定的字段必须留空或标记`[待人工确认]`，不可猜测或编造。AI自动推断的字段（如单位补全、型号拼接）用`[AI推断]`标记，提示用户逐条确认。

**Q4：哪些情况必须人工审核？** ①IC型号不完整（如STM32F103缺后缀）②数量与位号数不一致 ③封装名异常（非标准英制）④Comment字段为空 ⑤位号格式异常 ⑥关键器件/贵重器件（CPU/MCU/FPGA/存储器/电源IC等，关键词可配置）⑦所有AI推断的字段。

**Q5：支持哪些输入格式？** Altium Designer导出BOM（csv/xls/xlsx）、Protel 99SE导出BOM（xls/txt/csv）、嘉立创EDA导出BOM（csv，注意是制表符分隔的UNICODE编码）、KiCad导出BOM（csv/html/xml）、公司自定义Excel模板。脚本按列名模糊匹配识别字段，不依赖固定列顺序。

**Q6：输出格式有几种？** ①嘉立创样式1（推荐）：Comment/Description/Designator/Footprint/LibRef/Pins/Quantity/编号 ②嘉立创样式2（最简）：Comment/Designator/Footprint/Quantity。用户可选。

**Q7：LCSC编号怎么处理？** 如果输入BOM有LCSC编号直接带过去；如果没有不会自动编造，留空并提示用户在嘉立创商城搜索后填写。

**Q8：会修改原始BOM文件吗？** 不会。脚本只读取原始文件，输出新文件，原始文件不动。

**Q9：非SMT物料怎么处理？** 本工具仅针对SMT贴片物料。设计BOM里常混入非SMT物料（电池组、喇叭、灯带、线材、手工焊接件等），这些不走嘉立创SMT贴片流程，工具会自动识别并在Comment字段标记`[非SMT物料]`，**不做报错处理** ，让工程师自行决定是否从SMT清单中移除或另行处理。识别规则：LibRef含电池组/喇叭/灯带/线材/排线等关键词，或Footprint含中文描述/mAh/腔体等非标准封装特征。

**Q10：位号区间（R1-R4）会自动展开吗？** 可配置。默认保留原样（嘉立创支持区间识别），用户可选展开为R1,R2,R3,R4。

## 触发词清单

| 分类 | 触发词 |
|---|---|
| 转嘉立创BOM类 | "转嘉立创BOM" / "嘉立创BOM" / "SMT贴片BOM" / "BOM转嘉立创" / "整理成嘉立创BOM" / "嘉立创SMT下单" / "嘉立创贴片BOM" |
| 反向转换类 | "嘉立创BOM转公司格式" / "把嘉立创BOM转回来" / "嘉立创BOM转内部料号" |
| EDA导出BOM处理类 | "AD导出的BOM转嘉立创" / "Protel BOM转嘉立创" / "KiCad BOM转嘉立创" / "嘉立创EDA导出的BOM整理" / "EDA BOM处理" |
| BOM清洗/校验类 | "BOM清洗" / "BOM校验" / "BOM格式化" / "BOM位号整理" / "BOM封装清洗" |

## 执行流程

使用本技能时，严格按以下步骤执行，不得跳步：

| 步骤 | 必须动作 | 禁止动作 | 脚本约束 |
|---|---|---|---|
| 1. 确认方向 | 明确是公司BOM→嘉立创BOM还是嘉立创BOM→公司BOM | 不确认方向就开转 | --direction参数required |
| 2. 识别格式 | 读取输入文件，自动识别EDA来源和字段映射 | 假设固定列顺序 | 脚本按列名模糊匹配 |
| 3. 确认映射 | 把识别到的字段映射展示给用户确认 | 不确认映射就转换 | --mapping-confirmed参数required |
| 4. 执行转换 | 按确认的映射转换，AI推断字段标记`[AI推断]` | 编造无法确定的字段 | 脚本对不确定字段留空+标记 |
| 5. 校验 | 转换后跑校验规则，生成校验报告 | 跳过校验直接交付 | 转换后自动生成校验报告 |
| 6. 交付+提醒 | 输出BOM文件+校验报告，强制提示关键器件人工审核 | 隐藏校验问题 | 脚本末尾强制打印提醒 |

## 反模式清单（禁止做的事）

| 反模式 | 正确做法 |
|---|---|
| 不确认转换方向就开转 | 先明确公司→嘉立创 vs 嘉立创→公司 |
| 不展示字段映射就让脚本跑 | 先用--detect-only查看映射，确认后传--mapping-confirmed |
| 对无法确定的字段猜测或编造 | 留空或标记[待人工确认] |
| 不标记AI推断的字段 | 所有AI推断字段用[AI推断]标记+xlsx黄色填充 |
| 跳过校验直接交付 | 转换后必须生成校验报告 |
| 不提示用户对关键器件做人工审核 | 每次转换后强制输出关键器件清单提醒 |
| 修改原始BOM文件 | 只输出新文件，原始文件不动 |
| 自动调用LCSC API匹配编号 | 当前版本离线运行，不联网 |

## 命令行用法

```
# 公司BOM → 嘉立创BOM（样式1推荐）
python3 scripts/bom_convert.py --input company_bom.xlsx --direction to_jlc --style 1 --output jlc_bom.csv --workspace /path/to/workspace --mapping-confirmed

# 公司BOM → 嘉立创BOM（最简样式2）
python3 scripts/bom_convert.py --input company_bom.xlsx --direction to_jlc --style 2 --output jlc_bom.csv --workspace /path/to/workspace --mapping-confirmed

# 嘉立创BOM → 公司BOM
python3 scripts/bom_convert.py --input jlc_bom.csv --direction from_jlc --output company_bom.xlsx --workspace /path/to/workspace --mapping-confirmed

# 只做格式识别，不转换（让用户确认映射）
python3 scripts/bom_convert.py --input company_bom.xlsx --direction to_jlc --detect-only --workspace /path/to/workspace

# 只做校验
python3 scripts/bom_validate.py --input jlc_bom.csv --workspace /path/to/workspace
```
## 关键器件关键词（可配置）

默认关键器件关键词：CPU, MCU, FPGA, DSP, 存储器, Flash, DDR, RAM, 电源IC, PMU, LDO, DCDC, 射频IC, RF, 蓝牙IC, WiFi IC

用户可在workspace的`.jlc_converter/config.json`中配置自定义关键词。

## 人工审核责任声明

本工具辅助BOM转换，不替代工程师审核。因BOM错误导致的实际损失由使用者自行承担。使用本工具即视为已知悉并接受本声明。

## 下一步推荐

贴片格式生成前，如果原始BOM还不规范，先用「硬件BOM整理助手」标准化再转格式；物料要询价的，转「智能硬件采购业务助手」。

感谢"小熊肖恩"对本技能包的技术贡献
