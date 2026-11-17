# tiss-hello-app

Day 1 的 hello（Flask）。push 到 main 後由 GitHub Actions 自動部署：

```
push main → build image → ghcr.io/rhuei/hello:<commit 前 7 碼>
          → 更新 rHuei/tiss-day2-gitops 的 newTag / VERSION
          → Argo CD 同步到叢集（namespace fg01）
```

## 一次性設定
1. **GitOps repo 寫入權限**：GitHub → Settings → Developer settings → Fine-grained tokens
   - Repository access：只選 `tiss-day2-gitops`
   - Permissions：Contents → Read and write
2. 本 repo → Settings → Secrets and variables → Actions → New repository secret
   - `GITOPS_TOKEN` = 上一步的 token
3. 第一次 build 完成後：GitHub → Packages → `hello` → Package settings → Change visibility → **Public**
   （或在叢集建 ghcr 的 imagePullSecret，見下方）

## 叢集拉 Private image（不想設 Public 時）
```bash
kubectl -n fg01 create secret docker-registry ghcr-pull \
  --docker-server=ghcr.io --docker-username=rHuei --docker-password=<PAT，read:packages>
kubectl -n fg01 patch serviceaccount default -p '{"imagePullSecrets":[{"name":"ghcr-pull"}]}'
```

## 本機測試
```bash
docker build -t hello:dev . && docker run --rm -p 8080:80 hello:dev
curl localhost:8080
```
