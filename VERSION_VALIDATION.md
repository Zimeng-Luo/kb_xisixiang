# 知识库版本校验要求

## 统一元数据

`nodes/`、`edges/` 和 `materials/` 下每个权威 JSON 必须包含：

```json
{
  "schema_version": "1.0",
  "revision": 0,
  "status": "draft",
  "updated_at": "2026-07-18T00:00:00Z"
}
```

`schema_version` 是结构版本，`revision` 是对象内容修订号，`status` 只能是 `draft`、`reviewed`、`published`、`deprecated`，`updated_at` 必须是 ISO 8601 UTC 时间。RAG 默认只读取 `published`。

## manifest.json

根目录 `manifest.json` 记录 `snapshot_id`、`schema_version`、`source_revision` 和 `indexes` 状态。节点、边或材料变化时递增 `source_revision`；发布快照时生成新的 `snapshot_id`。`indexes.source_fingerprint` 保存所有权威节点、边和材料文件的 SHA-256 指纹，用于识别绕过数据接口的未登记修改。

## data_function 校验顺序

1. 解析 JSON，失败返回 `invalid_json`。
2. 按文件路径选择 `schemas/` 下的独立 schema。
3. 校验同类 ID 唯一、边端点和材料引用存在。
4. 校验版本号、修订号和时间格式。
5. 校验依赖边无自环、重复边和环。
6. 校验 `indexes.built_from_revision` 与 `source_revision` 一致，并校验 `source_fingerprint` 与当前权威文件一致；不一致视为结构错误。数据变更必须通过 `data_function` 同步更新索引，不提供另行执行的索引操作。

错误对象至少包含 `code`、`path`、`message`、`severity`。建议支持 `schema_error`、`duplicate_id`、`missing_reference`、`cycle_detected`、`unsupported_schema_version`。

## 兼容性

增加可选字段是次版本变更；删除字段、修改类型或改变语义是主版本变更。不支持的主版本必须拒绝，旧次版本只能按显式兼容表读取。