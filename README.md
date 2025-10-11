# Fireact.dev Documentation Site

This repository hosts the official documentation website for Fireact.dev, built with [Hugo](https://gohugo.io/) and the [Docsy theme](https://www.docsy.dev/).

## Overview

The documentation site provides comprehensive guides, API references, and tutorials for developers using Fireact.dev. It includes:

- Getting Started guides
- Component API documentation
- Custom development tutorials
- Cloud Functions reference
- Type definitions
- Best practices and examples

## Live Site

Visit the live documentation at: [docs.fireact.dev](https://docs.fireact.dev)

## Tech Stack

- **Hugo**: Static site generator
- **Docsy Theme**: Documentation theme with search and navigation
- **Markdown**: Content format
- **PostCSS**: CSS processing
- **Autoprefixer**: CSS vendor prefixing

## Prerequisites

- **Hugo Extended** (v0.110.0 or higher) - [Installation Guide](https://gohugo.io/getting-started/installing/)
- **Node.js** (v18 or higher) for PostCSS processing
- **npm** for dependency management

### Installing Hugo

**macOS:**
```bash
brew install hugo
```

**Windows:**
```powershell
choco install hugo-extended
# or
scoop install hugo-extended
```

**Linux:**
```bash
# Snap
snap install hugo --channel=extended

# Or download from GitHub releases
wget https://github.com/gohugoio/hugo/releases/download/v0.125.0/hugo_extended_0.125.0_linux-amd64.deb
sudo dpkg -i hugo_extended_0.125.0_linux-amd64.deb
```

Verify installation:
```bash
hugo version
# Should show "extended" in the version
```

## Getting Started

### 1. Clone and Setup

```bash
# Navigate to docs directory
cd docs

# Install dependencies
npm install

# Initialize theme submodule (if needed)
git submodule update --init --recursive
```

### 2. Run Local Server

```bash
# Start Hugo server with drafts
hugo server -D

# Or without drafts
hugo server
```

The site will be available at: http://localhost:1313

**Features:**
- Live reload on content changes
- Fast rebuild times
- Draft content preview with `-D` flag

### 3. Build for Production

```bash
# Build static site
hugo

# Output will be in public/ directory
```

## Project Structure

```
docs/
├── content/                # Documentation content (Markdown)
│   ├── _index.md          # Homepage
│   ├── getting-started.md # Getting started guide
│   ├── app/               # App package docs
│   │   ├── components/    # Component documentation
│   │   ├── contexts/      # Context API docs
│   │   ├── hooks/         # Custom hooks docs
│   │   └── types/         # TypeScript types docs
│   ├── functions/         # Cloud Functions docs
│   │   ├── functions/     # Function reference
│   │   └── types/         # Function types docs
│   └── custom-development/ # Development guides
├── themes/                 # Hugo themes
│   └── docsy/             # Docsy theme (submodule)
├── static/                 # Static assets (images, CSS, JS)
├── layouts/                # Custom layout overrides
├── config.toml             # Hugo configuration
├── package.json            # Node dependencies
└── node_modules/           # npm dependencies
```

## Content Management

### Adding New Documentation

1. **Create a new markdown file**:
   ```bash
   # For a new page
   hugo new content/path/to/page.md

   # For a new section
   hugo new content/section/_index.md
   ```

2. **Front matter template**:
   ```markdown
   ---
   title: "Page Title"
   linkTitle: "Short Title"
   type: "docs"
   weight: 10
   description: >
     Brief description of the page
   ---

   # Content starts here
   ```

3. **Section index template**:
   ```markdown
   ---
   title: "Section Name"
   linkTitle: "Section"
   type: "docs"
   weight: 10
   cascade:
     type: "docs"
   ---

   ## Section Overview
   ```

### Documentation Structure Guidelines

**Front Matter Fields:**
- `title`: Full page title (shown in browser tab)
- `linkTitle`: Short title (shown in navigation)
- `type`: Always "docs" for documentation pages
- `weight`: Controls sort order (lower = higher in menu)
- `description`: SEO and preview description
- `no_list`: Set to `true` to hide page from section list

**Content Guidelines:**
- Use clear, concise language
- Include code examples where appropriate
- Add screenshots for UI features
- Link to related documentation
- Keep paragraphs short
- Use headings for structure

### Code Examples

Use fenced code blocks with language specification:

````markdown
```typescript
// TypeScript example
import { useAuth } from '@fireact.dev/app';

const MyComponent = () => {
  const { user } = useAuth();
  return <div>{user?.displayName}</div>;
};
```
````

### Images

Place images in `static/images/` and reference them:

```markdown
![Alt text](/images/screenshot.png)
```

## Customization

### Configuration

Main configuration in `config.toml`:

```toml
baseURL = "https://docs.fireact.dev/"
title = "Fireact.dev Documentation"
theme = "docsy"

[params]
  description = "Official documentation for Fireact.dev"
  github_repo = "https://github.com/fireact-dev/fireact.dev"

  [params.ui]
  navbar_logo = true
  sidebar_menu_compact = true
```

### Styling

Custom CSS in `assets/scss/_styles_project.scss`:

```scss
// Custom styles
.custom-class {
  // Your styles
}
```

### Layouts

Override theme layouts by creating files in `layouts/` with the same path as in `themes/docsy/layouts/`.

## Search

The documentation site includes search functionality powered by Docsy's built-in search.

To rebuild the search index:
```bash
hugo server
# Index rebuilds automatically
```

## Deployment

### Automatic Deployment (Recommended)

The documentation automatically deploys when changes are pushed to the main branch.

### Manual Deployment

```bash
# Build the site
hugo

# Deploy public/ directory to your hosting provider
```

### Firebase Hosting

```bash
# Build
hugo

# Deploy
firebase deploy --only hosting
```

### Netlify

1. Connect repository to Netlify
2. Set build command: `hugo`
3. Set publish directory: `public`
4. Deploy

### GitHub Pages

```bash
# Build
hugo

# Push public/ directory to gh-pages branch
git subtree push --prefix public origin gh-pages
```

## Contributing to Documentation

### Writing Guidelines

1. **Clear and Concise**: Use simple language
2. **Code Examples**: Include working examples
3. **Screenshots**: Add visuals for UI features
4. **Links**: Cross-reference related documentation
5. **Structure**: Use headings and lists
6. **Testing**: Verify all examples work
7. **Grammar**: Proofread before submitting

### Contribution Process

1. **Fork the repository**
2. **Create a branch**: `git checkout -b docs/improve-guide`
3. **Make changes** to markdown files in `content/`
4. **Test locally**: `hugo server`
5. **Commit**: `git commit -m "docs: improve getting started guide"`
6. **Push**: `git push origin docs/improve-guide`
7. **Create Pull Request**

### Documentation Standards

- Use Markdown formatting consistently
- Follow existing file naming conventions
- Include front matter for all pages
- Test all code examples
- Check links are not broken
- Optimize images before adding
- Follow the style guide

## Maintenance

### Updating Docsy Theme

```bash
# Update theme submodule
cd themes/docsy
git checkout main
git pull origin main
cd ../..

# Commit theme update
git add themes/docsy
git commit -m "docs: update Docsy theme"
```

### Updating Dependencies

```bash
# Update npm packages
npm update

# Check for outdated packages
npm outdated
```

### Link Checking

```bash
# Install hugo link checker
npm install -g @lycheeverse/lychee

# Check links
lychee 'content/**/*.md'
```

## Troubleshooting

### Hugo Not Found

Ensure Hugo Extended is installed:
```bash
hugo version
# Should show "extended"
```

### Theme Not Loading

Update submodules:
```bash
git submodule update --init --recursive
```

### Build Errors

Clear Hugo cache:
```bash
hugo --cleanDestinationDir
rm -rf public/
hugo
```

### PostCSS Errors

Reinstall dependencies:
```bash
rm -rf node_modules package-lock.json
npm install
```

### Port Already in Use

Change the port:
```bash
hugo server -p 1314
```

## Resources

- **Hugo Documentation**: https://gohugo.io/documentation/
- **Docsy Theme Docs**: https://www.docsy.dev/docs/
- **Markdown Guide**: https://www.markdownguide.org/
- **Hugo Shortcodes**: https://gohugo.io/content-management/shortcodes/
- **Main Project**: https://github.com/fireact-dev/fireact.dev

## License

This documentation is open source and available under the [MIT License](../LICENSE).
