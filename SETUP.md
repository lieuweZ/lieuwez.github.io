# Portfolio Setup Guide

## Overview
This portfolio is styled after gamesbymanuel.com with a clean, modern design featuring a grid layout for projects.

## Quick Start

1. **Update Site Information**
   - Open `_config.yml`
   - Replace "YOUR NAME HERE" with your actual name
   - Update the description, email, and social media handles

2. **Add Your Projects**
   - Add project markdown files to the `_projects` folder
   - Each project should have:
     - `title`: Project name
     - `category`: e.g., "Digital, Published" or "Physical, Prototype"
     - `image`: Path to project thumbnail (e.g., `/assets/images/project1.jpg`)
     - `order`: Number for sorting projects
   - Add project images to `assets/images/`

3. **Add Project Images**
   - Place your project images in `assets/images/`
   - Recommended size: 700x700px or similar square/landscape format
   - Use JPG or PNG format

4. **Customize Pages**
   - `about.markdown`: Add your bio and experience
   - `ludography.md`: List all your games and projects
   - `contact.md`: Update contact information (auto-populated from _config.yml)

## Running Locally

```bash
bundle exec jekyll serve
```

Then visit `http://localhost:4000`

## Publishing to GitHub Pages

1. Push your changes to GitHub
2. Go to repository Settings > Pages
3. Select source: main branch
4. Your site will be live at `https://lieuweZ.github.io`

## File Structure

```
├── _config.yml           # Site configuration
├── _layouts/             # Page templates
│   ├── default.html      # Main layout with header/footer
│   └── project.html      # Individual project page
├── _projects/            # Project markdown files
├── _posts/               # Blog posts (optional)
├── assets/
│   ├── css/
│   │   └── style.css     # Main stylesheet
│   └── images/           # Project and content images
├── index.markdown        # Home page (projects grid)
├── about.markdown        # Bio page
├── ludography.md         # Game credits
└── contact.md            # Contact information
```

## Customization Tips

### Change Colors
Edit `assets/css/style.css` and update:
- `#1a1a1a` - Dark text color
- `#666` - Medium gray
- `#999` - Light gray
- `#fff` - White background

### Adjust Grid Layout
In `style.css`, modify `.projects-grid`:
```css
grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
```
Change `350px` to adjust project card size.

### Add More Navigation Items
Edit `_layouts/default.html` in the `<nav class="main-nav">` section.

## Need Help?

- Jekyll Documentation: https://jekyllrb.com/docs/
- GitHub Pages Guide: https://docs.github.com/en/pages
- Markdown Guide: https://www.markdownguide.org/

## Notes

- The site is configured to work with GitHub Pages
- Remember to restart Jekyll server after changing `_config.yml`
- Markdown linting warnings (MD033) are normal for HTML in markdown files
