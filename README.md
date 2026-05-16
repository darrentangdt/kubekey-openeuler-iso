# kubekey-openeuler-iso

本仓库用于通过 GitHub Actions 自动构建各操作系统（openEuler、AlmaLinux、CentOS、Kylin、Ubuntu、Debian 等）的离线依赖包 ISO 镜像。这些离线包主要用于 [KubeKey](https://github.com/kubesphere/kubekey) 部署 Kubernetes 集群时，为无法连接公网的内网环境提供预下载的 RPM/DEB 包。

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

### 挂载 ISO

```bash
# 创建挂载点
mkdir -p /mnt/iso

# 挂载 ISO 文件
mount -o loop openEuler-XX.XX-xxx.iso /mnt/iso

# 查看包列表
ls -la /mnt/iso/
```

### 配置本地 YUM 源（以 openEuler 为例）

挂载后，ISO 中已包含 `repodata/` 元数据，可直接配置为本地 YUM 源使用：

```bash
# 备份原有 repo 配置
cp /etc/yum.repos.d/backup/ /etc/yum.repos.d/
rm -f /etc/yum.repos.d/*.repo

# 添加本地 ISO 源
cat > /etc/yum.repos.d/local-iso.repo << EOF
[local-iso]
name=Local ISO Repository
baseurl=file:///mnt/iso
enabled=1
gpgcheck=0
EOF

# 清理缓存并使用
yum clean all
yum makecache
```

### 在 KubeKey 中使用

在 KubeKey 配置文件 `config-sample.yaml` 中，可指定离线包来源：

```yaml
spec:
  hosts:
    - {name: node1, address: 192.168.0.1, internalAddress: 192.168.0.1, arch: amd64}
  registry:
    registryMirrors: [ ]
    insecureRegistries: [ ]
  addons: [ ]
```

将 ISO 挂载到节点上并配置好本地源后，KubeKey 部署过程中会优先使用本地仓库中的依赖包。

---

## 项目结构

```
.
├── .github/workflows/
│   └── gen-repository-iso.yaml     # GitHub Actions 工作流定义
├── gen-repository-iso/
│   ├── dockerfile.openeuler*       # openEuler 各版本的 Dockerfile (8 个)
│   ├── dockerfile.kylin*           # 银河麒麟各版本
│   ├── dockerfile.almalinux*       # AlmaLinux
│   ├── dockerfile.centos*          # CentOS
│   ├── dockerfile.debian*          # Debian
│   ├── dockerfile.ubuntu*          # Ubuntu
│   ├── packages.yaml               # 公共包列表定义
│   └── download-pkgs.sh            # DEB 系包下载脚本
├── script/                         # 手动构建参考脚本（已集成到主工作流）
│   ├── Dockerfile.*
│   ├── gen-openeuler-iso.sh
│   └── README
├── iso/
│   └── README.md                   # 构建产物输出目录
└── README.md                       # 本文件
```

---

## 手动触发工作流

除了通过推送 `iso-latest` 标签自动触发外，也可以手动触发：

1. 进入仓库 **Actions** → **GenRepositoryISO**
2. 点击 **Run workflow** → 选择分支 → 点击 **Run workflow**

### 通过命令行触发

```bash
# 删除并重新创建 iso-latest 标签来触发构建
git tag -d iso-latest
git push origin :refs/tags/iso-latest
git tag iso-latest
git push origin iso-latest
```

---

## 本地手动构建（参考）

如果需要在本地测试或构建单个 ISO：

```bash
# 以 openEuler 22.03 LTS 为例
cd gen-repository-iso

# 构建 Docker 镜像
docker build -f dockerfile.openeuler2203 -t iso-builder:openeuler2203 .

# 运行并导出 ISO
docker run --rm -v $(pwd)/../iso:/output iso-builder:openeuler2203
cp /output/*.iso ./
```

---

## 包列表

各操作系统共用的包列表定义在 [`gen-repository-iso/packages.yaml`](./gen-repository-iso/packages.yaml) 中：

- `common`：所有系统共用的基础包（bash-completion, chrony, conntrack, curl, git, haproxy, ipvsadm, keepalived, lvm2 等）
- `rpms`：RPM 系额外包（bind-utils, conntrack-tools, nfs-utils, nss 等）
- `debs`：DEB 系额外包（dnsutils, nfs-common, openssh-server 等）

需要新增或修改包时，直接编辑 `packages.yaml` 后重新触发工作流即可。
