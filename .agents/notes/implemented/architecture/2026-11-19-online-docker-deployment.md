# Agent Note: Docker 镜像在线部署入口

Status: implemented

## Problem

原来的 Compose 服务同时声明本地 `build` 和本地镜像名。部署机必须先获取完整源码，再执行构建，无法直接从镜像仓库拉取并启动已构建的版本。

## Decision

生产 `compose.yaml` 使用 `ghcr.io/peko2r/alimpay:latest` 作为默认镜像，并设置 `pull_policy: always`，使部署命令直接获取仓库发布的镜像。镜像由 `.github/workflows/docker-publish.yml` 在 `main` 推送和 `v*.*.*` 标签时构建并发布，同时生成版本标签和提交 SHA 标签。

本地源码构建移到 `compose.build.yaml` override。部署文档提供从 GitHub Raw 下载 Compose 和环境模板的步骤，并保留 `ALIMPAY_IMAGE` 覆盖入口以便固定版本或使用私有镜像。

## Alternatives considered

直接把源码仓库作为部署入口的优点是无需配置镜像仓库，但每台机器都要安装构建工具并耗费构建时间，无法满足在线拉取后直接运行的目标。

使用 Docker Hub 可以匿名拉取，但需要额外维护 Docker Hub 账号和访问令牌；当前代码已经托管在 GitHub，GHCR 能直接使用仓库的 `GITHUB_TOKEN` 完成发布，减少凭据配置。

## Consequences

部署机只需要 Docker、Compose 和一个可写的数据目录即可启动；发布流程会为 `main` 提供滚动的 `latest`，为版本标签提供可回滚的版本镜像。GHCR 包首次创建后仍需在 GitHub Packages 页面将可见性设为 Public，私有包部署则需要额外的 `docker login`。

本地开发多维护一个 override 文件，但仍可继续从当前源码构建，不会把开发流程绑定到远程镜像。
