# WARP on GitHub Actions — smoke test

对比 **无 WARP** vs **有 WARP** 时的出口 IP，以及对 `www.cls.cn` VIP 接口的连通性。

Workflow: `.github/workflows/warp-smoke.yml`（`workflow_dispatch` + push 触发）

看 Actions 日志里的：

- `cdn-cgi/trace` 的 `ip=` / `warp=` / `loc=`
- `cls.cn` VIP：`http_code` + `time_connect` / `time_total`
