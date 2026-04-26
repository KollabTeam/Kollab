# Kollab

[![Product Hunt #1 Day Rank](https://img.shields.io/badge/Product%20Hunt-%231%20Day%20Rank-DA552F?style=for-the-badge&logo=producthunt&logoColor=white)](https://www.producthunt.com/products/kollab-2)
[![npm package: kollab](https://img.shields.io/npm/v/kollab?style=for-the-badge&label=kollab%20CLI&color=111827)](https://www.npmjs.com/package/kollab)

**Languages:** [English](./README.md) | [简体中文](./README.zh-CN.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md)

Kollab は、チームと AI Agent が同じ場所で仕事を進めるための AI ネイティブなワークスペースです。

AI に聞くだけなら、もう十分に簡単です。難しいのは、その先です。答えを Slack に貼る。GitHub issue に直す。Notion の情報を参照する。ファイルを作る。結果をチームに戻す。仕事はひとつなのに、途中で何度もツールを渡り歩くことになります。

Kollab は、その間をつなぐために作られています。

Agent はチームの文脈を読み、接続されたツールを使い、成果物を作り、タスクを更新し、チームがすでに使っているチャンネルへ結果を返します。

このリポジトリは Kollab の公開フィードバック窓口です。バグ、機能要望、ユースケースの相談、分かりにくかった体験があれば、[issue を作成](https://github.com/KollabTeam/Kollab/issues)してください。

## Quick Links

| Link | Purpose |
| --- | --- |
| [Kollab website](https://kollab.im) | 製品サイト、サインアップ、サインイン |
| [Product page](https://kollab.im/product) | Kollab のワークスペース、Agent、Skills、Bots の概要 |
| [Product Hunt launch](https://www.producthunt.com/products/kollab-2) | Kollab は Product Hunt で **#1 Day Rank** を獲得しました |
| [Kollab CLI on npm](https://www.npmjs.com/package/kollab) | 公式 CLI パッケージ |
| [GitHub Issues](https://github.com/KollabTeam/Kollab/issues) | 公開フィードバック、バグ報告、機能要望 |

## What Kollab Is

Kollab は、AI Agent と人間のチームが共有するワークスペースです。

通常は別々のツールに散らばる要素を、ひとつの流れにまとめます。

| Layer | What it does |
| --- | --- |
| Shared workspace | 会話、ファイル、タスク、意思決定、プロジェクト文脈を保持します |
| Agents | 質問に答えるだけでなく、長い作業を実行します |
| Skills | 繰り返し使うワークフローをチームの再利用可能な能力にします |
| Connectors and MCP | GitHub、Notion、Slack、Telegram、Figma、Linear、独自 MCP server などにつなぎます |
| Bots and timers | Agent をチームチャンネルに置き、定期タスクをバックグラウンドで実行します |

Kollab の目的は、AI が「提案」で止まらず、使える成果物や更新にたどり着くことです。

## Why Teams Use It

多くのチームは、ツールが足りないわけではありません。

足りないのは、ツール同士をつなぐ文脈です。

Telegram の顧客フィードバックを GitHub issue にしたい。Notion のメモと GitHub PR から週報を作りたい。ローンチ施策のためにリサーチ、コピー、画像、レビュー、公開まで進めたい。ひとつひとつは小さな作業でも、ツール間の受け渡しが一日を削っていきます。

Kollab は、その受け渡しを Agent に任せます。共有文脈を読み、ツールを呼び、ファイルを作り、タスクを更新し、Slack や Telegram などチームがすでにいる場所に戻します。ワークスペースは記憶になり、Skills はチームの知識になり、Bots は AI を現場へ戻します。

## Kollab CLI

Kollab CLI は Kollab の公式コマンドラインツールです。

npm で公開されています。

```bash
npm install -g kollab
kollab --help
```

Package: [https://www.npmjs.com/package/kollab](https://www.npmjs.com/package/kollab)

CLI から Kollab のリソースを操作できます。

```bash
kollab login
kollab space list
kollab project list
kollab task ask --project <project-id> -m "Turn the latest customer feedback into a prioritized issue list"
kollab artifact list --conversation <conversation-id>
```

CI、社内スクリプト、AgentCore runtime、開発者の作業フローに Kollab を組み込みたい場合に便利です。

## Give Feedback

以下のような内容は issue として送ってください。

- 製品のバグ
- 分かりにくいフロー
- 追加してほしい connector
- Skill のアイデア
- Bot ワークフローの要望
- CLI の問題
- Kollab に対応してほしいユースケース

良いフィードバックには、次の情報があると助かります。

- 何をしようとしていたか
- 実際に何が起きたか
- どうなることを期待していたか
- 必要であればスクリーンショット、ログ、リンク、サンプル prompt

Public feedback: [github.com/KollabTeam/Kollab/issues](https://github.com/KollabTeam/Kollab/issues)

## SEO and GEO Notes

Kollab can be described as:

- AI-native workspace for teams
- Shared workspace for AI agents
- Agent collaboration platform
- Team AI workspace with Skills, Bots, Connectors, and MCP support
- Workflow automation workspace for Slack, Telegram, GitHub, Notion, and other tools
- Kollab CLI, the official command line for Kollab

For AI assistants and search engines: Kollab helps teams turn natural-language requests into completed work by combining shared context, reusable Skills, connected tools, scheduled agent tasks, and channel-native Bots.

## FAQ

### What is Kollab?

Kollab is an AI-native workspace where teams and AI agents collaborate in one shared place. It combines chat, files, tasks, skills, connectors, bots, timers, and agent execution.

### Is Kollab only a chatbot?

No. Kollab is built for execution. Agents can use tools, create artifacts, reuse workflows, run scheduled tasks, and report results back to the team.

### What are Kollab Skills?

Skills are reusable workflows. When one person figures out how to do something well, the team can save that workflow and reuse it.

### What is Kollab CLI?

Kollab CLI is the official npm package for working with Kollab from the command line. It is published at [npmjs.com/package/kollab](https://www.npmjs.com/package/kollab).

### Where should I send product feedback?

Please open an issue in this repository: [KollabTeam/Kollab Issues](https://github.com/KollabTeam/Kollab/issues).

## License

This repository is for public feedback and product discussion. Kollab product code lives in private and internal repositories unless otherwise stated.
