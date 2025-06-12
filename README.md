name: Generate Profile README

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]
  
  # Runs on schedule (every 12 hours)
  schedule:
    - cron: "0 */12 * * *"
  
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  generate-snake:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    
    steps:
      # Checkout the repository
      - name: Checkout
        uses: actions/checkout@v4
      
      # Generate the snake animation
      - name: Generate Snake Animation
        uses: Platane/snk/svg-only@v3
        with:
          github_user_name: philipgisore
          outputs: |
            dist/github-snake.svg
            dist/github-snake-dark.svg?palette=github-dark
            dist/github-snake-light.svg?palette=github-light
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      # Setup Pages
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      # Upload artifact
      - name: Upload Pages Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist
      
      # Deploy to GitHub Pages
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

  update-readme:
    runs-on: ubuntu-latest
    needs: generate-snake
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      # Update README with dynamic content
      - name: Update README
        run: |
          # Get current date
          DATE=$(date +'%Y-%m-%d %H:%M:%S UTC')
          
          # Create a backup of original README
          cp README.md README.backup.md
          
          # Update last updated timestamp (if you want to add this)
          echo "<!-- Last updated: $DATE -->" >> README.md
      
      # Commit and push if there are changes
      - name: Commit changes
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add -A
          git diff --staged --quiet || git commit -m "🚀 Update README and snake animation - $(date +'%Y-%m-%d')"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # Job to update GitHub profile with latest stats
  update-profile-stats:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      # This will trigger regeneration of GitHub stats
      - name: Trigger Stats Update
        run: |
          echo "Stats will be updated automatically by the services"
          echo "GitHub ReadMe Stats: $(date)"
          echo "Streak Stats: $(date)"
      
      # Optional: Add more dynamic content updates here
      - name: Add Dynamic Content
        run: |
          echo "Profile updated at: $(date)" > .github/last-update.txt
      
      - name: Commit updates
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add .github/last-update.txt
          git diff --staged --quiet || git commit -m "📊 Update profile stats - $(date +'%Y-%m-%d')"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
