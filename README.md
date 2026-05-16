# kubekey-openeuler-iso

本仓库用于通过 GitHub Actions 自动构建openEuler操作系统的离线依赖包 ISO 镜像。这些离线包主要用于 [KubeKey](https://github.com/kubesphere/kubekey) 部署 Kubernetes 集群时，为无法连接公网的内网环境提供预下载的 RPM包。 
其他操作系统的离线依赖ISO镜像请至[KubeKey release](https://github.com/kubesphere/kubekey/releases/tag/iso-latest).

## 产物下载

最新构建产物通过 GitHub Release 自动发布：

**[→ 前往 Release 页面下载](https://github.com/darrentangdt/kubekey-openeuler-iso/releases/tag/iso-latest)**

### 当前发布的 openEuler 离线包（最新构建）

| 版本 | AMD64 (x86_64) | ARM64 (aarch64) |
|------|----------------|-----------------|
| openEuler 22.03 LTS | `openeuler-22.03-lts-rpms-amd64.iso` (52.0 MB) | `openeuler-22.03-lts-rpms-arm64.iso` (50.6 MB) |
| openEuler 22.03 LTS SP1 | `openeuler-22.03-lts-sp1-rpms-amd64.iso` (50.6 MB) | `openeuler-22.03-lts-sp1-rpms-arm64.iso` (49.2 MB) |
| openEuler 22.03 LTS SP2 | `openeuler-22.03-lts-sp2-rpms-amd64.iso` (48.7 MB) | `openeuler-22.03-lts-sp2-rpms-arm64.iso` (47.4 MB) |
| openEuler 22.03 LTS SP3 | `openeuler-22.03-lts-sp3-rpms-amd64.iso` (47.4 MB) | `openeuler-22.03-lts-sp3-rpms-arm64.iso` (46.1 MB) |
| openEuler 22.03 LTS SP4 | `openeuler-22.03-lts-sp4-rpms-amd64.iso` (46.3 MB) | `openeuler-22.03-lts-sp4-rpms-arm64.iso` (44.9 MB) |
| openEuler 24.03 LTS | `openeuler-24.03-lts-rpms-amd64.iso` (57.6 MB) | `openeuler-24.03-lts-rpms-arm64.iso` (55.6 MB) |
| openEuler 24.03 LTS SP1 | `openeuler-24.03-lts-sp1-rpms-amd64.iso` (69.0 MB) | `openeuler-24.03-lts-sp1-rpms-arm64.iso` (66.8 MB) |
| openEuler 24.03 LTS SP2 | `openeuler-24.03-lts-sp2-rpms-amd64.iso` (66.1 MB) | `openeuler-24.03-lts-sp2-rpms-arm64.iso` (64.3 MB) |

每个版本同时附带 `.iso.sha256sum.txt` 校验文件。

> **说明**：
> - ISO 大小因版本不同有所差异，大版本（如 24.03 SP1/SP2）通常包含更多新包
> - 所有 ISO 均包含架构命名后缀（`-amd64` / `-arm64`），请根据目标机器架构下载
> - 首次发布日期：2026-05-16

---

## ISO 文件使用说明
由于kk在openEuler中部署时，仅识别大版本号，比如22.03，24.03，并不关心其sp1、sp2等不同版本号，所以你需要将下载的iso名称中的“lts-sp1-rpms-”删除，重命名为：openEuler-22.03-arm64.iso 或 openEuler-22.03-amd64.iso 这样的文件名。


### 在 KubeKey 中使用

kk自动部署时，会在如下目录寻找对应系统版本的iso
```bash
# openEuler 22.03 arm64（aarch64） 架构
kubekey/repository/arm64/openEuler/22.03/openEuler-22.03-arm64.iso

# openEuler 22.03 amd64（x86_64） 架构
kubekey/repository/amd64/openEuler/22.03/openEuler-22.03-amd64.iso
```
请提前将对应你操作系统版本的iso文件下载并放入对应目录中。

---


## 包列表

各操作系统共用的包列表定义在 [`gen-repository-iso/packages.yaml`](./gen-repository-iso/packages.yaml) 中，此文件从[Kubekey](https://github.com/kubesphere/kubekey)仓库中提取：

- `common`：所有系统共用的基础包（bash-completion, chrony, conntrack, curl, git, haproxy, ipvsadm, keepalived, lvm2 等）
- `rpms`：RPM 系额外包（bind-utils, conntrack-tools, nfs-utils, nss 等）
- `debs`：DEB 系额外包（dnsutils, nfs-common, openssh-server 等）

需要新增或修改包时，直接编辑 `packages.yaml` 后重新触发工作流即可。
