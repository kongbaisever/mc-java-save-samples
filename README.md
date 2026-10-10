# Minecraft Java 版存档结构样本（跨版本对照）

本仓库收集了 6 个代表性 Minecraft Java 版的**全新默认世界存档**，用于研究不同版本之间的存档（世界）格式差异，辅助实现地图编辑器 / 世界读写工具。

每个世界均由对应版本的官方服务端生成（默认种子、默认地形、出生点附近区块），未做任何人工修改。

## 版本列表

| 目录 | 版本 | 发布时间 | 服务端来源 |
|---|---|---|---|
| `saves/b1.3/` | Beta 1.3_01 | 2011-02 | Betacraft 归档（Mojang 未托管服务端） |
| `saves/1.2.1/` | Release 1.2.1 | 2012-02 | Betacraft 归档（Mojang 未托管服务端） |
| `saves/1.13/` | Release 1.13（Aquatic 更新） | 2018-07 | BMCLAPI 镜像 |
| `saves/1.16/` | Release 1.16（下界更新） | 2020-06 | BMCLAPI 镜像 |
| `saves/1.18/` | Release 1.18（洞穴与山崖 Part 2） | 2021-11 | BMCLAPI 镜像 |
| `saves/1.20.5/` | Release 1.20.5 | 2024-04 | BMCLAPI 镜像 |

## 顶层目录结构差异

一个世界存档根目录的内容随版本演进如下：

```
b1.3/
├── level.dat            # 世界元数据（NBT, gzip），仅 ~159 字节
├── level.dat_old        # 上一次备份
├── session.lock
├── players/            # 玩家数据
└── region/             # r.*.mcr  ← 旧 Region 格式

1.2.1/
├── level.dat
├── level.dat_old
├── session.lock
├── data/               # 地图、遗迹信息等
├── DIM-1/              # 下界（Nether）
├── DIM1/               # 末地（End）
├── players/
└── region/             # r.*.mca  ← 已是 Anvil 格式

1.13/
├── level.dat
├── level.dat_old
├── session.lock
├── data/
├── datapacks/          # ← 1.13 新增（数据包）
├── DIM-1/
├── DIM1/
├── playerdata/         # ← 由 players/ 改名而来
└── region/             # r.*.mca

1.16/
├── ...（同 1.13）
├── poi/                # ← 1.14+ 新增（Points of Interest：床、工作方块等）
└── region/

1.18/
├── ...（同 1.16）
├── entities/           # ← 1.18 新增（实体从区块文件中拆分出来独立存储）
└── region/

1.20.5/
├── ...（同 1.18）
└── region/
```

## 关键格式变迁点

### 1. Region 文件格式：`.mcr` → `.mca`
- **b1.3** 使用旧的 **Region 格式**，区块文件为 `region/r.X.Z.mcr`。
- **1.2.1 起** 切换到 **Anvil 格式**，区块文件为 `region/r.X.Z.mca`（Zlib 压缩的 NBT，结构兼容但区块内数据布局不同）。
- 编辑器必须按版本选择正确的解析器。

### 2. 维度目录
- b1.3：只有主世界，未访问下界 / 末地时不生成 `DIM-1` / `DIM1`。
- 1.2.1 及以后：`DIM-1`（下界）、`DIM1`（末地）随服务端启动即生成空壳目录。
- 每个维度有自己的 `region/`、`entities/`、`poi/` 子目录。

### 3. 实体存储位置
- **1.17 及以前**：实体数据内嵌在区块 NBT 的 `Entities` 列表中，与方块数据同处一个 `.mca`。
- **1.18 起**：实体拆出到独立的 `entities/r.X.Z.mca` 文件。

### 4. `level.dat`
- 始终为 gzip 压缩的 NBT 格式。
- 体积随版本增长：b1.3 ~159B → 1.2.1 ~245B → 1.13 ~925B → 1.18 ~1.2KB → 1.20.5 ~1.5KB。
- 内含世界规则、生成器设置、游戏时间、出生点、版本 ID（`Version` 子节点，含 `Id` / `Name` / `Snapshot`）等。

### 5. 其他目录
- `data/`：存放地图（`map_*.dat`）、遗迹/结构标记、村庄信息等。
- `datapacks/`：1.13 引入，自定义战利品/函数/世界生成。
- `playerdata/`（旧称 `players/`）：每个已登录玩家一份 `<uuid>.dat`。
- `session.lock`：文件锁，运行时占用，可忽略。

## 生成方式（可复现）

```bash
# 以 1.20.5 为例（需 Java 21）
mkdir run && cd run
echo "eula=true" > eula.txt
cat > server.properties <<'EOF'
online-mode=false
level-name=world
view-distance=5
EOF
java -Xmx600M -jar server.jar nogui
# 等待出现 "Done (" 后输入 stop
```

各版本所需 JDK：
- b1.3 / 1.2.1 / 1.13 / 1.16 → Java 8
- 1.18 → Java 17
- 1.20.5 → Java 21

## 服务端下载镜像

- 新版本服务端元数据与 jar：BMCLAPI（`https://bmclapi2.bangbang93.com`）
- 旧版本（1.2.1 及更早）服务端：Betacraft 归档（`https://files.betacraft.uk/server-archive/`），Mojang 现行 manifest 不含这些老服务端。
