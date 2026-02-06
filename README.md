# Geosite2Domain

一个从 v2fly/domain-list-community 仓库生成和维护组织化域名列表的数据仓库。本项目处理并分类域名数据，用于网络过滤、代理配置和防火墙规则。

## 📁 仓库结构

```
Geosite2Domain/
├── category/           # 分类域名列表 (104 个文件)
│   ├── category-ads-all.txt
│   ├── category-ai-!cn.txt
│   └── ...
├── non-category/       # 服务域名列表 (1,343 个文件)
│   ├── 115.txt
│   ├── 18comic.txt
│   └── ...
├── .github/
│   └── workflows/
│       └── main.yml   # CI/CD 工作流
└── AGENTS.md          # 编码代理指南
```

## 🚀 项目功能

- **域名来源**: [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community)
- **分类分离**: 将域名分为基于类别和基于服务的列表
- **格式化**: 为网络工具和过滤系统格式化域名
- **自动更新**: 通过 GitHub Actions 每日自动更新两次

## 📋 域名列表格式

所有域名条目遵循以下格式之一：

- `domain:example.com` - 标准域名条目
- `domain:example.com:@ads` - 广告相关域名
- `full:example.com/path` - 完整 URL 路径
- `regexp:^pattern$` - 正则表达式模式

## 🔄 自动更新

本仓库通过 GitHub Actions 自动更新：
- **更新时间**: UTC+8 时间每日 04:00 和 16:00 各一次
- **数据源**: v2fly/domain-list-community 仓库
- **处理流程**: 克隆 → 处理 → 导出 → 提交

## 🛠️ 手动构建过程

如需手动生成域名列表：

```bash
# 克隆源仓库
git clone https://github.com/v2fly/domain-list-community.git

# 处理分类域名列表
cd domain-list-community
export EXPORT_LISTS=$(ls -1 data | grep 'category-' | tr '\n' ',' | sed 's/,$//')
go mod download
go run ./ --datapath=./data --outputdir=../category --exportlists=${EXPORT_LISTS}

# 处理非分类域名列表
export EXPORT_LISTS=$(ls -1 data | grep -v 'category-' | tr '\n' ',' | sed 's/,$//')
go run ./ --datapath=./data --outputdir=../non-category --exportlists=${EXPORT_LISTS}

# 清理
cd ..
rm -rf domain-list-community
```

## 📂 文件命名约定

### 分类文件
- 格式: `category-[名称].txt`
- 示例: `category-ads-all.txt`, `category-ai-!cn.txt`
- 后缀: `-cn` (中国), `-!cn` (非中国), `-jp` 等

### 服务文件
- 格式: `[服务名].txt`
- 示例: `115.txt`, `18comic.txt`, `github.txt`

## 🎯 使用场景

这些域名列表适用于：
- **网络过滤** 系统
- **代理配置** (Clash, V2Ray 等)
- **防火墙规则** 和屏蔽
- **DNS 过滤** 解决方案

## ✅ 验证

验证域名列表格式：

```bash
# 检查分类文件
for file in category/*.txt; do
  grep -E '^(domain:|full:|regexp:)' "$file" > /dev/null || echo "格式错误: $file"
done

# 检查非分类文件
for file in non-category/*.txt; do
  grep -E '^(domain:|full:|regexp:)' "$file" > /dev/null || echo "格式错误: $file"
done
```

## 📊 统计信息

- **分类文件**: 104 个
- **非分类文件**: 1,343 个
- **更新频率**: 每日两次
- **数据源**: v2fly/domain-list-community

## 📄 许可证

本项目采用 Apache License 2.0 with Commons Clause v1.0 许可证。

## 🔗 相关资源

- [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community) - 源域名数据
- [v2ray/domain-list-community](https://github.com/v2ray/domain-list-community) - 备用数据源

## 🤝 贡献

这主要是一个自动化数据仓库。如需贡献：
- 报告域名分类问题
- 建议处理工作流改进
- 帮助验证和测试

---

*自动从 v2fly/domain-list-community 生成*