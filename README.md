# Remotion x Claude Code: Comprehensive Technical Guide

Programmatic video production using **Remotion** (React) with **Claude Code** as the agentic development interface.

## Overview

This repository contains a technical guide covering the full Remotion + Claude Code workflow — from project scaffolding to cloud-scale rendering. The guide is written for senior developers and creators proficient in Claude Code, agentic AI workflows, and multimedia authoring.

### Topics Covered

- **Architecture** — Four-layer rendering model (Agentic Interface, React Composition Engine, Render Pipeline, Output Formats)
- **Environment Setup** — Project scaffolding, directory structure, Zod-based prop schemas
- **Deterministic Rendering** — Frame-as-state model, why CSS transitions are forbidden
- **Animation System** — `interpolate()`, `spring()`, `interpolateColors()`, `TransitionSeries`
- **Audio & Music Integration** — Spectrum analysis, waveform visualization, audio-reactive animation
- **Composition Patterns** — Scene-per-component, data-driven batch generation, Canvas/WebGL
- **CLAUDE.md Setup** — Prescriptive context for reliable Claude Code output
- **Render Pipeline** — CLI rendering, codec selection, CRF guidelines, SSR API
- **Cloud Scaling** — AWS Lambda, Google Cloud Run, batch rendering
- **Advanced Recipes** — Typewriter effect, particle systems, shapes library, dynamic metadata
- **Troubleshooting** — Common pitfalls, performance tuning, debugging

## Available Languages

| Language | File |
|----------|------|
| English | [english.md](english.md) |
| Korean | [korean.md](korean.md) |

## Prerequisites

- Node.js >= 18
- React + TypeScript fluency
- Familiarity with Claude Code CLI
- Remotion v4.x / v5.x

## Quick Start

Open either guide and follow **Section 2: Environment Setup with Claude Code** to scaffold a new Remotion project.

```bash
npx create-video@latest my-video --template-type typescript
cd my-video
npm install @remotion/media-utils @remotion/tailwind-v4 @remotion/shapes @remotion/transitions
```

## Assets

The [assets/](assets/) directory contains SVG diagrams referenced in the guides:

- `01-architecture-overview.svg` — Four-layer architecture diagram
- `02-animation-cheatsheet.svg` — Animation system reference
- `03-workflow-pipeline.svg` — Render pipeline workflow

## License

Guide content: Technical Guide v1.0 (March 2026)

Remotion itself uses a [Business Source License (BSL 1.1)](https://remotion.dev/license) — free for individuals and teams of 3 or fewer.

## References

- [Remotion Documentation](https://remotion.dev/docs)
- [Remotion GitHub](https://github.com/remotion-dev/remotion)
- [Remotion Templates](https://remotiontemplates.dev)

---

<br><br><br>

# Remotion x Claude Code: 종합 기술 가이드

**Remotion**(React) 기반 프로그래매틱 영상 제작과 **Claude Code**를 에이전틱 개발 인터페이스로 활용하는 방법을 다룹니다.

## 개요

이 저장소는 프로젝트 스캐폴딩부터 클라우드 대규모 렌더링까지, Remotion + Claude Code 워크플로 전체를 아우르는 기술 가이드를 제공합니다. Claude Code, 에이전틱 AI 워크플로, 멀티미디어 제작에 능숙한 시니어 개발자 및 크리에이터를 대상으로 작성되었습니다.

### 다루는 주제

- **아키텍처** — 4계층 렌더링 모델 (에이전틱 인터페이스, React 컴포지션 엔진, 렌더 파이프라인, 출력 포맷)
- **환경 설정** — 프로젝트 스캐폴딩, 디렉터리 구조, Zod 기반 props 스키마
- **결정론적 렌더링** — 프레임-as-상태 모델, CSS 트랜지션이 금지되는 이유
- **애니메이션 시스템** — `interpolate()`, `spring()`, `interpolateColors()`, `TransitionSeries`
- **오디오 & 음악 통합** — 스펙트럼 분석, 파형 시각화, 오디오 반응형 애니메이션
- **컴포지션 패턴** — 씬별 컴포넌트, 데이터 기반 일괄 생성, Canvas/WebGL
- **CLAUDE.md 설정** — Claude Code의 안정적 출력을 위한 컨텍스트 구성
- **렌더 파이프라인** — CLI 렌더링, 코덱 선택, CRF 가이드라인, SSR API
- **클라우드 스케일링** — AWS Lambda, Google Cloud Run, 일괄 렌더링
- **고급 레시피** — 타이프라이터 효과, 파티클 시스템, Shapes 라이브러리, 동적 메타데이터
- **트러블슈팅** — 흔한 실수, 성능 튜닝, 디버깅

## 제공 언어

| 언어 | 파일 |
|------|------|
| 영어 | [english.md](english.md) |
| 한국어 | [korean.md](korean.md) |

## 사전 요구사항

- Node.js >= 18
- React + TypeScript 숙련
- Claude Code CLI 사용 경험
- Remotion v4.x / v5.x

## 빠른 시작

가이드를 열고 **섹션 2: Claude Code를 활용한 환경 설정**을 따라 새 Remotion 프로젝트를 스캐폴딩하세요.

```bash
npx create-video@latest my-video --template-type typescript
cd my-video
npm install @remotion/media-utils @remotion/tailwind-v4 @remotion/shapes @remotion/transitions
```

## 에셋

[assets/](assets/) 디렉터리에 가이드에서 참조하는 SVG 다이어그램이 포함되어 있습니다:

- `01-architecture-overview.svg` — 4계층 아키텍처 다이어그램
- `02-animation-cheatsheet.svg` — 애니메이션 시스템 레퍼런스
- `03-workflow-pipeline.svg` — 렌더 파이프라인 워크플로

## 라이선스

가이드 콘텐츠: Technical Guide v1.0 (2026년 3월)

Remotion 자체는 [Business Source License (BSL 1.1)](https://remotion.dev/license)를 사용하며, 개인 및 3인 이하 팀은 무료입니다.

## 참고 자료

- [Remotion 문서](https://remotion.dev/docs)
- [Remotion GitHub](https://github.com/remotion-dev/remotion)
- [Remotion 템플릿](https://remotiontemplates.dev)
