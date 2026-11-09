# Sidecar

**Ask the talk anything, live. The people stuck on the same thing become your hallway track.**

The best part of a conference is the people, but starting a conversation with a stranger is hard, and most hallway chats are forgettable. Sidecar introduces you to people because you share a real problem, so **every introduction comes with the conversation already started**.

Built at the Cloud Native Hackathon (WeMakeDevs, Salt Lake City, Nov 9 2026). Theme: *make conferences better.*

## What it does
- **Plan my day:** a schedule built from what you're working on (GitHub + profile), your goals, level and pace. Every pick explains why.
- **Live talk:** scan the room's QR code → live captions → ask questions privately, answered from what the speaker *just* said and from every other talk at the conference.
- **Connect:** blind profiles of people stuck on the *same problem* (not just the same keywords). Mutual "want to meet" reveals names and suggests where to meet.
- **Across rooms:** "Room B is talking about your topic right now."
- **Recap:** an auto-built trip report: takeaways tied to your work, people you met and why, open questions.
- **Speaker view:** where the room got lost, minute by minute.

## Docs (start here)
| Doc | What's in it |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | **The tech plan.** Stack, deployment on the Radar cluster, ES data model, API, flows, observability, CI and review, build order |
| [docs/PLAN.md](docs/PLAN.md) | **The build plan.** Product flow, design decisions, ES personalization, demo script, architecture, team split, open questions |
| [docs/HACKATHON_REQUIREMENTS.md](docs/HACKATHON_REQUIREMENTS.md) | Rules, tracks, judging criteria |
| [docs/research/all-conferences-problems.md](docs/research/all-conferences-problems.md) | Researched problems across all conference types, with sources |
| [docs/research/kubecon-problems.md](docs/research/kubecon-problems.md) | KubeCon-specific complaints, with sources |
| [docs/IDEAS.md](docs/IDEAS.md) | Brainstorm history and alternatives considered |

## Built with
Elasticsearch (core data layer: hybrid search, semantic matching, percolator alerts, aggregations) · Dash0 (OpenTelemetry traces/logs/metrics, RUM, SLOs, Agent0) · Radar (Kubernetes UI + AI SRE, Argo CD deploys) · Gitar (AI code review on every PR) · OpenAI

## Setup
_TBD: filled in as services land._

## Architecture
See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## AI usage disclosure
This project was built with AI coding assistants (Claude Code and others), as allowed by the hackathon rules. AI was used for research, planning, and code generation. All code is reviewed, understood and tested by the team, and every feature lands through a reviewed pull request.

## Team
_TBD_
