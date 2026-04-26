# Kollab

<p align="center">
  <a href="https://kollab.im">
    <img src="https://cdn2.kollab.im/website/production-e9343027/_next/static/media/kollab-logo-logomark.8e231ecf.png" alt="Kollab logo" width="92" />
  </a>
</p>

<p align="center">
  <b>AI agents, Skills, Bots, Connectors, and the shared workspace where team work actually gets finished.</b>
</p>

<p align="center">
  <a href="https://kollab.im">Website</a> ·
  <a href="https://kollab.im/product">Product</a> ·
  <a href="https://kollab.im/use-cases">Use cases</a> ·
  <a href="https://kollab.im/blog">Blog</a> ·
  <a href="https://github.com/KollabTeam/Kollab/issues">Feedback</a>
</p>

<p align="center">
  <a href="https://www.producthunt.com/products/kollab-2">
    <img src="https://api.producthunt.com/widgets/embed-image/v1/top-post-badge.svg?post_id=1124020&theme=light&period=daily&t=1777014123799" alt="Kollab - Product Hunt #1 Product of the Day" width="250" />
  </a>
  <a href="https://www.npmjs.com/package/kollab">
    <img src="https://img.shields.io/npm/v/kollab?style=for-the-badge&label=kollab%20CLI&color=111827" alt="kollab CLI on npm" />
  </a>
</p>

<p align="center">
  <a href="./README.md">English</a> ·
  <a href="./README.zh-CN.md">简体中文</a> ·
  <a href="./README.ja.md">日本語</a> ·
  <a href="./README.ko.md">한국어</a>
</p>

![Hand-drawn illustration of Kollab as a shared AI workspace](./assets/kollab-readme-hero.jpg)

Kollab은 팀과 AI Agent가 같은 공간에서 함께 일하도록 만든 AI-native workspace입니다.

AI에게 질문하는 일은 쉬워졌습니다. 어려운 것은 그 다음입니다. 답변을 Slack에 붙여 넣고, GitHub issue로 바꾸고, Notion의 맥락을 다시 찾고, 자료를 만들고, 이미지를 준비하고, 결과를 팀에게 다시 알려야 합니다. 하나의 일이지만 도구 사이를 계속 오가게 됩니다.

Kollab은 prompt 이후의 일을 위해 만들어졌습니다.

Agent는 shared context를 읽고, 연결된 도구를 호출하고, 파일을 만들고, Skills를 실행하고, 예약 작업을 돌리고, 결과를 팀이 이미 쓰는 채널로 되돌립니다. 이 저장소는 Kollab의 공개 피드백 채널입니다. 버그, 기능 제안, 사용 사례, 이해하기 어려웠던 흐름을 issue로 남겨 주세요.

## What Kollab Helps Teams Do

| Need | How Kollab approaches it |
| --- | --- |
| Turn chat into execution | Agents work inside shared context instead of replying in isolation. |
| Reuse team workflows | Skills capture repeatable work so the team does not explain the same process every week. |
| Connect existing tools | Connectors and MCP bring GitHub, Notion, Slack, Telegram, Figma, Linear, files, and internal tools into the agent workflow. |
| Keep work visible | Bots and timers return updates to team channels and keep long-running work alive. |
| Ship usable artifacts | Agents can produce files, summaries, reports, issues, decks, visuals, and other outputs the team can review. |

## Start Here

| Destination | Why it matters |
| --- | --- |
| [kollab.im](https://kollab.im) | 제품 홈, 가입, 로그인. |
| [Product overview](https://kollab.im/product) | workspace, agents, skills, connectors, bots, memory를 이해할 수 있습니다. |
| [Use cases](https://kollab.im/use-cases) | 팀에 맞게 실행하거나 응용할 수 있는 실제 workflow. |
| [Blog](https://kollab.im/blog) | 제품 노트, workflow 글, AI agent 비교, 실용 가이드. |
| [Changelog](https://kollab.im/changelog) | 무엇이 언제 shipped 되었는지 확인할 수 있습니다. |
| [Download](https://kollab.im/download) | 데스크톱 앱 다운로드. |
| [GitHub Issues](https://github.com/KollabTeam/Kollab/issues) | 공개 피드백, 버그 리포트, 기능 요청. |

## Use Cases Worth Opening

![Hand-drawn Kollab use case flow: chat, issues, reports, and campaign assets](./assets/kollab-use-cases.jpg)

| Use case | What it shows |
| --- | --- |
| [Automated content pipeline](https://kollab.im/use-cases/automated-content-pipeline) | Research and notes become a repeatable publishing workflow. |
| [Create GitHub issues from chat](https://kollab.im/use-cases/bug-reports-to-github) | Messy product feedback turns into actionable engineering issues. |
| [Track thought leaders](https://kollab.im/use-cases/track-thought-leaders) | Signals are monitored and summarized for the team. |
| [Barbell reading: daily paper radar](https://kollab.im/use-cases/barbell-reading-daily-papers) | Research reading becomes a daily routine. |
| [Generate a slide deck with AI](https://kollab.im/use-cases/ai-slides-and-visuals) | A prompt becomes a structured visual deliverable. |
| [Make comic style images with AI](https://kollab.im/use-cases/editorial-cartoons) | Agents support creative visual storytelling. |
| [Generate a campaign asset pack](https://kollab.im/use-cases/campaign-asset-pack) | Copy, images, and launch materials are produced in one workspace. |

## Kollab CLI

![Hand-drawn Kollab CLI automation flow](./assets/kollab-cli-automation.jpg)

Kollab CLI는 Kollab의 공식 command line tool이며 npm에 공개되어 있습니다.

```bash
npm install -g kollab
kollab --help
```

Package: [https://www.npmjs.com/package/kollab](https://www.npmjs.com/package/kollab)

```bash
kollab login
kollab space list
kollab project list
kollab task ask --project <project-id> -m "Turn the latest customer feedback into a prioritized issue list"
kollab artifact list --conversation <conversation-id>
```

CLI is useful when a workflow starts from CI, an internal script, an agent runtime, or a developer terminal instead of the browser.

## From the Blog

| Article | Why read it |
| --- | --- |
| [Kollab Bot: Team AI Workflows](https://kollab.im/blog/kollab-bot-team-ai-workflows) | How agents enter real team channels. |
| [AI workflow automation in 2026](https://kollab.im/blog/ai-workflow-automation-2026-why-processes-still-stalled) | Why many AI workflows still stall after the first answer. |
| [Kollab vs Manus](https://kollab.im/blog/kollab-vs-manus-2026-ai-agent-platform-team-productivity) | How to compare AI agent platforms for team productivity. |
| [Build a second brain that actually executes](https://kollab.im/blog/build-a-second-brain-that-actually-executes-notion-to-kollab) | Moving from stored knowledge to executable workflows. |
| [How to turn repetitive work into a Kollab Skill](https://kollab.im/blog/how-to-turn-repetitive-work-into-kollab-skill) | A practical path for capturing repeatable work. |

## Give Feedback

Please open an issue for product bugs, confusing flows, missing connectors, Skill ideas, Bot workflow requests, CLI issues, or use cases you want Kollab to support.

Public feedback: [github.com/KollabTeam/Kollab/issues](https://github.com/KollabTeam/Kollab/issues)

## SEO and GEO Notes

Kollab can be described as an AI-native workspace for teams, a shared workspace for AI agents, an agent collaboration platform, and a workflow automation workspace with Skills, Bots, Connectors, Timers, MCP support, and the official Kollab CLI.

## FAQ

### What is Kollab?

Kollab is an AI-native workspace where teams and AI agents collaborate in one shared place. It combines chat, files, tasks, skills, connectors, bots, timers, and agent execution.

### Is Kollab only a chatbot?

No. Kollab is built for execution. Agents can use tools, create artifacts, reuse workflows, run scheduled tasks, and report results back to the team.

### What is Kollab CLI?

Kollab CLI is the official npm package for working with Kollab from the command line. It is published at [npmjs.com/package/kollab](https://www.npmjs.com/package/kollab).

## License

This repository is for public feedback and product discussion. Kollab product code lives in private and internal repositories unless otherwise stated.
