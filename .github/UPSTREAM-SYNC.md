# 上游同步说明（Sync Upstream）

本仓库是 [`mosheng20205/lingbuilder`](https://github.com/mosheng20205/lingbuilder) 的镜像部署，
通过 GitHub Actions 每天自动检查上游是否有新提交，有则自动同步。

## 工作方式

| 项目 | 说明 |
| :--- | :--- |
| 触发时机 | 每天 UTC 02:17（北京时间 10:17），GitHub 服务器执行，本机无需开机 |
| 手动触发 | 仓库页 `Actions` → `Sync Upstream` → `Run workflow` |
| 配置文件 | `.github/upstream-sync.json` |
| 同步范围 | 分支提交 + 上游 tag |

## 配置项

```json
{
  "upstream": "https://github.com/mosheng20205/lingbuilder.git",
  "branch": "main",
  "mode": "merge",
  "syncTags": true
}
```

- `mode: "merge"`（默认）—— 上游更新与本仓库改动合并，冲突时任务失败并列出冲突文件，不会破坏你的副本。
- `mode: "mirror"` —— 硬重置为上游状态，完全一致，但**会丢弃本仓库的所有本地改动**。
- `upstream` / `branch` 可改，因此这套文件可以直接复制到别的镜像仓库复用。

## 首次部署说明

本仓库的第一个提交是**引导提交**：只有 `.github/` 三个文件（约 4 KB）。
因为本机到 GitHub 的上行带宽受代理限制、推不动 164 MB 的上游仓库，
所以首次全量内容由工作流在 GitHub 内网直接从上游拉取后写入——
工作流检测到「与上游无共同祖先」会自动执行这一步并强推，之后再走正常的合并同步。

## 两个限制要知道

1. **定时任务会休眠**：仓库连续 60 天没有任何提交活动，GitHub 会自动停用 schedule，需要手动进 Actions 点一次 `Enable workflow`。
2. **私有仓库计费**：Public 仓库的 Actions 免费；Private 仓库会消耗账号的 Actions 分钟数（每月有免费额度）。

## 在本地手动同步

```bash
cd lingbuilder
git fetch upstream --prune --tags --force
git merge --no-edit upstream/main
git push origin main
```

需要走代理时（本机实测 HTTP 代理会缓冲大请求体，必须用 SOCKS5）：

```bash
git -c http.proxy=socks5h://127.0.0.1:7897 fetch upstream --prune --tags
```

批量同步多个镜像仓库：使用工作区根目录的 `sync-all-upstream.sh`。
