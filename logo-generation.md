# Logo Generation

> Creating logos and favicons for new apps using AI tools.

---

## Interview First

Ask the user:

1. **What's the app name?**
2. **What does it do?** (helps with visual metaphors)
3. **Any style preferences?**
   - Minimal/clean?
   - Playful/fun?
   - Professional/corporate?
   - Geometric/abstract?
4. **Color preferences?** (or use brand colors if already defined)
5. **Icon only, text only, or combination?**

---

## Tool: Nano Banana (Browser)

Use the Claude-in-Chrome browser automation to generate logos via Nano Banana:

### Steps

1. Navigate to https://nanobanana.com (or similar AI logo tool)
2. Enter the app name and description
3. Select style preferences
4. Generate options
5. Download the best one

### Prompt Template for Logo Generation

```
Logo for [APP NAME]

App description: [WHAT IT DOES]

Style: [minimal/playful/professional]
Colors: [primary color], [secondary color]
Type: [icon only / wordmark / icon + text]

The icon should convey: [key concept - e.g., "speed", "security", "simplicity"]
```

---

## Required Assets

After generating the main logo, create these assets:

| Asset | Size | Use |
|-------|------|-----|
| `logo.svg` | Vector | Main logo, light backgrounds |
| `logo-dark.svg` | Vector | Main logo, dark backgrounds |
| `icon.svg` | Vector | Square icon only |
| `favicon.ico` | 16x16, 32x32 | Browser tab |
| `favicon-16x16.png` | 16x16 | Browser tab |
| `favicon-32x32.png` | 32x32 | Browser tab |
| `apple-touch-icon.png` | 180x180 | iOS home screen |
| `og-image.jpg` | 1200x630 | Social sharing |

---

## Favicon Generation

Once you have the icon, generate favicons:

### Option 1: RealFaviconGenerator

1. Go to https://realfavicongenerator.net
2. Upload your icon (SVG or high-res PNG)
3. Configure settings for each platform
4. Download the package
5. Place files in `public/`

### Option 2: Manual with ImageMagick

```bash
# From SVG to various sizes
convert icon.svg -resize 16x16 public/favicon-16x16.png
convert icon.svg -resize 32x32 public/favicon-32x32.png
convert icon.svg -resize 180x180 public/apple-touch-icon.png

# Create .ico with multiple sizes
convert icon.svg -resize 16x16 icon-16.png
convert icon.svg -resize 32x32 icon-32.png
convert icon-16.png icon-32.png public/favicon.ico
```

---

## File Placement

```
public/
├── favicon.ico
├── favicon-16x16.png
├── favicon-32x32.png
├── apple-touch-icon.png
├── icon.svg
├── og-image.jpg
└── site.webmanifest
```

---

## HTML Head Tags

Add to `application.html.erb`:

```erb
<%# Favicons %>
<link rel="icon" type="image/x-icon" href="/favicon.ico">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="icon" type="image/svg+xml" href="/icon.svg">
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="manifest" href="/site.webmanifest">
<meta name="theme-color" content="#[YOUR_PRIMARY_COLOR]">
```

---

## Web Manifest

Create `public/site.webmanifest`:

```json
{
  "name": "[APP NAME]",
  "short_name": "[APP NAME]",
  "icons": [
    {
      "src": "/favicon-16x16.png",
      "sizes": "16x16",
      "type": "image/png"
    },
    {
      "src": "/favicon-32x32.png",
      "sizes": "32x32",
      "type": "image/png"
    },
    {
      "src": "/apple-touch-icon.png",
      "sizes": "180x180",
      "type": "image/png"
    }
  ],
  "theme_color": "#[PRIMARY_COLOR]",
  "background_color": "#ffffff",
  "display": "standalone"
}
```

---

## OG Image (Social Sharing)

For social media previews, create a 1200x630 image with:
- App logo
- Tagline or value prop
- Brand colors

Can be created in:
- Figma
- Canva
- AI image generators
- Or simple HTML screenshot

---

## Alternative Logo Tools

If Nano Banana isn't available:

| Tool | URL | Notes |
|------|-----|-------|
| Looka | looka.com | AI logo generator |
| Brandmark | brandmark.io | AI-powered |
| Hatchful | hatchful.shopify.com | Free, Shopify |
| Canva | canva.com | Manual with templates |
| Midjourney | midjourney.com | AI generation |
