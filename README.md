# SoloLandscapes — Vlogs & Video Integration Guide

A developer guide for implementing **9:16 YouTube Shorts / Field Reels** on `/blogs` and **In-Article YouTube Video Embeds** in the blog editor and detail pages.

---

## 🔗 Live Interactive Prototypes

| Page | Live URL | Description |
| :--- | :--- | :--- |
| **Shorts Row on `/blogs`** | [test-vlog.vercel.app/blogs](https://test-vlog.vercel.app/blogs) | 9:16 Shorts shelf placed **directly above** the hero section |
| **Admin Shorts Manager** | [test-vlog.vercel.app/admin](https://test-vlog.vercel.app/admin) | CMS dashboard: reorder, add short, active/draft toggle |
| **In-Article Media Inserter** | [test-vlog.vercel.app/editor](https://test-vlog.vercel.app/editor) | Editor prototype: insert Image vs YouTube Video under headings |
| **Blog Detail Page** | [test-vlog.vercel.app/blogs/6a72f375c211742677db1b06](https://test-vlog.vercel.app/blogs/6a72f375c211742677db1b06) | In-article 16:9 landscape video player |

---

## Part 1: Row of Short Videos Above Hero on `/blogs`

### 1. Placement Architecture

Place the **Field Shorts & Reels** shelf between the Sticky Header and the Blogs Hero:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. <Header activeTab="vlogs" />                             │
├─────────────────────────────────────────────────────────────┤
│ 2. <FieldShortsRow shorts={activeShorts} />   [NEW SECTION] │
│    Horizontal scroll of 9:16 vertical cards                 │
├─────────────────────────────────────────────────────────────┤
│ 3. <BlogsHero />                                            │
│    "Explore Our Travel Stories & Insights"                  │
├─────────────────────────────────────────────────────────────┤
│ 4. <BlogArticleGrid articles={articles} />                  │
└─────────────────────────────────────────────────────────────┘
```

### 2. Frontend Component (`components/vlogs/FieldShortsRow.tsx`)

```tsx
import React, { useState } from 'react';
import { Zap, Play, MapPin } from 'lucide-react';
import { CleanShortModal } from './CleanShortModal';

interface ShortItem {
  id: string;
  youtubeId: string;
  title: string;
  location: string;
  guideName: string;
  guideAvatar: string;
  badge?: string;
  linkedArticle?: string;
}

export function FieldShortsRow({ shorts }: { shorts: ShortItem[] }) {
  const [selectedIndex, setSelectedIndex] = useState<number | null>(null);

  return (
    <section className="bg-white border-b border-slate-200 py-4 sm:py-6 shadow-xs">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        
        {/* Header */}
        <div className="flex items-center justify-between mb-3.5">
          <div className="flex items-center gap-2">
            <div className="w-6 h-6 rounded-md bg-rose-600 text-white flex items-center justify-center">
              <Zap className="w-3.5 h-3.5 fill-white" />
            </div>
            <div>
              <h2 className="text-sm sm:text-base font-bold text-slate-900 leading-tight">
                Field Shorts &amp; Reels
              </h2>
              <p className="text-[11px] text-slate-500">Quick vertical dispatches from our guides on location</p>
            </div>
          </div>
          <span className="text-[10px] font-bold text-rose-600 bg-rose-50 border border-rose-200 px-2.5 py-0.5 rounded-full">
            Swipe to Watch &rarr;
          </span>
        </div>

        {/* 9:16 Horizontal Cards Shelf */}
        <div className="flex gap-3.5 overflow-x-auto pb-2 scrollbar-none snap-x">
          {shorts.map((short, idx) => (
            <div
              key={short.id}
              onClick={() => setSelectedIndex(idx)}
              className="relative shrink-0 w-[140px] sm:w-[160px] md:w-[175px] aspect-[9/16] rounded-2xl overflow-hidden shadow-sm hover:shadow-md transition-all group cursor-pointer bg-slate-900 border border-slate-200 snap-start select-none"
            >
              {/* YouTube Thumbnail */}
              <img
                src={`https://img.youtube.com/vi/${short.youtubeId}/hqdefault.jpg`}
                alt={short.title}
                className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
              />
              <div className="absolute inset-0 bg-gradient-to-b from-black/50 via-transparent to-black/90" />

              {/* Guide Tag */}
              <div className="absolute top-2.5 left-2.5 flex items-center gap-1.5 z-10">
                <img
                  src={short.guideAvatar}
                  alt={short.guideName}
                  className="w-6 h-6 rounded-full object-cover border border-white"
                />
                <span className="bg-black/50 backdrop-blur-xs text-[9px] font-bold text-white px-1.5 py-0.5 rounded">
                  {short.guideName}
                </span>
              </div>

              {/* Play Icon */}
              <div className="absolute inset-0 flex items-center justify-center pointer-events-none">
                <div className="w-9 h-9 rounded-full bg-white/30 backdrop-blur-md text-white flex items-center justify-center shadow group-hover:scale-110 group-hover:bg-rose-600 transition-all">
                  <Play className="w-4 h-4 fill-white ml-0.5" />
                </div>
              </div>

              {/* Bottom Info */}
              <div className="absolute bottom-2.5 left-2.5 right-2.5 text-white z-10">
                <p className="text-xs font-bold leading-tight line-clamp-2 drop-shadow">
                  {short.title}
                </p>
                <p className="text-[9px] text-teal-200 mt-1 flex items-center gap-1">
                  <MapPin className="w-2.5 h-2.5" /> {short.location}
                </p>
              </div>
            </div>
          ))}
        </div>

      </div>

      {/* Lightbox Modal */}
      {selectedIndex !== null && (
        <CleanShortModal
          shorts={shorts}
          currentIndex={selectedIndex}
          onClose={() => setSelectedIndex(null)}
          onChangeIndex={(newIdx) => setSelectedIndex(newIdx)}
        />
      )}
    </section>
  );
}
```

### 3. Lightbox Modal Rules (Clean & Conflict-Free)

1. **Keep Website Controls Outside the YouTube Iframe**:
   * Do NOT position the close `[✕]` or open `[↗]` buttons inside the video container.
   * Place them on a dedicated top bar above the 9:16 frame. This guarantees they **never collide with YouTube's sound button, title bar, or 3-dot menu**.
2. **Touch Swipe Up/Down**:
   * Add `touchstart` and `touchend` listeners on the container to detect vertical swipes (> 45px) to navigate between shorts without requiring button clicks.
3. **No Social Gimmicks**:
   * No fake comment bars, heart reaction counters, or animated story progress bars. This is a viewing-only travel dispatch.

---

## Part 2: Admin Dashboard for Shorts Management

Live prototype: **[test-vlog.vercel.app/admin](https://test-vlog.vercel.app/admin)**

### 1. Database Schema (`prisma/schema.prisma`)

```prisma
model ShortVideo {
  id            String   @id @default(cuid())
  youtubeId     String   // 11-char YouTube ID (e.g., "JNE1T-Q1TBU")
  title         String   // e.g., "Sunrise mangrove kayak loop in Kampot"
  location      String   // e.g., "Kampot River Trail"
  badge         String   @default("Short") // "Short" | "Reel" | "Guide Tip"
  
  guideName     String   // "Lisa"
  guideAvatar   String   // URL to photo
  
  displayOrder  Int      @default(0) // Order on /blogs top shelf
  isActive      Boolean  @default(true) // Toggle Active vs Draft
  
  linkedArticle String?  // Optional path to blog (e.g., "/blogs/6a72f375...")
  viewsCount    Int      @default(0)
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  @@index([isActive, displayOrder])
}
```

### 2. Auto-Extracting YouTube ID from Admin Input

When the admin pastes any format into the URL field:

```ts
export function extractYouTubeId(url: string): string {
  if (!url) return '';
  const trimmed = url.trim();
  // If raw 11-character ID was provided
  if (trimmed.length === 11 && !trimmed.includes('/') && !trimmed.includes('.')) {
    return trimmed;
  }
  // Matches shorts/ID, watch?v=ID, youtu.be/ID
  const regex = /(?:youtube\.com\/(?:[^\/]+\/.+\/|(?:v|e(?:mbed)?|shorts)\/|.*[?&]v=)|youtu\.be\/)([^"&?\/\s]{11})/i;
  const match = trimmed.match(regex);
  return match ? match[1] : '';
}
```

* As soon as an ID is detected, display the 9:16 thumbnail instantly using:
  `https://img.youtube.com/vi/${id}/hqdefault.jpg`

---

## Part 3: Inserting Image OR YouTube Video Inside Blog Content

Live prototype: **[test-vlog.vercel.app/editor](https://test-vlog.vercel.app/editor)**

Under blog headings (such as *"What Solo Travel Actually Gives You"* in your blog editor), admins need to easily insert **either an image gallery or a YouTube video**.

### 1. Install TipTap YouTube Extension

SoloLandscapes' blog editor uses **TipTap** (ProseMirror). Enable official YouTube support with one package:

```bash
npm install @tiptap/extension-youtube @tiptap/extension-image
```

### 2. Editor Configuration (`components/editor/BlogEditor.tsx`)

```ts
import { Editor, EditorContent } from '@tiptap/react'
import StarterKit from '@tiptap/starter-kit'
import Image from '@tiptap/extension-image'
import Youtube from '@tiptap/extension-youtube'

const editor = new Editor({
  extensions: [
    StarterKit,
    Image.configure({
      inline: true,
      allowBase64: false,
      HTMLAttributes: {
        class: 'rounded-2xl shadow-md my-6 w-full object-cover',
      },
    }),
    Youtube.configure({
      controls: true,
      nocookie: true,
      allowFullscreen: true,
      modestbranding: true,
      HTMLAttributes: {
        class: 'w-full aspect-video rounded-2xl overflow-hidden shadow-lg my-6 border border-slate-200',
      },
    }),
  ],
  content: initialContent,
})
```

### 3. Adding the Options to the Toolbar `[+ Add ▾]`

In your existing editor toolbar, expand the **`[+ Add]`** button into a dropdown menu:

```tsx
<div className="relative">
  <button onClick={() => setMenuOpen(!menuOpen)} className="btn-add">
    + Add ▾
  </button>

  {menuOpen && (
    <div className="dropdown-menu">
      {/* Option 1: Image */}
      <button onClick={() => insertImageFile()}>
        <ImageIcon className="w-4 h-4" /> Upload Image
      </button>

      {/* Option 2: YouTube Video */}
      <button onClick={() => promptYouTubeUrl()}>
        <VideoIcon className="w-4 h-4 text-red-600" /> YouTube Video
      </button>
    </div>
  )}
</div>
```

Helper function for inserting video:

```ts
function addYouTubeVideo(url: string) {
  if (!url) return;
  editor.commands.setYoutubeVideo({
    src: url,
    width: 640,
    height: 360,
  });
}
```

### 4. Zero-Click Auto-Paste (Modern Notion/Ghost Style)

TipTap's YouTube extension automatically converts pasted links. If an admin pastes a YouTube URL onto an empty line, TipTap converts it into the 16:9 video player without requiring toolbar navigation.

---

## Part 4: Frontend Detail Page Rendering (`/blogs/[slug]`)

When rendering the saved article HTML on `sololandscapes.co/blogs/[slug]`, ensure the CSS handles responsive aspect ratios:

```css
/* Responsive 16:9 container for TipTap YouTube iframes */
.prose iframe[src*="youtube.com"] {
  width: 100% !important;
  aspect-ratio: 16 / 9;
  height: auto !important;
  border-radius: 1rem;
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
}
```

---

## Summary Checklist for Developers

- [ ] Place `<FieldShortsRow />` on `/blogs` **above** the Hero section.
- [ ] Ensure the video modal controls (`✕`, `↗`) sit **above** the video frame to avoid YouTube UI collisions.
- [ ] Add touch swipe up/down event listeners for mobile gesture navigation.
- [ ] In the admin dashboard, create the **Field Shorts & Reels** table with `↑`/`↓` sequence reordering.
- [ ] In TipTap, install `@tiptap/extension-youtube` and add **"YouTube Video"** to the `[+ Add]` toolbar menu.
