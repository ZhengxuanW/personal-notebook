这是为您整理的现代化 HPC 集群搭建简要流程，**严格按照依赖顺序排列**，确保无冲突：

### 第一阶段：管理节点 (Master) 基础设施

1.  **安装操作系统与网络**
    * 操作：安装系统，配置内网/外网/IB静态IP。
    * 工具：**Rocky Linux 9**, `nmcli`
2.  **配置时间同步服务端** (必须最先做)
    * 操作：配置上游同步（阿里云），并允许内网网段同步。
    * 工具：**Chrony**
3.  **配置身份认证与 DNS** (核心)
    * 操作：部署域控，配置 DNS 转发器 (Forwarder)。
    * 工具：**FreeIPA**
4.  **配置 DHCP 与 PXE** (避坑步骤)
    * 操作：配置 DHCP 和 TFTP，**务必禁用 DNS 功能 (`port=0`)** 以免与 FreeIPA 冲突。
    * 工具：**Dnsmasq**, `syslinux`
5.  **配置网络转发 (NAT)**
    * 操作：开启 IP 伪装 (Masquerade)，让计算节点能上网。
    * 工具：**Firewalld**
6.  **配置共享存储服务端**
    * 操作：导出 `/home` 和 `/software` 目录。
    * 工具：**NFS-Utils**

---

### 第二阶段：计算节点 (Compute) 批量装机

7.  **定义节点信息**
    * 操作：绑定 MAC 地址与静态 IP。
    * 工具：**Dnsmasq (配置文件)**
8.  **批量系统安装**
    * 操作：通过 IPMI 远程启动，自动加载应答脚本安装 OS。
    * 工具：**IPMI**, **PXE**, **Kickstart**
    * PS:在这个步骤解决 **SElinux** 以及**公钥访问**

---

### 第三阶段：计算节点统一配置 (使用 Ansible)

9.  **配置时间同步客户端** (第一优先级)
    * 操作：强制指向管理节点同步时间。
    * 工具：**Ansible** + **Chrony**
10. **加入用户域**
    * 操作：接入 FreeIPA，统一 UID/GID。
    * 工具：**Ansible** + **FreeIPA Client**
11. **挂载存储**
    * 操作：配置按需自动挂载 `/home`。
    * 工具：**Ansible** + **Autofs**
12. **配置高速网络**
    * 操作：配置 InfiniBand IP (IPoIB)。
    * 工具：**Ansible** + `nmcli`

---

### 第四阶段：调度与环境

13. **部署调度系统**
    * 操作：安装 Munge 密钥，部署控制端和计算端守护进程，配置 Cgroups。
    * 工具：**Slurm**, **Munge**
14. **构建软件栈**
    * 操作：自动编译编译器、MPI、数学库及应用软件。
    * 工具：**Spack**, **CMake**
15. **配置环境变量**
    * 操作：生成层级化的模块文件。
    * 工具：**Lmod**

---

### 第五阶段：监控与可视化

16. **部署监控系统**
    * 操作：管理节点跑数据库和面板，计算节点跑采集器。
    * 工具：**Docker Compose** (Server), **Prometheus**, **Grafana**, **Node/DCGM Exporter**