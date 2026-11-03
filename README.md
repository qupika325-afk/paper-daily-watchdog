# arxiv-daily-watchdog

兜底看门狗：给 `qupika325-afk/zotero-arxiv-daily` 的「Send emails daily」保送达。

## 为什么单独一个仓库
GitHub 会丢弃长期没有活动的仓库（尤其是 fork）的 `schedule` 触发。2026-10 那次论文邮件断供，就是因为
那个 fork 自 10-01 起零活动，10-14 之后 GitHub 一次 `schedule` 都没有投递，而仓库里用来保活的
keep-alive 工作流自己也依赖 `schedule`，直接死锁。

这个仓库**不是 fork**，并且每天由工作流自己提交一次心跳（`state/last-check.txt`），活动度自循环，
所以它的 cron 能稳定跑；再用它去触发那边的日报。

## 干什么
每天 08:20 UTC：

1. 查目标仓库的 `main.yml` 今天有没有 run；
2. 如果有 queued/in_progress 或已 success → 跳过（不重复发信）；
3. 否则调 `POST /actions/workflows/main.yml/dispatches` 补跑一次；
4. 提交一行心跳，顺带保住本仓库的活动度。

判定失败（token 失效 / 权限不足 / 触发返回非 204）时这一步直接 exit 1，Actions 里显示红色，
GitHub 会给你的邮箱发失败通知。

## 需要的配置
仅一个仓库 Secret：`DISPATCH_TOKEN` = fine-grained PAT，最小权限：

- Resource owner: 你自己（`qupika325-afk`）
- Repository access: **Only select repositories** → `zotero-arxiv-daily`
- Permissions: **Actions: Read and write**（只有 `workflow_dispatch` 这一个调用）
- Expiration: 90 天

心跳提交用的是工作流自带的 `GITHUB_TOKEN`（`permissions: contents: write`），不需要额外凭证。
