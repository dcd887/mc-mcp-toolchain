# Minecraft MCP Toolchain

一套面向 **Minecraft 整合包开发** 的 MCP (Model Context Protocol) 服务器集群，覆盖从建包、诊断、配置管理、资源包处理到代码生成的完整工作流。

## 架构图

```
                          ┌─────────────────────────────────────────────┐
                          │           Minecraft MCP Toolchain           │
                          │            (6 个 MCP Servers)               │
                          └─────────────────────────────────────────────┘
                                           │
          ┌────────────────────────────────┼────────────────────────────────┐
          │                                │                                │
          ▼                                ▼                                ▼
┌───────────────────┐          ┌───────────────────┐          ┌───────────────────┐
│   🏗️ 建包阶段      │          │   🔍 诊断阶段      │          │   ⚙️ 配置阶段      │
│                   │          │                   │          │                   │
│ mc-pack-builder   │          │ mc-modpack-mcp    │          │ mc-mod-config-mcp │
│ mc-changelog-mcp  │          │                   │          │                   │
└────────┬──────────┘          └────────┬──────────┘          └────────┬──────────┘
         │                              │                              │
         │  整合包结构                    │  运行时问题                  │  配置文件
         │  + Modrinth 数据              │  + 崩溃报告                  │  + .cfg/.toml/.json
         │                              │                              │
         ▼                              ▼                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            🎨 资源包 & 🤖 代码生成                               │
│                                                                                 │
│  mc-rp-assistant-mcp              minecraft-mcp-server (Cloudflare Workers)     │
│  ─────────────────                ───────────────────────────                   │
│  • pack.mcmeta 验证               • FTB Quests 章节 & 任务 (SNBT)                 │
│  • 纹理尺寸 & 命名规范检查          • KubeJS 脚本：武器/盔甲/配方/移除              │
│  • 语言文件缺失翻译检测             • SNBT 语法校验 & KubeJS 错误检查              │
│  • Modrinth 资源包搜索             • 零网络依赖，纯算法生成                        │
└─────────────────────────────────────────────────────────────────────────────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │   AI 诊断 (可选)     │
                              │   mc-modpack-mcp    │
                              │   通义千问 API       │
                              └─────────────────────┘
```

## 项目总览

| 仓库 | 角色 | 核心功能 | 协议 |
|------|------|---------|------|
| [mc-pack-builder-mcp](https://github.com/dcd887/mc-pack-builder-mcp) | 🏗️ 建包 | 创建整合包结构、Modrinth 搜索、依赖解析、兼容性检查 | stdio |
| [mc-changelog-mcp](https://github.com/dcd887/mc-changelog-mcp) | 📝 更新日志 | 比较两个整合包差异、生成 Markdown/文本 Changelog | stdio |
| [mc-modpack-mcp](https://github.com/dcd887/mc-modpack-mcp) | 🔍 诊断 | 冲突检测、日志分析、崩溃报告、AI 智能诊断（通义千问） | stdio |
| [mc-mod-config-mcp](https://github.com/dcd887/mc-mod-config-mcp) | ⚙️ 配置管理 | 读取/验证/对比模组配置文件 (.cfg/.toml/.json)、孤儿检测 | stdio |
| [mc-rp-assistant-mcp](https://github.com/dcd887/mc-rp-assistant-mcp) | 🎨 资源包 | pack.mcmeta 验证、纹理分析、语言文件检查、Modrinth 搜索 | stdio |
| [minecraft-mcp-server](https://github.com/dcd887/minecraft-mcp-server) | 🤖 代码生成 | FTB Quests 任务书、KubeJS 脚本、SNBT 校验、KubeJS 诊断 | HTTP (Cloudflare Workers) |

## 典型工作流

```
1️⃣  用 mc-pack-builder 创建新整合包骨架
        │
2️⃣  用 mc-changelog 跟踪模组版本变化 → 生成更新日志
        │
3️⃣  用 mc-mod-config 管理所有模组的配置文件
        │
4️⃣  用 mc-rp-assistant 检查/制作资源包
        │
5️⃣  运行 mc-modpack 诊断冲突和崩溃
   ├── 本地引擎：冲突规则 + 崩溃分类（无需网络）
   └── 可选 AI：通义千问生成智能诊断报告
        │
6️⃣  用 minecraft-mcp-server 生成 FTB Quests 和 KubeJS 脚本
```

## 技术栈

| 技术 | 用途 |
|------|------|
| TypeScript + Node.js 18+ | 5 个 stdio MCP Server |
| @modelcontextprotocol/sdk v1.x | MCP 协议实现 |
| Cloudflare Workers + Wrangler | minecraft-mcp-server 部署 |
| 通义千问 (DashScope) API | mc-modpack-mcp 可选 AI 诊断 |
| Modrinth API | 模组/资源包元数据查询 |

## 快速开始

所有 stdio 服务器均可通过环境变量方式配置，避免 Windows 路径问题：

```json
{
  "mcpServers": {
    "mc-pack-builder":  { "command": "node", "args": ["${MC_PACK_BUILDER_ROOT}/dist/index.js"] },
    "mc-modpack":       { "command": "node", "args": ["${MCPACK_ROOT}/dist/index.js"] },
    "mc-mod-config":    { "command": "node", "args": ["${MC_MOD_CONFIG_ROOT}/dist/index.js"] },
    "mc-rp-assistant":  { "command": "node", "args": ["${MC_RP_ROOT}/dist/index.js"] },
    "mc-changelog":     { "command": "node", "args": ["${MC_CHANGELOG_ROOT}/dist/index.js"] },
    "minecraft-mod":    { "url": "https://minecraft-mcp.mcjj.workers.dev/mcp" }
  }
}
```

> Windows 用户将 `${VAR_ROOT}` 替换为带正斜杠的绝对路径，例如 `["C:/Users/xxx/Desktop/mc-pack-builder-mcp/dist/index.js"]`。

## 单项目入口

各仓库均可独立安装、构建和使用：

```bash
# 以 mc-modpack-mcp 为例
git clone https://github.com/dcd887/mc-modpack-mcp.git
cd mc-modpack-mcp
npm install
npm run build
node dist/index.js   # 或直接运行诊断：npm run diagnose
```

## License

MIT — 所有子仓库均采用相同许可证。
