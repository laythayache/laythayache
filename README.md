# .github/workflows/update_readme.yml
# ------------------------------
# This GitHub Action runs daily (and on pushes to main) to regenerate README.md
# using the `scripts/update_readme.py` script.
# ------------------------------
name: Update README

on:
  schedule:
    - cron: '0 0 * * *'       # every day at midnight UTC
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0       # so we can push tags if needed

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install requests

      - name: Generate README
        env:
          GITHUB_TOKEN: 
            ${{ secrets.GITHUB_TOKEN }}
          GITHUB_REPOSITORY: 
            ${{ github.repository }}
        run: |
          python scripts/update_readme.py > README.md

      - name: Commit and Push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git diff --quiet && echo "No changes to commit" || git commit -m "chore: update README.md"
          git push


# scripts/update_readme.py
# ------------------------------
# Fetches all public repos for the current user/org and prints
# a Markdown list for inclusion in README.md
# ------------------------------

import os
import requests
token = os.getenv('GITHUB_TOKEN')
repo_full = os.getenv('GITHUB_REPOSITORY')      # e.g. 'laythayache/Repos'
username = repo_full.split('/')[0]

headers = {'Authorization': f'token {token}'} if token else {}
url = f'https://api.github.com/users/{username}/repos'
params = {'sort': 'updated', 'direction': 'desc', 'per_page': 100}

repos = requests.get(url, headers=headers, params=params).json()

print("## Public Repositories")
print()
for r in repos:
    name = r['name']
    desc = r.get('description') or 'No description.'
    url = r['html_url']
    print(f"- **[{name}]({url})**  ")
    if desc:
        print(f"  {desc}")

# End of script
