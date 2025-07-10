# GitHub Pages Deployment Instructions

Follow these steps to deploy your book to GitHub Pages:

## 1. Create a GitHub Repository

1. Go to [GitHub](https://github.com) and sign in
2. Click "New repository" (green button)
3. Name your repository: `random-sampling-app-book`
4. Make it **Public** (required for free GitHub Pages)
5. Don't initialize with README (we already have one)
6. Click "Create repository"

## 2. Upload Your Book

### Option A: Using GitHub Web Interface (Easiest)

1. In your new repository, click "uploading an existing file"
2. Drag and drop ALL files from your `book-site` folder
3. Write a commit message: "Initial book upload"
4. Click "Commit changes"

### Option B: Using Git Command Line

```bash
# In your book-site folder
git init
git add .
git commit -m "Initial book upload"
git branch -M main
git remote add origin https://github.com/yourusername/random-sampling-app-book.git
git push -u origin main
```

## 3. Enable GitHub Pages

1. Go to your repository on GitHub
2. Click "Settings" tab
3. Scroll down to "Pages" in the left sidebar
4. Under "Source", select "GitHub Actions"
5. **That's it!** - No save button needed
   - GitHub Actions is now enabled for Pages
   - The page will show "GitHub Actions" as the source
   - You might see a message about workflows being detected once you upload your files

**Note**: Don't worry about suggested workflows or custom domains at this point - we'll handle everything with our custom setup.

## 4. Update Configuration

Before deploying, update these files:

### `mkdocs.yml`
Replace placeholders with your actual information:
- `site_url: "https://yourusername.github.io/random-sampling-app-book/"`
- `repo_name: "yourusername/random-sampling-app-book"`
- `repo_url: "https://github.com/yourusername/random-sampling-app-book"`

### Social Links in `mkdocs.yml`
Update the social links in the `extra` section:
```yaml
extra:
  social:
    - icon: fontawesome/brands/github
      link: https://github.com/yourusername
    - icon: fontawesome/brands/twitter
      link: https://twitter.com/yourusername
    - icon: fontawesome/brands/linkedin
      link: https://linkedin.com/in/your-linkedin-username
```

**To find your LinkedIn username**: Look at your LinkedIn profile URL - it's the part after `/in/`. For example, if your URL is `https://www.linkedin.com/in/chandersheel-pahel-7b7a48104/`, then your username is `chandersheel-pahel-7b7a48104`.

## 5. Deploy

**If you've already uploaded all files and made changes directly on GitHub:**

1. **Your deployment should start automatically!** 
   - GitHub will detect the workflow file (`.github/workflows/deploy.yml`) 
   - The deployment will begin as soon as you save changes to `mkdocs.yml`

2. **Check the deployment status:**
   - Go to the "Actions" tab in your repository
   - You should see a workflow running (it will show "Deploy MkDocs to GitHub Pages")
   - Wait for it to complete (usually 2-3 minutes)
   - A green checkmark means success, a red X means there's an error

3. **Your book will be available at:** `https://yourusername.github.io/random-sampling-app-book/`

**If you need to make more changes later:**
- You can edit files directly on GitHub (click the pencil icon to edit)
- Or download the repository, make changes locally, and push them back
- Any change to the `main` branch will trigger a new deployment automatically

## 6. Automatic Updates

Once set up, your book will automatically update whenever you:
1. Make changes to any markdown files
2. Push changes to the `main` branch
3. The GitHub Action will rebuild and deploy automatically

## 7. Custom Domain (Optional)

If you want to use a custom domain:

1. Add a `CNAME` file to your `docs` folder with your domain name
2. Configure DNS settings with your domain provider
3. Update the `site_url` in `mkdocs.yml`

## Troubleshooting

### Common Issues

**No workflow running in Actions tab:**
1. **Check if the workflow file was uploaded correctly:**
   - In your repository, navigate to `.github/workflows/`
   - You should see a file called `deploy.yml`
   - If it's missing, you need to create the folder structure and upload the file

2. **Workflow file might be in wrong location:**
   - The file MUST be at exactly: `.github/workflows/deploy.yml`
   - Note the dot (.) at the beginning of `.github`

3. **Repository might not have detected the workflow yet:**
   - Make any small change to any file (like adding a space to README.md)
   - Commit the change - this will trigger workflow detection

4. **Actions might be disabled:**
   - Go to repository Settings → Actions → General
   - Make sure "Actions permissions" is set to "Allow all actions and reusable workflows"

**Build fails**: Check the Actions tab for error messages
**Site not loading**: Ensure repository is public and Pages is enabled
**Images not showing**: Check file paths and ensure images are in the docs folder
**Custom domain not working**: Verify DNS settings and CNAME file

### Getting Help

- Check the [MkDocs documentation](https://www.mkdocs.org/)
- Visit [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- Review [GitHub Pages documentation](https://docs.github.com/en/pages)

## Next Steps

Once your book is live:
1. Share the URL with others
2. Add the link to your GitHub profile
3. Consider adding analytics to track visitors
4. Keep content updated as you improve the application

---

**Your book will be live at:** `https://yourusername.github.io/random-sampling-app-book/`

Remember to replace `yourusername` with your actual GitHub username!
