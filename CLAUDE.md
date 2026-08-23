# Piyush Labs - Personal Blog

This repository contains the source code for a personal blogging site hosted at https://me.piyushlabs.com/.

## About

This is Piyush Anand's personal hobby blog - a place away from work to share experiences and projects related to:
- **Self-hosting**: Running personal services and infrastructure
- **3D Printing**: Projects, tips, and experiences with 3D printing
- **Home Automation**: Smart home projects and automations
- **Open Source Software**: Contributions, explorations, and experiences with OSS

## Technical Stack

- **Static Site Generator**: [Hugo](https://gohugo.io/)
- **Theme**: [PaperMod](https://github.com/adityatelange/hugo-PaperMod) (included as git submodule)
- **Configuration**: `hugo.yaml`
- **Content**: Markdown files in `content/posts/`

## Repository Structure

```
├── archetypes/          # Content templates
│   └── default.md
├── content/             # All site content
│   ├── posts/          # Blog posts organized by topic
│   │   ├── 3d-printing/
│   │   ├── HomeAutomation/
│   │   └── self-hosing/
│   └── search.md       # Search page
├── themes/             # Hugo themes
│   └── PaperMod/       # Git submodule
├── hugo.yaml           # Main site configuration
└── .gitignore          # Git ignore patterns
```

## Site Configuration

- **Base URL**: https://me.piyushlabs.com/
- **Language**: English (en-us)
- **Environment**: Production (enables analytics, opengraph, twitter-cards, schema)
- **Features**: Search enabled, social links (GitHub, LinkedIn)

## Working with this Repository

### Local Development
```bash
# Start Hugo development server
hugo server -D

# Build site
hugo
```

### Creating New Posts
```bash
# Create a new post in a specific category
hugo new posts/[category]/[post-name].md
```

### Theme Management
The PaperMod theme is included as a git submodule. To update:
```bash
git submodule update --remote themes/PaperMod
```

## Author Information

- **Name**: Piyush Anand
- **Location**: Boston
- **Role**: Software Engineer
- **GitHub**: [@piyush-an](https://github.com/piyush-an)
- **LinkedIn**: [anandpiyush](https://www.linkedin.com/in/anandpiyush/)

## Development Guidelines

- Keep content focused on hobby projects and personal experiences
- Organize posts by topic in appropriate subdirectories
- Use meaningful file names and frontmatter
- Test locally before deploying
- Maintain clean commit history
