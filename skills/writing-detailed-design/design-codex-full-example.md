# 真实目录单个添加医保药品查询逻辑调整详细设计（完整样例）

- 日期：2026-05-15
- 任务：`real-drug-medins-single-add-query-adjustment`
- 适用范围：`yt-pha-opm-server`、`yt-pha-rp-server`、`yt-pha-rp-api`
- 文档口径：本文保留设计决策、边界、方案、功能点和验证口径；稳定事实以 [`core-cognition.md`](./core-cognition.md) 为唯一主落点。
- 样例来源：`harness-hh/spec/v1-39-0-version-iteration/specs/v2/真实目录单个添加药品查询逻辑调整`

## 配套文件

1. 核心认知：[`core-cognition.md`](./core-cognition.md)
2. 地基门禁：[`artifacts/foundation-gate.md`](./artifacts/foundation-gate.md)
3. 主时序图：[`artifacts/real-drug-medins-end-to-end-sequence.puml`](./artifacts/real-drug-medins-end-to-end-sequence.puml)
4. 保存细时序图：[`artifacts/real-drug-medins-single-save-sequence.puml`](./artifacts/real-drug-medins-single-save-sequence.puml)
5. API 契约：[`artifacts/controller-openapi.yaml`](./artifacts/controller-openapi.yaml)

## 1. 背景和目标

药房端新增真实目录医保药品时，页面只要求用户录入目录、商品类别、国家医保药品代码、药店商品编码、商品价格，药品 69 码可选。现有 `POST /r/{yunId}/80200/dp/730` 保存接口要求完整 `DrugCatalogueDto`，不能直接接收少字段入参。

本次目标：

1. 按国家医保药品代码精确查询标准库药品。
2. 同一医保码命中多条标准库记录时，稳定取第一条。
3. 前端只传少数字段，后端补齐标准库字段后保存。
4. 复用现有 `/80200/dp/730` 和 `/80200/dp/525`，不新增接口。
5. 现有完整字段新增、修改、批量导入行为不变。

成功标准：

1. `/80200/dp/525` 传医保码时最多返回一条标准库记录，且多次查询结果稳定。
2. `/80200/dp/730` 在 `simplifiedMedInsAdd=true` 时能按医保码补齐并保存真实目录。
3. `simplifiedMedInsAdd=false/null` 时，原完整字段保存链路不进入医保码补齐分支。
4. 不新增表、字段、索引或缓存。

## 2. 范围和非目标

范围：

1. 药房端真实目录单个新增医保药品。
2. `POST /r/{yunId}/80200/dp/525` 查询匹配药品信息。
3. `POST /r/{yunId}/80200/dp/730` 新增保存。
4. 标准库 `olt_drug_library` 和真实目录 `olt_dp_drug_catalogue` 的查询、补齐、保存关系。

非目标：

1. 不做前端页面布局设计。
2. 不新增数据库表、字段、索引或缓存。
3. 不回写或修正标准库医保码。
4. 不改造 `POST /r/{yunId}/80200/dp/750` 修改保存。
5. 不改变批量简化导入 `POST /r/{yunId}/80200/dp/excel/525`。

## 3. 执行、上线和兼容约束

1. 接口路径不变：继续使用 `POST /r/{yunId}/80200/dp/730` 和 `POST /r/{yunId}/80200/dp/525`。
2. 新增能力是可选简化新增模式，默认保持原完整字段保存。
3. 字段校验继续复用 `DrugDataVerify`，不放宽全局校验。
4. “第一条”固定为 `olt_drug_library.DRUG_LIBRARY_ID ASC LIMIT 1`，避免 MySQL 自然顺序漂移。
5. 无 DDL 变更；回滚时关闭前端简化入口，或让后端忽略 `simplifiedMedInsAdd` 即可回到原行为。

## 4. 核心认知引用

- 核心认知文档：[`core-cognition.md`](./core-cognition.md)
- Foundation Gate：[`artifacts/foundation-gate.md`](./artifacts/foundation-gate.md)
- 核心认知版本：`Vfinal`

本设计依赖的核心结论：

1. `/730` 不支持少字段直接保存，必须先补齐。
2. 标准库医保码不是唯一键，必须定义稳定第一条。
3. 真实目录医保码保存到 `olt_dp_drug_catalogue.med_ins_code`，不回写标准库。
4. 简化新增在真实目录保存校验前补齐字段。

本文不重复表结构、实体属性、状态集合和全局约束。若实现探索发现新的稳定事实，先更新 `core-cognition.md`，再同步本文和配套文件。

## 5. Foundation Gate 消费

Foundation Gate 状态：`confirmed`。

本设计消费两个模块：

| 模块 | 功能点 | API | 存储 | 一致性方案 | 状态 |
| --- | --- | --- | --- | --- | --- |
| 标准库查询 | 国家医保药品代码精确查询第一条 | `POST /r/{yunId}/80200/dp/525` | 无 DDL 变更；只读 `olt_drug_library` | 只读查询，无事务 | `confirmed` |
| 真实目录保存 | 单个医保药品简化新增、完整字段新增兼容 | `POST /r/{yunId}/80200/dp/730` | 无 DDL 变更；写 `olt_dp_drug_catalogue` | 沿用现有保存事务边界 | `confirmed` |

关键证据：

1. `RestDrugCatalogueServiceImpl#queryDpmMatchDrugInfo` 是 `/525` RP 查询落点。
2. `DrugLibraryDao.xml` 已有 `medical_insurance_code` 精确条件实践。
3. `RestDrugStoreController#saveOrUpdateRealDrug`、`RestDrugDirServiceImpl#singleSaveHandler`、`AbstractRealAndVirtualDrugDirService#independentSingleSaveHandler` 是 `/730` 保存链路落点。
4. `olt_drug_library.medical_insurance_code` 不唯一，按 `DRUG_LIBRARY_ID ASC` 稳定取第一条。

## 6. 方案比选

| 方案 | 描述 | 优点 | 缺点 | 结论 |
| --- | --- | --- | --- | --- |
| A | 复用 `/730`，RP 层补齐后保存 | 不新增保存接口；复用现有真实目录保存校验、日志、目录复制和初始化；前端不承担标准库字段一致性 | 新增一个 DTO 触发字段和一个 DAO 精确查询方法；保存前多一次标准库查询 | 采用 |
| B | 新增单独保存接口 | 新旧协议边界清楚 | 接口数量膨胀；权限、日志、文档和联调成本更高；保存规则容易分叉 | 不采用 |
| C | 前端透传完整标准库字段 | 后端改动少 | 前端需要理解真实目录保存必填字段和默认值；标准库字段一致性分散到前端 | 不采用 |

推荐方案：`A`。新增可选简化模式，后端按医保码补齐标准库字段，再进入既有 `/730` 保存链路。

## 7. 总体设计

### 7.1 查询链路

`/80200/dp/525` 入参携带 `medicalInsuranceCode` 或 `medInsCode` 时进入医保码精确查询模式：

1. OPM Controller 继续接收 `DrugCatalogueDto`。
2. RP Service 归一医保码字段：优先 `medicalInsuranceCode`，为空则取 `medInsCode`。
3. 标准库 DAO 执行精确 SQL：`medical_insurance_code = ? ORDER BY DRUG_LIBRARY_ID ASC LIMIT 1`。
4. 命中时转换为现有 `DrugCatalogueVo`，返回列表最多 1 条。
5. 未命中时返回空列表。

### 7.2 保存链路

`/80200/dp/730` 增加简化新增模式：

1. 前端请求携带 `simplifiedMedInsAdd=true`。
2. Controller 保持现有入口，不在 OPM 层补齐标准库字段。
3. RP 真实目录保存实现进入字段校验前判断简化新增模式。
4. 根据医保码查询标准库第一条。
5. 将标准库字段补齐到 `DrugCatalogueDto`，医保码和标准参数以命中标准库第一条为准。
6. 继续执行现有目录类型校验、字段校验、默认值、写库和返回。

补齐位置放在 `AbstractRealAndVirtualDrugDirService#independentSingleSaveHandler` 的 `type` 必填校验之前。简化入参可能不具备完整 `type`，且 `/730` 子药商复制保存同样调用该方法。

### 7.3 前端入参合同

简化新增推荐请求体：

```json
{
  "simplifiedMedInsAdd": true,
  "dpId": 123,
  "type": 0,
  "dpDrugCode": "D001",
  "drugPrice": 12.34,
  "medInsCode": "T001700368",
  "medicalInsuranceCode": "T001700368",
  "drugBarCode": "6900000000000",
  "status": 1,
  "lackStatus": 0
}
```

| 字段 | 必填 | 来源 | 说明 |
| --- | --- | --- | --- |
| `simplifiedMedInsAdd` | 简化分支是 | 前端固定 `true` | 触发简化补齐；`false` 或 `null` 走原逻辑 |
| `dpId` | 简化分支是 | 页面目录 | 真实目录主键 |
| `type` | 建议传 | 页面商品类别 | 医保药品默认 `0`；补齐后以标准库字段校验 |
| `dpDrugCode` | 简化分支是 | 用户录入 | 药店商品编码 |
| `drugPrice` | 简化分支是 | 用户录入 | 大于 0 |
| `medInsCode` | 条件必填 | 用户录入 | 与 `medicalInsuranceCode` 二选一 |
| `medicalInsuranceCode` | 条件必填 | 同 `medInsCode` | 兼容现有字段归一逻辑 |
| `drugBarCode` | 否 | 用户录入 | 药品 69 码，最长 32 |
| `status` | 否 | 默认值 | 默认 `1` |
| `lackStatus` | 否 | 默认值 | 默认 `0` |

## 8. 功能点详细设计

### 8.1 功能点一：医保码精确查询标准库第一条

功能：输入国家医保药品代码，后端精确查询标准库第一条，供页面展示默认参数和标准参数。

参与实体：

1. `DrugCatalogueDto`：接收 `dpId`、医保码、`type`。
2. `DrugLibraryVo`：承载标准库查询结果。
3. `olt_drug_library`：标准库事实源。

执行路径：

1. 前端调用 `POST /r/{yunId}/80200/dp/525`。
2. OPM Controller 透传 `DrugCatalogueDto`。
3. RP Service 归一医保码字段。
4. DAO 按 `medical_insurance_code` 精确查询，并按 `DRUG_LIBRARY_ID ASC LIMIT 1` 取第一条。
5. 返回最多一条 `DrugCatalogueVo`。

序列图：[`artifacts/real-drug-medins-end-to-end-sequence.puml`](./artifacts/real-drug-medins-end-to-end-sequence.puml)

| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| Q1 | 请求提供 `dpId`，且 `medicalInsuranceCode` 或 `medInsCode` 非空 | 医保码命中一条或多条标准库记录 | 返回列表最多 1 条，取 `DRUG_LIBRARY_ID` 最小记录 | 无 | 用同一医保码多次查询，返回同一 `drug_library_code` |
| Q2 | 请求提供 `dpId`，且医保码非空 | 医保码未命中标准库 | 返回空列表，不写库 | 无 | 用不存在医保码查询，不报错且不写库 |
| Q3 | `dpId` 为空 | 调用 `/525` | 沿用现有“入参_药商主键不能为空” | 请求失败 | 调用接口确认返回现有错误语义 |

### 8.2 功能点二：单个医保药品简化新增

功能：前端少字段提交，后端按医保码补齐标准库字段后保存真实目录。

参与实体：

1. `DrugCatalogueDto`：简化请求和补齐后的保存 DTO。
2. `DrugLibraryVo`：标准库第一条补齐来源。
3. `olt_dp_drug_catalogue`：真实目录保存目标。
4. `olt_drug_library`：补齐来源。

执行路径：

1. 前端调用 `POST /r/{yunId}/80200/dp/730`。
2. 请求体带 `simplifiedMedInsAdd=true`。
3. RP 保存链路在字段校验前查询标准库第一条。
4. 标准库字段补齐到 `DrugCatalogueDto`。
5. 继续走现有 `singleSaveHandler`、字段校验、写库和返回。

序列图：[`artifacts/real-drug-medins-single-save-sequence.puml`](./artifacts/real-drug-medins-single-save-sequence.puml)

| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| S1 | `simplifiedMedInsAdd=true`，`dpId`、`dpDrugCode`、`drugPrice`、医保码非空，且标准库第一条字段完整 | 调用 `/730` | 保存成功；记录包含 `drug_library_code`、`med_ins_code`、价格、药店商品编码和补齐字段 | 无 | 查 `olt_dp_drug_catalogue`，`med_ins_code` 和 `drug_library_code` 来源于标准库第一条 |
| S2 | 同 S1，但医保码未命中标准库 | 调用 `/730` | 不保存，返回“未匹配到标准库药品，请核对医保码” | 请求失败 | 调用 `/730` 后确认无新增记录 |
| S3 | 同 S1，但 `dpId + dpDrugCode` 已存在 | 调用 `/730` | 沿用“药店商品编码已存在” | 请求失败 | 预置同编码记录后调用，确认被拒绝 |
| S4 | 同 S1，但标准库第一条缺少保存必填字段 | 调用 `/730` | 进入 `DrugDataVerify`，返回具体字段错误 | 请求失败 | 选择缺字段标准库记录，确认错误字段明确 |
| S5 | 同 S1，且前端传药品 69 码 | 调用 `/730` | 保存到 `drug_bar_code` | 长度超 32 时拒绝 | 保存后查库；超长输入确认失败 |

### 8.3 功能点三：完整字段新增兼容

功能：不带简化新增标识的 `/730` 请求保持现有完整字段保存行为。

参与实体：

1. `DrugCatalogueDto`：现有完整保存请求。
2. `olt_dp_drug_catalogue`：真实目录保存目标。

执行路径：

1. 前端调用 `POST /r/{yunId}/80200/dp/730`。
2. `simplifiedMedInsAdd` 为空或 `false`。
3. 后端不执行医保码补齐。
4. 继续原完整字段保存链路。

序列图：[`artifacts/real-drug-medins-single-save-sequence.puml`](./artifacts/real-drug-medins-single-save-sequence.puml)

| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| C1 | `simplifiedMedInsAdd` 为空或 `false`，请求体为现有完整 `DrugCatalogueDto` | 完整字段新增 | 不执行医保码补齐，行为与现状一致 | 沿用现有校验 | 用现有完整字段请求回归新增成功 |
| C2 | 同 C1，但完整字段缺少必填字段 | 完整字段新增 | 沿用现有字段校验错误 | 请求失败 | 缺少通用名或剂型编码，确认错误不变化 |

## 9. 改动资产

### 9.1 Java 属性

`yt-pha-rp-api/.../DrugCatalogueDto.java`

- 新增 `private Boolean simplifiedMedInsAdd;`
- 默认 `false` 或 `null` 表示关闭简化分支。

### 9.2 Java 方法

`yt-pha-rp-server/.../DrugLibraryDao.java`

- 新增 `selectFirstByMedicalInsuranceCode(String medicalInsuranceCode)`，返回 `DrugLibraryVo`。

`yt-pha-rp-server/.../DrugLibraryDao.xml`

- 新增 SQL：`medical_insurance_code = ? ORDER BY DRUG_LIBRARY_ID ASC LIMIT 1`。

`yt-pha-rp-server/.../AbstractRealAndVirtualDrugDirService.java`

- 新增 `isSimplifiedMedInsAdd(DrugCatalogueDto)`。
- 新增 `enrichSingleMedInsAddData(DrugCatalogueDto)`。
- 调用位置：`independentSingleSaveHandler` 开始处，早于 `type` 必填校验。

`yt-pha-rp-server/.../RestDrugCatalogueServiceImpl.java`

- 调整 `queryDpmMatchDrugInfo(DrugCatalogueDto)`：医保码非空时进入精确查询分支，返回最多一条。

## 10. 字段补齐规则

前端字段保留，不被标准库覆盖：

1. `dpId`
2. `dpDrugCode`
3. `drugPrice`
4. `drugBarCode`
5. `status`
6. `lackStatus`

医保码字段处理：

1. `medInsCode`、`medicalInsuranceCode` 在命中标准库第一条后统一回填为标品值。
2. 回填口径与简化批量导入保持一致。

标准库补齐字段至少包括：

1. `type`
2. `drugChemName`
3. `drugTradeName`
4. `drugSpec`
5. `drugUnit`
6. `drugForm`
7. `drugLibraryCode`

## 11. 接口与验收

接口增量：

| 字段 | 位置 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| `simplifiedMedInsAdd` | `DrugCatalogueDto` 请求体 | `Boolean` | 否 | `true` 时触发简化新增补齐；`false/null` 保持原完整字段保存 |
| `medicalInsuranceCode` / `medInsCode` | `DrugCatalogueDto` 请求体 | `String` | 简化分支条件必填 | 国家医保药品代码，二选一；前端推荐两者同传 |

验收用例：

1. 查询同一医保码多次返回同一条标准库记录。
2. 查询不存在医保码返回空列表，不报错、不写库。
3. 简化新增成功后，真实目录记录的 `med_ins_code`、`drug_library_code` 与标准库第一条一致。
4. 简化新增医保码未命中时拒绝保存。
5. 完整字段新增不进入医保码补齐分支。

回归验收：

1. `/80200/dp/730` 既有完整字段新增行为不变。
2. `/80200/dp/750` 修改保存不受影响。
3. `/80200/dp/excel/525` 批量简化导入不受本次查询分支影响。

## 12. 风险、待确认和受影响资产

风险：

1. 标准库同一医保码存在多条记录，若不固定排序会导致页面展示和保存补齐不稳定。本设计用 `DRUG_LIBRARY_ID ASC LIMIT 1` 固定口径。
2. 简化新增请求字段少于完整保存字段，补齐必须发生在 `type` 必填校验之前。
3. 前端若同时传 `medicalInsuranceCode` 和 `medInsCode` 且值不一致，后端需按字段归一规则处理，建议前端同传同值。

待确认：

- 无阻塞待确认。

受影响资产：

- 后端：`DrugCatalogueDto`、`DrugLibraryDao`、`DrugLibraryDao.xml`、`AbstractRealAndVirtualDrugDirService`、`RestDrugCatalogueServiceImpl`。
- 前端：真实目录单个新增医保药品页面。
- 数据库：无结构变更。

计划同步：

- 若 `PLAN_PATH` 不存在，记录“不涉及既有计划同步”。
- 若 `PLAN_PATH` 已存在，需检查新增 DTO 字段、DAO 查询、保存补齐、查询回归和完整新增回归是否进入实现任务。

## 13. 完成前自检

1. 主文档已引用 `core-cognition.md` 和 `artifacts/foundation-gate.md`。
2. 无 DDL 变更已在范围、Foundation Gate 和存储约束中明确。
3. `/525` 查询、`/730` 保存和完整新增兼容均有功能点和验收。
4. `then` 与 `verify` 可以按编号对应。
5. 旧接口保留、旧链路不变、回滚方式清楚。

