# How to Start Your Jekyll Site Locally

## Quick Start
```bash
cd /Users/mdasifbinsyed/coding/website/asifbinsyed.github.io
bundle exec jekyll serve
```

Then open your browser to: **http://localhost:4000**

## Detailed Steps

### 1. Navigate to your project directory
```bash
cd /Users/mdasifbinsyed/coding/website/asifbinsyed.github.io
```

### 2. Start the Jekyll development server
```bash
bundle exec jekyll serve
```

### 3. Open your browser
- Go to: **http://localhost:4000**
- The site will auto-reload when you make changes!

## Alternative Server Options

### Run on different host/port
```bash
bundle exec jekyll serve --host 0.0.0.0 --port 4000
```

### Run with incremental builds (faster for large sites)
```bash
bundle exec jekyll serve --incremental
```

### Run in production mode (for testing)
```bash
bundle exec jekyll serve --environment production
```

## Stopping the Server
- Press **Ctrl+C** in the terminal where the server is running

## Troubleshooting

### If you get Ruby/gem errors:
```bash
bundle install
```

### If you get permission errors:
```bash
bundle exec jekyll serve --host 0.0.0.0 --port 4000
```

### If the site doesn't load:
1. Check that the server is running (you should see "Server running...")
2. Make sure you're using the correct URL: http://localhost:4000
3. Try refreshing your browser

## What Happens When Running
- ✅ **Auto-regeneration**: Changes to files automatically rebuild the site
- ✅ **Live reload**: Refresh your browser to see changes
- ✅ **Error reporting**: Any build errors will show in the terminal
- ✅ **File watching**: The server watches for changes in all your files

## File Structure
- `index.md` - Your homepage
- `blog.md` - Blog page
- `contact.md` - Contact information
- `_config.yml` - Site configuration
- `_sass/` - SCSS styling files
- `_layouts/` - Page templates
- `_includes/` - Reusable components

## Making Changes
1. Edit any `.md` file (like `index.md`, `blog.md`)
2. Save the file
3. Refresh your browser at http://localhost:4000
4. Your changes will appear instantly!

---
*Last updated: September 2024*
