# Quick Start Guide

Get the LLM Scaling Paper running in 3 minutes.

## Prerequisites Check

```bash
# Check if Node.js is installed (need v18+)
node --version

# Check if npm is installed
npm --version
```

**Don't have Node.js?** Download from [nodejs.org](https://nodejs.org/) (get LTS version)

## Setup in 3 Steps

### Step 1: Install Dependencies (2 minutes)

```bash
npm install
```

Wait for packages to download...

### Step 2: Start Development Server (30 seconds)

```bash
npm run dev
```

You should see:
```
Nuxt 3.x.x with Nitro 2.x.x
  ➜ Local:    http://localhost:3000/
```

### Step 3: Open in Browser

Go to: **http://localhost:3000**

Done! 🎉

## Adding Your Images

1. Create a `public` folder in the project root:
```bash
mkdir public
```

2. Copy your 8 WhatsApp images to the `public` folder

3. Update image paths in `app.vue`:
```html
<!-- Replace placeholder with your actual image -->
<img src="/WhatsApp_Image_2026-02-08_at_12_28_15.jpeg" alt="..." />
```

Image mapping:
- Figure 1: `WhatsApp_Image_2026-02-08_at_12_28_15.jpeg` (Eiffel Tower/Paris scatter)
- Figure 2: `WhatsApp_Image_2026-02-08_at_12_28_17__1_.jpeg` (Feature selection menu)
- Figure 3: `WhatsApp_Image_2026-02-08_at_12_28_16.jpeg` (Weak superposition graph)
- Figure 4: `WhatsApp_Image_2026-02-08_at_12_28_19__1_.jpeg` (Two-panel scaling)
- Figure 5: `WhatsApp_Image_2026-02-08_at_12_28_17.jpeg` (Empire State menu)
- Figure 6: `WhatsApp_Image_2026-02-08_at_12_28_19.jpeg` (Interference visualization)

Text excerpts:
- Section 4.2: `WhatsApp_Image_2026-02-08_at_12_28_18__1_.jpeg`
- Section 5.2: `WhatsApp_Image_2026-02-08_at_12_28_18.jpeg`

## Common Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Generate static site
npm run generate

# Preview production build
npm run preview
```

## Stopping the Server

Press `Ctrl + C` in the terminal

## Troubleshooting

**Port 3000 already in use?**
```bash
PORT=3001 npm run dev
```

**Build errors?**
```bash
# Clear cache and reinstall
rm -rf .nuxt node_modules
npm install
```

**Still having issues?**
Check the full README.md for detailed troubleshooting.

## Deployment (Static Site)

```bash
# 1. Generate static files
npm run generate

# 2. Upload the .output/public folder to any web host
# - Netlify: Drag & drop the folder
# - Vercel: Connect your Git repo
# - GitHub Pages: Copy to docs/ folder
```

## File Structure

```
Your Project/
├── app.vue          ← Main file with all content
├── package.json     ← Dependencies
├── nuxt.config.ts   ← Configuration
├── public/          ← Add images here
└── README.md        ← Full documentation
```

## Customization

**Change colors?** Edit the `<style>` section in `app.vue`

**Modify content?** Edit the `<template>` section in `app.vue`

**Add sections?** Follow the pattern in existing sections

## Need Help?

1. Check README.md for detailed guide
2. Visit [nuxt.com/docs](https://nuxt.com/docs)
3. Check Node.js is v18 or higher

---

**That's it!** You now have a running academic paper website.
