# WARP on GitHub Actions — 研究备忘

**现状：不上 `news-crawler` 生产。** 本仓仅保留冒烟与结论，供以后（尤其 **Matrix 分片扫描**）评估。

## 为何值得留着

- VIP 门控已可用 **CF Worker 代拉**（`jxie.ccwu.cc/vip`），不必靠 WARP。
- **分片 job** 仍大量直连 `www.cls.cn`（`/es/quotes/articles` 等）。若 Azure 出口间歇 `ETIMEDOUT`，WARP 换出口可能有用；每个 shard job 都要各自 Setup WARP。

## 冒烟结果（2026-09-22）

Run: https://github.com/jxie8301-pixel/test/actions/runs/35724856103  
Action: `fscarmen/warp-on-actions@v1.4`，`mode: client`，`stack: ipv4`

| | baseline（无 WARP） | with WARP |
|--|---------------------|-----------|
| 出口 IP | `20.98.18.68`（Azure） | `104.28.227.110`（CF） |
| `warp=` | `off` | `on` |
| `cls.cn` VIP 连通 | `http=200`，connect≈0.67s | `http=200`，connect≈0.46s |
| VIP 业务体 | `errno=10012` 签名错误（裸 curl 无 sign，预期内） | 同左 |

结论：

1. WARP **能**把 GHA 出口从 Azure 换成 Cloudflare，且 `warp-cli` Connected。
2. **该次** Azure 也能通 `cls.cn`，看不出 WARP 对 VIP 的额外收益；线上超时是间歇性的，需多次/故障窗口再测。
3. 出口换 CF ≠ 国内 IP；`loc` 仍可能是 US。

## 以后若上分片

- 只给 **scan shard** job 加 WARP（gate / merge 可不加，或 gate 继续走 CF `/vip`）。
- Pin action 到 commit SHA；先在本仓复测再改 `news-crawler`。
- 评估：安装耗时、失败重试、Cloudflare/GitHub ToS、第三方 action 供应链。

## 分片扫描对比（WARP vs baseline）

Workflow: `.github/workflows/warp-shard-scan.yml`

- 复用公开仓 `jxie8301-pixel/news-crawler` 的 `node src/cli.js scan`
- 小股票池：`fixtures/pool-smoke.json`（约 100 只），5 分片
- 两组 Matrix：`scan-baseline` / `scan-warp`（每片先打 WARP）
- 日志里看每片 `errors` / `errorKinds` / `elapsed_sec`；artifact 为 `*-shard-N.json`

**生产 `news-crawler` 仍不上 WARP。**
