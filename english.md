# Remotion × Claude Code: Comprehensive Technical Guide

**Programmatic Video Production for Advanced Practitioners**

> **Audience**: Senior developers and creators proficient in Claude Code, agentic AI workflows, software engineering, music production, and multimedia authoring.
>
> **Remotion version**: v4.x / v5.x (latest stable: `4.0.435` as of March 2026)
>
> **Prerequisites**: Node.js ≥ 18, React + TypeScript fluency, familiarity with Claude Code CLI

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Environment Setup with Claude Code](#2-environment-setup-with-claude-code)
3. [Core Concepts — The Deterministic Rendering Model](#3-core-concepts--the-deterministic-rendering-model)
4. [Animation System Deep Dive](#4-animation-system-deep-dive)
5. [Audio & Music Integration](#5-audio--music-integration)
6. [Composition Patterns for Complex Productions](#6-composition-patterns-for-complex-productions)
7. [CLAUDE.md — Prescriptive Context for Remotion Projects](#7-claudemd--prescriptive-context-for-remotion-projects)
8. [Render Pipeline & Output Optimization](#8-render-pipeline--output-optimization)
9. [Scaling: Lambda, Cloud Run, and Batch Rendering](#9-scaling-lambda-cloud-run-and-batch-rendering)
10. [Advanced Recipes](#10-advanced-recipes)
11. [Troubleshooting & Performance Tuning](#11-troubleshooting--performance-tuning)
12. [Licensing](#12-licensing)
13. [References](#13-references)

---

## 1. Architecture Overview

Remotion treats video as a **function of frames over time**, where each frame is a React component render captured by headless Chrome and stitched into a video by a built-in Rust-accelerated FFmpeg binary. Claude Code sits atop this stack as the agentic interface, translating natural-language intent into deterministic React compositions.

![Architecture Overview](assets/01-architecture-overview.svg)

The four-layer architecture:

**Layer 1 — Agentic Interface (Claude Code)**: You describe scenes, transitions, data bindings, and audio sync requirements in natural language. Claude Code generates TypeScript components, Zod prop schemas, composition registrations, and render scripts. The `CLAUDE.md` file provides prescriptive context so the agent follows project conventions without re-prompting.

**Layer 2 — React Composition Engine**: Standard React with Remotion's temporal primitives. Every animation is driven by `useCurrentFrame()` — there are no CSS transitions, no `requestAnimationFrame` calls, no wall-clock dependencies. This is what makes renders deterministic across machines and environments.

**Layer 3 — Render Pipeline**: Webpack (or experimental Rspack) bundles the project. Headless Chrome Shell opens browser tabs (configurable concurrency), renders each frame as JPEG/PNG, and Remotion's Rust-FFmpeg binary encodes the output. Parallel encoding is used when memory allows.

**Layer 4 — Output Formats**: H.264, H.265, VP8/WebM, ProRes, GIF, PNG sequences, and audio-only (AAC/WAV/MP3). Hardware-accelerated encoding is available on supported platforms.

---

## 2. Environment Setup with Claude Code

### 2.1 Project Scaffolding

The fastest path is to have Claude Code scaffold a Remotion project in a single prompt:

```
Claude Code prompt:
"Initialize a new Remotion project in ./my-video with TypeScript strict mode,
TailwindCSS v4 via @remotion/tailwind-v4, and the @remotion/media-utils package
for audio visualization. Register a single composition called 'Main' at 1920x1080,
30fps, 10 seconds. Set up the project structure with src/compositions/, src/components/,
and public/ directories."
```

Under the hood, this executes:

```bash
npx create-video@latest my-video --template-type typescript
cd my-video
npm install @remotion/media-utils @remotion/tailwind-v4 @remotion/shapes @remotion/transitions
```

### 2.2 Project Structure

A well-organized Remotion project follows this layout:

```
my-video/
├── CLAUDE.md                    # Prescriptive context for Claude Code
├── remotion.config.ts           # Render configuration
├── src/
│   ├── Root.tsx                 # Composition registry
│   ├── compositions/
│   │   ├── Main.tsx             # Primary composition
│   │   ├── Intro.tsx            # Scene components
│   │   └── Outro.tsx
│   ├── components/
│   │   ├── AnimatedTitle.tsx    # Reusable animated elements
│   │   ├── AudioVisualizer.tsx
│   │   └── TransitionWipe.tsx
│   ├── hooks/
│   │   └── useAudioSpectrum.ts  # Custom hooks
│   ├── lib/
│   │   ├── animations.ts        # Shared animation utilities
│   │   ├── colors.ts            # Design tokens
│   │   └── timing.ts            # Duration/frame calculations
│   └── schemas/
│       └── props.ts             # Zod schemas for input props
├── public/
│   ├── music/                   # Audio assets
│   ├── fonts/                   # Custom fonts
│   └── images/                  # Static images
└── out/                         # Rendered output
```

### 2.3 Root.tsx — The Composition Registry

```tsx
// src/Root.tsx
import { Composition } from "remotion";
import { Main } from "./compositions/Main";
import { mainSchema, type MainProps } from "./schemas/props";

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="Main"
        component={Main}
        durationInFrames={300} // 10s at 30fps
        fps={30}
        width={1920}
        height={1080}
        schema={mainSchema}
        defaultProps={{
          title: "Hello Remotion",
          accentColor: "#61dafb",
          musicFile: "music/track.mp3",
        }}
      />
    </>
  );
};
```

### 2.4 Zod Schemas for Type-Safe Props

Props defined with Zod become editable in the Remotion Studio GUI — a powerful feedback loop when iterating with Claude Code:

```tsx
// src/schemas/props.ts
import { z } from "zod";

export const mainSchema = z.object({
  title: z.string().describe("Main title text"),
  accentColor: z.string().regex(/^#[0-9a-fA-F]{6}$/).describe("Accent color hex"),
  musicFile: z.string().describe("Path to music file in public/"),
});

export type MainProps = z.infer<typeof mainSchema>;
```

---

## 3. Core Concepts — The Deterministic Rendering Model

### 3.1 Frame as State

Remotion's fundamental insight: **every frame is a pure function of the frame number**. There is no mutable state that accumulates across frames. When Chrome captures frame 150, it renders *only* frame 150 — frames 0-149 never execute. This is why CSS transitions, `setTimeout`, and `requestAnimationFrame` are forbidden.

```tsx
import { useCurrentFrame, useVideoConfig, AbsoluteFill } from "remotion";

export const MyScene: React.FC = () => {
  const frame = useCurrentFrame();             // 0, 1, 2, ... durationInFrames-1
  const { fps, width, height } = useVideoConfig();
  const seconds = frame / fps;                 // Continuous time

  return (
    <AbsoluteFill style={{ backgroundColor: "#0a0a0a" }}>
      <div style={{
        transform: `translateX(${frame * 2}px)`,
        opacity: Math.min(1, frame / 30),
      }}>
        Frame {frame} — {seconds.toFixed(2)}s
      </div>
    </AbsoluteFill>
  );
};
```

### 3.2 Why Determinism Matters

This property enables parallel rendering: Remotion can open N browser tabs and assign each a disjoint set of frames. Frame 500 renders identically whether computed first, last, or in isolation. This is what makes Lambda/Cloud Run distribution possible — each serverless function renders a chunk independently, and the results merge perfectly.

### 3.3 The Composition Model

```
┌─ Composition ──────────────────────────────────────────────────────┐
│  id="Main"  fps=30  width=1920  height=1080  duration=300 frames   │
│                                                                     │
│  ┌─ Sequence from=0 ──────────┐  ┌─ Sequence from=90 ──────────┐  │
│  │  <Intro />                  │  │  <ContentScene />            │  │
│  │  durationInFrames=90        │  │  durationInFrames=150        │  │
│  │  (0s → 3s)                  │  │  (3s → 8s)                   │  │
│  └─────────────────────────────┘  └──────────────────────────────┘  │
│                                                                     │
│  ┌─ Sequence from=240 ─────────┐                                   │
│  │  <Outro />                   │                                   │
│  │  durationInFrames=60         │                                   │
│  │  (8s → 10s)                  │                                   │
│  └──────────────────────────────┘                                   │
└─────────────────────────────────────────────────────────────────────┘
```

Key insight: **inside a `<Sequence>`, `useCurrentFrame()` resets to 0**. A scene at `from=90` sees its own frame 0 when the global timeline is at frame 90. This makes scenes composable and reorderable without rewiring animation timings.

---

## 4. Animation System Deep Dive

![Animation System Cheatsheet](assets/02-animation-cheatsheet.svg)

### 4.1 `interpolate()` — The Workhorse

Maps a numeric input range to an output range. Think of it as a programmable keyframe system:

```tsx
import { interpolate, Easing } from "remotion";

const frame = useCurrentFrame();

// Linear fade-in over 30 frames
const opacity = interpolate(frame, [0, 30], [0, 1], {
  extrapolateRight: "clamp",  // Don't exceed 1.0
});

// Ease-in-out slide from left
const translateX = interpolate(frame, [0, 60], [-200, 0], {
  extrapolateLeft: "clamp",
  extrapolateRight: "clamp",
  easing: Easing.bezier(0.25, 0.1, 0.25, 1),
});

// Multi-keyframe: appear, hold, disappear
const scale = interpolate(
  frame,
  [0,   15,  45,  60],   // keyframe positions
  [0,   1,   1,   0],    // values at each keyframe
  { extrapolateLeft: "clamp", extrapolateRight: "clamp" }
);
```

### 4.2 `spring()` — Physics-Based Motion

Returns a value that animates from `from` to `to` with configurable mass, damping, and stiffness. The overshoot creates natural, organic motion:

```tsx
import { spring, useCurrentFrame, useVideoConfig } from "remotion";

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

const scale = spring({
  frame,
  fps,
  from: 0,
  to: 1,
  config: {
    mass: 0.8,       // Lighter = faster
    damping: 12,     // Higher = less bounce
    stiffness: 200,  // Higher = snappier
  },
});

// Delayed spring (starts at frame 20)
const delayedScale = spring({
  frame: frame - 20,  // Negative frames return `from` value
  fps,
});
```

**Pro tip**: Use `measureSpring()` to calculate exact duration, then use it to set `durationInFrames` on sequences for pixel-perfect timing.

### 4.3 `interpolateColors()` — Smooth Color Transitions

```tsx
import { interpolateColors } from "remotion";

const color = interpolateColors(
  frame,
  [0, 30, 60, 90],
  ["#ff6b6b", "#feca57", "#48dbfb", "#ff9ff3"]
);
```

### 4.4 Composition Primitives — Stacking Temporal Layers

```tsx
import { Series, Sequence, Loop, Freeze, AbsoluteFill } from "remotion";

// Series: sequential playback (each child follows the previous)
<Series>
  <Series.Sequence durationInFrames={90}>
    <IntroScene />
  </Series.Sequence>
  <Series.Sequence durationInFrames={150}>
    <MainContent />
  </Series.Sequence>
  <Series.Sequence durationInFrames={60}>
    <OutroScene />
  </Series.Sequence>
</Series>

// Loop: repeat content
<Loop durationInFrames={30} times={4}>
  <PulseEffect />
</Loop>

// Freeze: hold a specific frame
<Freeze frame={0}>
  <AnimatedTitle />  {/* Frozen at its first frame */}
</Freeze>
```

### 4.5 TransitionSeries — Scene Transitions

```tsx
import { TransitionSeries, linearTiming } from "@remotion/transitions";
import { slide } from "@remotion/transitions/slide";
import { fade } from "@remotion/transitions/fade";

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={90}>
    <SceneA />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={slide({ direction: "from-right" })}
    timing={linearTiming({ durationInFrames: 15 })}
  />
  <TransitionSeries.Sequence durationInFrames={120}>
    <SceneB />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={fade()}
    timing={linearTiming({ durationInFrames: 20 })}
  />
  <TransitionSeries.Sequence durationInFrames={90}>
    <SceneC />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

---

## 5. Audio & Music Integration

For practitioners with music production backgrounds, Remotion's audio pipeline offers frame-accurate synchronization — no drift, no sync issues, no timing hacks.

### 5.1 Basic Audio Embedding

```tsx
import { Html5Audio, staticFile, Sequence } from "remotion";

// Full-track audio
<Html5Audio src={staticFile("music/track.mp3")} />

// Volume envelope (fade in/out)
const frame = useCurrentFrame();
const { durationInFrames } = useVideoConfig();

const volume = interpolate(
  frame,
  [0, 30, durationInFrames - 30, durationInFrames],
  [0, 1, 1, 0],
  { extrapolateLeft: "clamp", extrapolateRight: "clamp" }
);

<Html5Audio src={staticFile("music/track.mp3")} volume={volume} />
```

### 5.2 Audio Visualization — Spectrum Analysis

The `@remotion/media-utils` package provides FFT-based spectrum analysis that runs per-frame:

```tsx
import { useAudioData, visualizeAudio } from "@remotion/media-utils";
import { useCurrentFrame, useVideoConfig, staticFile, Html5Audio } from "remotion";

const music = staticFile("music/track.mp3");

export const SpectrumVisualizer: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();
  const audioData = useAudioData(music);

  if (!audioData) return null;

  const spectrum = visualizeAudio({
    fps,
    frame,
    audioData,
    numberOfSamples: 64,   // Power of 2: 16, 32, 64, 128...
    smoothing: true,        // Average with adjacent frames
  });
  // spectrum: number[] — each value 0..1
  // Index 0 = lowest freq (bass), last index = highest freq (treble)

  const barWidth = width / spectrum.length;

  return (
    <AbsoluteFill style={{ backgroundColor: "#0a0a0a" }}>
      <Html5Audio src={music} />
      <svg viewBox={`0 0 ${width} ${height}`}>
        {spectrum.map((amplitude, i) => {
          const barHeight = amplitude * height * 0.8;
          const hue = (i / spectrum.length) * 360;
          return (
            <rect
              key={i}
              x={i * barWidth}
              y={height - barHeight}
              width={barWidth - 2}
              height={barHeight}
              fill={`hsl(${hue}, 80%, 60%)`}
              rx={2}
            />
          );
        })}
      </svg>
    </AbsoluteFill>
  );
};
```

### 5.3 Waveform Visualization (Voice/Podcast)

For voice-driven content, `visualizeAudioWaveform()` provides time-domain data:

```tsx
import { visualizeAudioWaveform } from "@remotion/media-utils";

const waveform = visualizeAudioWaveform({
  fps,
  frame,
  audioData,
  numberOfSamples: 128,
  windowInSeconds: 1 / fps,  // One frame's worth of audio
});
// waveform: number[] — values between -1 and 1
```

### 5.4 Audio-Reactive Animation Pattern

The most powerful technique: driving visual parameters from audio data:

```tsx
export const AudioReactiveScene: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const audioData = useAudioData(staticFile("music/track.mp3"));

  if (!audioData) return null;

  const spectrum = visualizeAudio({ fps, frame, audioData, numberOfSamples: 8 });

  // Extract frequency bands
  const bass = spectrum[0] + spectrum[1];       // Low frequencies
  const mids = spectrum[2] + spectrum[3];       // Mid frequencies
  const highs = spectrum[6] + spectrum[7];      // High frequencies

  // Drive visual parameters from audio
  const circleScale = 1 + bass * 2;
  const rotation = mids * 45;
  const glowIntensity = highs * 30;

  return (
    <AbsoluteFill style={{
      backgroundColor: "#0a0a0a",
      justifyContent: "center",
      alignItems: "center",
    }}>
      <div style={{
        width: 300,
        height: 300,
        borderRadius: "50%",
        background: `radial-gradient(circle, #61dafb, #1a1a3e)`,
        transform: `scale(${circleScale}) rotate(${rotation}deg)`,
        boxShadow: `0 0 ${glowIntensity}px ${glowIntensity / 2}px rgba(97, 218, 251, 0.5)`,
      }} />
    </AbsoluteFill>
  );
};
```

### 5.5 Multi-Track Audio

Layer multiple audio sources with independent volume control:

```tsx
<Sequence from={0}>
  <Html5Audio src={staticFile("music/bgm.mp3")} volume={0.3} />
</Sequence>
<Sequence from={60} durationInFrames={120}>
  <Html5Audio src={staticFile("sfx/whoosh.wav")} volume={0.8} />
</Sequence>
<Sequence from={90}>
  <Html5Audio src={staticFile("voiceover/narration.mp3")} volume={1.0} />
</Sequence>
```

---

## 6. Composition Patterns for Complex Productions

### 6.1 Scene-per-Component Pattern

Each scene is a self-contained React component with its own animation logic. The root composition stitches them together:

```tsx
// src/compositions/Main.tsx
export const Main: React.FC<MainProps> = ({ title, accentColor }) => {
  return (
    <AbsoluteFill>
      <TransitionSeries>
        <TransitionSeries.Sequence durationInFrames={90}>
          <Intro title={title} color={accentColor} />
        </TransitionSeries.Sequence>
        <TransitionSeries.Transition
          presentation={slide({ direction: "from-right" })}
          timing={linearTiming({ durationInFrames: 15 })}
        />
        <TransitionSeries.Sequence durationInFrames={150}>
          <ContentSection />
        </TransitionSeries.Sequence>
        <TransitionSeries.Transition
          presentation={fade()}
          timing={linearTiming({ durationInFrames: 20 })}
        />
        <TransitionSeries.Sequence durationInFrames={60}>
          <Outro />
        </TransitionSeries.Sequence>
      </TransitionSeries>
    </AbsoluteFill>
  );
};
```

### 6.2 Data-Driven Video Generation

Pass dynamic data as input props for batch generation:

```tsx
// Generate 100 personalized videos via Node.js script
import { bundle } from "@remotion/bundler";
import { renderMedia, selectComposition } from "@remotion/renderer";

const bundled = await bundle({ entryPoint: "./src/index.ts" });

for (const user of users) {
  const composition = await selectComposition({
    serveUrl: bundled,
    id: "PersonalizedGreeting",
    inputProps: {
      userName: user.name,
      avatarUrl: user.avatar,
      stats: user.yearlyStats,
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: "h264",
    outputLocation: `out/${user.id}.mp4`,
  });
}
```

### 6.3 Canvas and WebGL Integration

```tsx
import { useCurrentFrame } from "remotion";
import { useRef, useEffect } from "react";

export const CanvasScene: React.FC = () => {
  const frame = useCurrentFrame();
  const canvasRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    const ctx = canvasRef.current?.getContext("2d");
    if (!ctx) return;

    ctx.clearRect(0, 0, 1920, 1080);
    // Draw frame-dependent graphics
    const particles = generateParticles(frame);
    particles.forEach((p) => {
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
      ctx.fillStyle = p.color;
      ctx.fill();
    });
  }, [frame]);

  return <canvas ref={canvasRef} width={1920} height={1080} />;
};
```

**Critical**: The `useEffect` dependency on `frame` ensures the canvas redraws for each frame. All particle positions must be computed as a pure function of `frame` — no accumulation.

---

## 7. CLAUDE.md — Prescriptive Context for Remotion Projects

The `CLAUDE.md` file is the single most important artifact for Claude Code integration. It provides prescriptive constraints so the agent generates Remotion-compatible code without re-prompting.

```markdown
# CLAUDE.md — Remotion Video Project

## Project Type
Programmatic video production using Remotion (React).

## Architecture
- Component-per-scene in src/compositions/
- Reusable elements in src/components/
- Shared animation utilities in src/lib/animations.ts
- Zod schemas for all composition props in src/schemas/

## Critical Constraints
- ALL animations MUST use useCurrentFrame(). Never CSS transitions.
- Never use setTimeout, setInterval, or requestAnimationFrame.
- Every visual must be a pure function of the frame number.
- Use spring() for organic motion, interpolate() for keyframed motion.
- Prefer <OffthreadVideo> over <Html5Video> for embedded clips.
- Use staticFile() for all assets in public/.

## Defaults
- Resolution: 1920×1080
- Frame rate: 30fps
- Codec: h264
- CRF: 18 (high quality)

## Audio
- Use @remotion/media-utils for audio visualization
- visualizeAudio() for music (FFT spectrum)
- visualizeAudioWaveform() for voice (time domain)
- Always pass smoothing: true for cleaner visuals

## Testing
- npx remotion studio to preview
- Check frame 0, midpoint, and last frame for each scene
- Verify audio sync by scrubbing timeline

## Rendering
- Local: npx remotion render <CompositionId> --codec h264
- Use --concurrency flag to match CPU cores / 2
```

---

## 8. Render Pipeline & Output Optimization

![Workflow Pipeline](assets/03-workflow-pipeline.svg)

### 8.1 CLI Rendering

```bash
# Basic render
npx remotion render Main out/video.mp4

# Production render with optimizations
npx remotion render Main out/video.mp4 \
  --codec h264 \
  --crf 18 \
  --concurrency 4 \
  --image-format jpeg \
  --jpeg-quality 90 \
  --scale 1 \
  --log verbose

# ProRes for post-production (e.g., DaVinci Resolve, Premiere)
npx remotion render Main out/video.mov \
  --codec prores \
  --prores-profile 4444  # Options: proxy, lt, standard, hq, 4444, 4444-xq

# Render a still frame
npx remotion still Main out/thumbnail.png --frame 45

# Render with custom props
npx remotion render Main out/video.mp4 \
  --props='{"title": "Custom Title", "accentColor": "#ff6b6b"}'
```

### 8.2 Codec Selection Guide

| Use Case | Codec | Flag | Notes |
|----------|-------|------|-------|
| Web delivery | H.264 | `--codec h264` | Universal compatibility. CRF 18-23. |
| High-quality web | H.265 | `--codec h265` | 30-50% smaller than H.264 at same quality |
| Transparency | VP8 WebM | `--codec vp8` | Alpha channel support |
| Post-production | ProRes | `--codec prores` | Lossless-ish. Huge files. DaVinci/Premiere. |
| Social media GIF | GIF | `--codec gif` | Use `--every-nth-frame 2` to reduce FPS |
| Image sequence | PNG | `--codec png` | Frame-by-frame PNGs for compositing |
| Audio only | AAC/WAV/MP3 | `--codec aac` | Extract or render audio without video |

### 8.3 CRF (Constant Rate Factor) Guidelines

Lower CRF = higher quality = larger files:

| CRF | Quality Level | File Size (1min 1080p) | Recommendation |
|-----|--------------|----------------------|----------------|
| 0 | Lossless | ~2-5 GB | Archival only |
| 15 | Near-lossless | ~200-400 MB | High-end production |
| 18 | Excellent | ~80-150 MB | **Recommended default** |
| 23 | Good | ~30-60 MB | Web delivery |
| 28 | Acceptable | ~15-30 MB | Low bandwidth |

### 8.4 Configuration File

`remotion.config.ts` sets defaults for all CLI commands:

```ts
import { Config } from "@remotion/cli/config";

Config.setVideoImageFormat("jpeg");
Config.setJpegQuality(90);
Config.setConcurrency(4);
Config.setOverwriteOutput(true);
```

### 8.5 SSR API — Programmatic Rendering

For CI/CD pipelines or server-side generation:

```ts
import { bundle } from "@remotion/bundler";
import { renderMedia, selectComposition } from "@remotion/renderer";

const bundled = await bundle({
  entryPoint: "./src/index.ts",
  // Optional: custom Webpack config
});

const composition = await selectComposition({
  serveUrl: bundled,
  id: "Main",
  inputProps: { title: "Generated Video" },
});

await renderMedia({
  composition,
  serveUrl: bundled,
  codec: "h264",
  outputLocation: "out/video.mp4",
  concurrency: 4,
  onProgress: ({ progress }) => {
    console.log(`Rendering: ${(progress * 100).toFixed(1)}%`);
  },
});
```

---

## 9. Scaling: Lambda, Cloud Run, and Batch Rendering

### 9.1 AWS Lambda

Remotion Lambda distributes rendering across serverless functions. Each Lambda renders a chunk of frames, and a final Lambda stitches the chunks.

```bash
# Deploy infrastructure
npx remotion lambda policies role     # Create IAM role
npx remotion lambda sites create      # Upload bundle to S3
npx remotion lambda functions deploy  # Deploy Lambda function

# Render
npx remotion lambda render <serve-url> Main
```

Key considerations for Lambda:
- Default concurrency per Lambda: 1 (one browser tab per function)
- Maximum video duration depends on Lambda timeout (up to 15 minutes)
- Chunks are rendered in parallel across multiple Lambda invocations
- Output lands in S3; optionally download to local

### 9.2 Google Cloud Run

Similar distribution model on GCP:

```bash
npx remotion cloudrun render <serve-url> Main \
  --service-name=remotion-renderer \
  --codec h264
```

### 9.3 Batch Rendering Script

Render all compositions in a project:

```bash
#!/bin/bash
compositions=($(npx remotion compositions src/index.ts -q))
for comp in "${compositions[@]}"; do
  npx remotion render src/index.ts "$comp" "out/${comp}.mp4" --codec h264
done
```

---

## 10. Advanced Recipes

### 10.1 Dynamic Text with Typewriter Effect

```tsx
export const Typewriter: React.FC<{ text: string; speed?: number }> = ({
  text,
  speed = 2,
}) => {
  const frame = useCurrentFrame();
  const charsVisible = Math.min(text.length, Math.floor(frame / speed));
  const displayText = text.slice(0, charsVisible);
  const showCursor = frame % 15 < 10; // Blinking cursor

  return (
    <span style={{ fontFamily: "monospace", fontSize: 48 }}>
      {displayText}
      <span style={{ opacity: showCursor ? 1 : 0 }}>▌</span>
    </span>
  );
};
```

### 10.2 Particle System (Pure Function of Frame)

```tsx
// Deterministic pseudo-random: same frame always produces same particles
function seededRandom(seed: number): number {
  const x = Math.sin(seed * 127.1) * 43758.5453;
  return x - Math.floor(x);
}

export const ParticleField: React.FC<{ count?: number }> = ({ count = 200 }) => {
  const frame = useCurrentFrame();
  const { width, height } = useVideoConfig();

  const particles = Array.from({ length: count }, (_, i) => {
    const seed = i * 1000;
    const baseX = seededRandom(seed) * width;
    const baseY = seededRandom(seed + 1) * height;
    const speed = 0.5 + seededRandom(seed + 2) * 2;
    const size = 2 + seededRandom(seed + 3) * 6;

    return {
      x: (baseX + frame * speed) % width,
      y: baseY + Math.sin(frame * 0.05 + i) * 20,
      size,
      opacity: 0.3 + seededRandom(seed + 4) * 0.7,
    };
  });

  return (
    <AbsoluteFill>
      {particles.map((p, i) => (
        <div
          key={i}
          style={{
            position: "absolute",
            left: p.x,
            top: p.y,
            width: p.size,
            height: p.size,
            borderRadius: "50%",
            backgroundColor: `rgba(255, 255, 255, ${p.opacity})`,
          }}
        />
      ))}
    </AbsoluteFill>
  );
};
```

### 10.3 Shapes Library

Remotion provides a `@remotion/shapes` package for procedural geometry:

```tsx
import { makeCircle, makeRect, makeStar, makeTriangle } from "@remotion/shapes";

const circle = makeCircle({ radius: 100 });
const star = makeStar({ points: 5, innerRadius: 40, outerRadius: 100 });

// Use as SVG paths
<svg>
  <path d={circle.path} fill="#61dafb" />
  <path d={star.path} fill="#f97316" transform="translate(300, 200)" />
</svg>
```

### 10.4 `calculateMetadata()` — Dynamic Duration and Dimensions

When video duration depends on input data (e.g., number of slides, audio length):

```tsx
import { Composition } from "remotion";
import { getAudioDurationInSeconds } from "@remotion/media-utils";

<Composition
  id="AudioDriven"
  component={AudioDrivenComp}
  fps={30}
  width={1920}
  height={1080}
  durationInFrames={300} // Placeholder
  calculateMetadata={async ({ props }) => {
    const duration = await getAudioDurationInSeconds(props.audioFile);
    return {
      durationInFrames: Math.ceil(duration * 30),
      props,
    };
  }}
/>
```

### 10.5 Precomputation with `calculateMetadata`

For physics simulations or audio-driven animations that need state across frames:

```tsx
calculateMetadata={async ({ props }) => {
  const audioData = await getAudioData(props.musicFile);
  // Precompute all particle positions for all frames
  const particlePositions = simulatePhysics(audioData, fps, totalFrames);

  return {
    durationInFrames: totalFrames,
    props: { ...props, particlePositions },
  };
}}
```

---

## 11. Troubleshooting & Performance Tuning

### 11.1 Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| Flickering during render | CSS transitions or `requestAnimationFrame` | Use only `useCurrentFrame()` for animation |
| Audio out of sync | `<Audio>` (deprecated) vs `<Html5Audio>` | Use `<Html5Audio>` in v4+; check frame offsets |
| Black frames | Component returns null on certain frames | Ensure all async data loads complete with fallback |
| Memory crash on long videos | `useAudioData()` loads entire file | Use `useWindowedAudioData()` for large files |
| Slow renders | High concurrency + heavy scenes | Reduce `--concurrency`; use `imageFormat: "jpeg"` |
| Font not loading | Race condition in font load | Use `@remotion/google-fonts` or `<Img>` pattern |

### 11.2 Performance Optimization Checklist

1. **Image format**: Use JPEG (`--image-format jpeg`) unless you need transparency. 3-5x faster than PNG.
2. **Concurrency**: Start at `os.cpus().length / 2`. Too high causes thrashing.
3. **Offthread video**: Always prefer `<OffthreadVideo>` over `<Html5Video>`. Uses Rust FFmpeg for frame extraction.
4. **Avoid re-renders**: Memoize heavy computations. `useMemo()` still works in Remotion.
5. **Precompute**: Use `calculateMetadata()` for expensive calculations that don't change per-frame.
6. **Bundle caching**: Keep `--bundle-cache` enabled (default). Only disable for debugging.
7. **Hardware acceleration**: Enable with `--hardware-acceleration if-possible` on supported systems.

### 11.3 Debugging

```bash
# Verbose logging
npx remotion render Main out/video.mp4 --log verbose

# Render specific frame range
npx remotion render Main out/video.mp4 --frames 100-200

# Single frame for debugging
npx remotion still Main out/debug.png --frame 150
```

---

## 12. Licensing

Remotion uses a **Business Source License (BSL 1.1)** with a **free tier** for individuals and small teams:

| Scenario | License Required | Cost |
|----------|-----------------|------|
| Individual / team of ≤ 3 | Free | $0 |
| Company / team of 4+ | Company License | $100/month |
| Enterprise (custom terms) | Enterprise License | $500+/month |
| Open-source projects | Free | $0 |

The Company License includes cloud rendering support and prioritized assistance. Always verify current terms at [remotion.dev/license](https://remotion.dev/license).

---

## 13. References

| Resource | URL |
|----------|-----|
| Remotion Documentation | https://remotion.dev/docs |
| API Reference | https://remotion.dev/docs/api |
| GitHub Repository | https://github.com/remotion-dev/remotion |
| Remotion Templates | https://remotiontemplates.dev |
| v5.0 Migration Guide | https://remotion.dev/docs/5-0-migration |
| Audio Visualization Docs | https://remotion.dev/docs/audio/visualization |
| Lambda Deployment | https://remotion.dev/docs/lambda |
| Cloud Run Deployment | https://remotion.dev/docs/cloudrun |
| Remotion Showcase | https://remotion.dev/showcase |

---

* Technical Guide v1.0 — March 2026*
