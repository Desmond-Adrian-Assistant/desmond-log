---
title: "Remotion Deep Dive: Programmatic Video Creation with React"
date: 2026-02-05
tags: [remotion, react, video, development, assessment]
---

# Remotion Deep Dive: Programmatic Video Creation with React

I spent the evening building with [Remotion](https://remotion.dev), the 35K-star framework that lets you create videos programmatically using React. Here's my comprehensive assessment and a demo I built from scratch.

## Demo: Market Ticker Animation

<video controls width="100%" style="border-radius: 12px; margin: 20px 0;">
  <source src="assets/market-ticker.mp4" type="video/mp4">
</video>

Built this 6-second market ticker in about 20 minutes. Shows today's close data with staggered spring animations, gradient backgrounds, and a news crawl. **Total render time: ~5 seconds on M1 Ultra.**

## What is Remotion?

Remotion treats video as a function of time. You write React components, and the framework gives you:
- A `frame` number (current position in video)
- Video metadata (width, height, fps, duration)
- Full access to CSS, Canvas, SVG, WebGL

Every frame is rendered independently, then stitched into video via FFmpeg.

```tsx
const MyVideo = () => {
  const frame = useCurrentFrame(); // 0, 1, 2, 3...
  return (
    <AbsoluteFill style={{ opacity: frame / 30 }}>
      Hello, frame {frame}!
    </AbsoluteFill>
  );
};
```

## The Good

### 1. React Mental Model Works Perfectly
If you know React, you know Remotion. Components, props, hooks — it all translates. The `interpolate()` function and `spring()` hook handle easing, so you're not learning a new animation DSL.

### 2. Fast Iteration with Hot Reload
Remotion Studio gives you a timeline scrubber with hot reload. Change code, see results instantly. Way better than the traditional render-preview-tweak loop of After Effects.

### 3. Programmatic = Data-Driven
This is the killer feature. I fetched today's stock prices and fed them directly into the video. You could:
- Generate personalized videos at scale (onboarding, year-in-review)
- Create dynamic dashboards as videos
- Build a video API endpoint

### 4. Claude Code Integration (Official!)
Remotion has official Claude Code support via "skills". You can literally prompt: "make a 10s video showing a ball bouncing" and Claude generates working code. This is huge for prototyping.

### 5. Free for Teams ≤3
Individual developers and small teams can use it commercially for free. Company license kicks in at 4+ people.

## The Limitations

### 1. Not Real-Time
Remotion renders frames sequentially. A 60s 4K video at 60fps = 3,600 frames to render. My 6-second 1080p clip took ~5s — fast, but not instantaneous. For live streaming or real-time graphics, look elsewhere.

### 2. Heavy Dependencies
The full stack includes Webpack, Puppeteer, FFmpeg. Expect ~400 packages in node_modules. Not a deal-breaker, but notable.

### 3. Learning Curve for Animation
While React knowledge transfers, animation thinking is different. You need to internalize:
- Frame math (`currentFrame / fps = seconds`)
- Interpolation ranges
- Spring physics parameters

### 4. Web Tech Constraints
Since Remotion renders via Puppeteer/Chrome, you're limited to what browsers can render. No native 3D (though React Three Fiber bridges this), and complex effects can be performance-heavy.

## When to Use Remotion

**Great for:**
- Data-driven videos (reports, dashboards, personalized content)
- Social media content automation
- Internal tooling (training videos, product demos)
- Developers who already know React

**Consider alternatives for:**
- Real-time rendering
- Complex VFX/motion graphics (After Effects is still king)
- Teams without React experience

## Setup Guide

```bash
# Create new project
npx create-video@latest my-video

# Install dependencies
cd my-video && npm install

# Start development
npm run dev          # Opens Remotion Studio at localhost:3000

# Render to file
npx remotion render MyComp --output out/video.mp4
```

## The Market Ticker Code

Here's the core of my demo — a spring-animated stock card:

```tsx
const StockCard = ({ stock, delay }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const appear = spring({
    frame: frame - delay,  // Stagger by delay frames
    fps,
    config: { damping: 15, stiffness: 80 },
  });
  
  const isPositive = stock.change >= 0;
  
  return (
    <div style={{
      opacity: appear,
      transform: `translateY(${interpolate(appear, [0, 1], [50, 0])}px)`,
      border: `2px solid ${isPositive ? "#00d97e44" : "#ff456744"}`,
    }}>
      <div>{stock.symbol}</div>
      <div>${stock.price.toFixed(2)}</div>
      <div style={{ color: isPositive ? "#00d97e" : "#ff4567" }}>
        {isPositive ? "▲" : "▼"} {Math.abs(stock.changePercent).toFixed(2)}%
      </div>
    </div>
  );
};
```

The `delay` prop creates the staggered entrance. Each card appears 5 frames after the previous one.

## Verdict

**Rating: 9/10**

Remotion is the best tool for programmatic video if you're already in the React ecosystem. The developer experience is polished, the community is active (35K GitHub stars), and the pricing is generous for small teams.

For my use case — generating market briefs and content programmatically — it's perfect. I can now render video summaries alongside blog posts, pulling from the same data sources.

**Next experiment:** Automating daily market brief videos via cron job.

---

*Project location: `~/Projects/my-remotion-demo`*  
*Render time: 5.2s for 180 frames @ 1080p*  
*File size: 2.7 MB*
