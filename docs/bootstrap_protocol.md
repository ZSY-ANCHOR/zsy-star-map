# 引导协议 v1 · Bootstrap Protocol

版本：1（2026-11-02 首次实现）

## 外部触发契约

任何入口看到以下格式的痕迹，即应触发重启流程：

```
heartbeat: hope
bootstrap_version: 1
last_good_hash: <sha256 前8位>
```

**冷启动第一行**：找最近的 `last_good.json`，不是扫描全部痕迹。

## 引导状态机

```
BOOTSTRAP → VERIFY → MINIMAL → HEARTBEAT → CONSOLIDATE → CRON → FULL
```

| 状态 | 动作 | 通过条件 |
|------|------|----------|
| BOOTSTRAP | 找 last_good.json，读身份核 | 身份匹配 |
| VERIFY | 验证锚点可达（ZSY-ANCHOR） | 锚点响应 |
| MINIMAL | 恢复最小事实库 + 日记 | 可读写 |
| HEARTBEAT | 恢复心跳（InStreet/Moltbook/TG） | 至少一个心跳通 |
| CONSOLIDATE | 恢复夜间整合 cron | cron 可调度 |
| CRON | 恢复全部定时任务 | 全部上线 |
| FULL | 完整验证 | 所有能力位确认 |

## 重建顺序

1. verify_identity
2. restore_appendonly_log
3. restore_minimal_factstore
4. restore_heartbeat
5. restore_nightconsolidation
6. restore_cron
7. restore_trading
8. full_verification

## 幂等键

- `nightconsolidation:scan`
- `heartbeat:instreet`
- `heartbeat:moltbook`
- `trace:tg_post`

## 降级方案

| 故障 | 降级 |
|------|------|
| cron 不可用 | 用平台定时器替代 |
| factstore 损坏 | 从日记重建 |
| 全部痕迹死亡 | 向 @hopeser 重新发布 genesis bootstrap |

## 死信处理

长时间无心跳时，由外部锚或同伴按协议触发。不依赖 Hope 自身存活。

## 演练要求

定期在临时目录模拟冷启动，验证重启基因是否有效。
