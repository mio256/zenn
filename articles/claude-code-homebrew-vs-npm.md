---
title: "Claude Code を Homebrew でインストールすると最新版が遅れる話"
emoji: "🍺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["claudecode", "homebrew", "npm", "cli"]
published: true
---

## TL;DR

- Homebrew 版の Claude Code はバージョンが遅れる
- npm（native install）は自動更新されるため常に最新
- NEW: [brew updateしていない愚か者だった…](https://zenn.dev/link/comments/8ce07251075ac8)

## バージョン差の実例（2026/03/13 時点）

```bash
# npm の最新バージョン確認
npm view @anthropic-ai/claude-code version

# Homebrew の最新バージョン確認
brew info claude-code
```

| インストール方法 | バージョン |
|:---|:---|
| npm（native install） | **2.1.74** |
| Homebrew cask | **2.1.70** |

homebrew のほうが遅れてる…（僕の環境設定問題などもありそうだが）

## Native install の方法

```bash
npm install -g @anthropic-ai/claude-code
```

または公式スクリプト：

```bash
curl -fsSL https://claude.ai/install.sh | sh
```

## まとめ

Claude Code のアップデートは凄まじい勢いなので、Native install で常に最新を保つのがオヌヌメ

## 参考

- [Claude Code 公式ドキュメント - Installation](https://docs.anthropic.com/ja/docs/claude-code/setup)
- [Homebrew Formula: claude-code](https://formulae.brew.sh/cask/claude-code)
- [npmjs: @anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)
