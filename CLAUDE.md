# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Tribit Social Media Analytics Dashboard** - Static HTML dashboards visualizing social media metrics for Tribit (audio/electronics brand) across YouTube, TikTok, Instagram, and advertising platforms.

- **Architecture**: Standalone HTML files with inline CSS/JavaScript (no build process)
- **Data**: Hardcoded in `<script>` tags within each HTML file
- **Libraries**: ECharts 5.4.3 (primary), Chart.js (India KOL dashboard)
- **Deployment**: Vercel static hosting
- **Navigation**: Portal-based iframe system with hierarchical menu

## Commands

```bash
# Development (no build required)
open index.html                    # macOS
python -m http.server 8000        # Python server
npx http-server                   # Node.js server

# Deployment
vercel                            # Deploy to production
vercel --prod                     # Force production deployment

# Git workflow
git add .
git commit -m "message"
git push origin main              # Push to tribit3 (GitHub)
./push_to_github.sh              # Helper script with PAT instructions

# Testing checklist
# 1. Portal navigation (index.html iframe loading)
# 2. Chart rendering (ECharts/Chart.js initialization)
# 3. Video embeds (YouTube/TikTok/Instagram)
# 4. Responsive design (mobile/desktop)
# 5. Keyboard shortcuts (↑↓ navigation)
```

## Architecture

```
tribit3/
├── index.html                    # Portal with iframe navigation
├── tribit-selfkoc-*.html        # Self-generated content dashboards
├── tribit-kol-*.html            # KOL performance dashboards  
├── tribit-ads-*.html            # Advertising dashboards
├── data/                        # CSV files (reference only)
├── vercel.json                  # Deployment config
└── push_to_github.sh            # GitHub push helper
```

### Portal System (index.html)
- **Navigation Structure**: 3-tier hierarchy (Section → Category → Dashboard)
- **Sections**: Self-KOC, Global-KOL, Advertising Campaigns
- **Iframe Loading**: Dynamic dashboard loading with loading states
- **Keyboard Shortcuts**: Arrow keys (↑↓) for navigation
- **State Management**: Active item tracking, section collapsing

### Dashboard Architecture
- **Standalone Files**: Each dashboard is self-contained HTML
- **Data Location**: JavaScript arrays in `<script>` tags at bottom of file
- **Chart Libraries**: ECharts instances initialized per chart container
- **Styling**: Inline CSS with glassmorphism effects
- **Animations**: CSS keyframes (fadeIn, fadeInUp, fadeInDown)

## Key Code Patterns

### Portal Navigation (index.html)
```javascript
// Navigation item structure
<a class="nav-item" data-page="dashboard.html">
    <span class="nav-item-icon">📊</span>
    Dashboard Name
</a>

// Section with categories
<div class="nav-section">
    <div class="nav-section-title">Section Name</div>
    <div class="nav-submenu">
        <div class="nav-category">
            <div class="nav-category-title">Category</div>
            <div class="nav-category-items"><!-- nav-items --></div>
        </div>
    </div>
</div>
```

### Video Data Structure
```javascript
const videoData = [{
    no: 1,
    channel: 'youtube',      // youtube, tiktok, instagram
    account: '@username',
    url: 'https://...',
    likes: 103,
    comments: 0,
    views: 33000,
    date: '2025/4/4',       // YYYY/M/D format
    videoId: 'xxx',         // YouTube/TikTok only
    postId: 'xxx'           // Instagram only
}];
```

### ECharts Initialization
```javascript
const chart = echarts.init(document.getElementById('chartId'));
chart.setOption({
    backgroundColor: 'transparent',
    title: { text: 'Title', textStyle: { color: '#fff' } },
    tooltip: { 
        trigger: 'axis',
        backgroundColor: 'rgba(0, 0, 0, 0.8)',
        borderColor: '#667eea'
    },
    grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
    // Data arrays embedded directly
});

// Responsive handling
window.addEventListener('resize', () => {
    chart.resize();
});
```

### Platform-Specific Video Embeds

#### YouTube Shorts
```javascript
// Modal preview pattern
const iframe = document.createElement('iframe');
iframe.src = `https://www.youtube.com/embed/${videoId}?autoplay=1&mute=1`;
iframe.width = "360";
iframe.height = "640";
iframe.allow = "accelerometer; autoplay; clipboard-write; encrypted-media";
```

#### TikTok
```html
<blockquote class="tiktok-embed" 
    cite="${video.url}" 
    data-video-id="${videoId}" 
    style="max-width: 605px; min-width: 325px;">
</blockquote>
<script async src="https://www.tiktok.com/embed.js"></script>
```

#### Instagram (Updated Pattern)
```html
<!-- Loading state -->
<div class="loading-placeholder">
    <div class="spinner"></div>
    <div>Loading Instagram Reel...</div>
</div>
<!-- Embed container -->
<div class="instagram-embed-container" style="display: none;">
    <blockquote class="instagram-media" 
        data-instgrm-captioned 
        data-instgrm-permalink="${url}?utm_source=ig_embed&utm_campaign=loading" 
        data-instgrm-version="14">
    </blockquote>
</div>
<script async src="//www.instagram.com/embed.js"></script>
```

### CSS Patterns
- Glassmorphism: `backdrop-filter: blur(10px); background: rgba(255,255,255,0.1);`
- Animations: `fadeIn`, `fadeInUp`, `fadeInDown` classes
- Dark theme with `#0a0a0a` background
- Responsive breakpoint: `@media (max-width: 768px)`

## Development Workflow

### Adding New Dashboard
1. Copy similar dashboard as template (e.g., `tribit-selfkoc-tk-q1.html`)
2. Update `videoData` array with new data
3. Modify chart configurations and IDs
4. Add navigation entry in `index.html`:
   ```html
   <a class="nav-item" data-page="new-dashboard.html">
       <span class="nav-item-icon">📊</span>
       New Dashboard Name
   </a>
   ```
5. Test locally with HTTP server
6. Deploy: `vercel`

### Updating Dashboard Data
1. Open HTML file in editor
2. Locate `const videoData = [` section
3. Update JavaScript array values
4. Save and test rendering
5. Commit changes: `git add . && git commit -m "Update dashboard data"`
6. Push: `git push origin main`

### Data Update Locations
- **Video data**: Search for `const videoData = [`
- **Chart data**: Look for `series:` within `chart.setOption()`
- **Statistics**: Find calculation sections after data arrays
- **Dates**: Update both data arrays and title text

## Common Issues & Solutions
- **Charts not rendering**: Check element ID matches `echarts.init()` call
- **Video embeds broken**: Verify video IDs and platform-specific embed formats
- **Portal navigation fails**: Ensure dashboard filename in `data-page` attribute exists
- **CORS errors**: Use local HTTP server, not `file://` protocol
- **Data not updating**: Clear browser cache after editing JavaScript arrays
- **Instagram embeds not loading**: Check postId format and Instagram script loading

## Platform Notes

- **YouTube**: Shorts use 9:16 aspect ratio, modal preview supported
- **TikTok**: May be blocked by some networks, uses standard embed
- **Instagram**: Requires loading placeholders, progressive display
- **All platforms**: Test with real video IDs before deployment