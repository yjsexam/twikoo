```
jobs:
  update_release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Node.js environment
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Get latest release
        id: get-latest-release
        uses: pozetroninc/github-action-get-latest-release@master
        with:
          repository: twikoojs/twikoo
          excludes: prerelease, draft
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Install jq
        run: sudo apt-get install -y jq

      - name: Update package.json with latest release version
        run: |
          latest_release="${{ steps.get-latest-release.outputs.release }}"
          jq --arg ver "$latest_release" '.dependencies["twikoo-vercel"] = $ver' package.json > package.tmp.json && mv package.tmp.json package.json

      - name: Commit and push changes if necessary
        run: |
          git config user.name "GitHub Action"
          git config user.email "action@github.com"
          git add package.json
          git diff-index --quiet HEAD || git commit -m "v${{ steps.get-latest-release.outputs.release }}"
          git push
```

```
jobs:
  update_master:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout master branch
      uses: actions/checkout@v4
      with:
        ref: master

    - name: Download and unzip main branch
      run: |
        curl -L https://codeload.github.com/walinejs/waline/zip/refs/heads/main -o waline-main.zip
        unzip waline-main.zip -d waline-main
        rm waline-main.zip

    - name: Remove .github folder
      run: rm -rf waline-main/waline-main/.github

    - name: Prepare and commit files
      run: |
        rsync -av waline-main/waline-main/ ./
        rm -rf waline-main
        git config user.name "github-actions"
        git config user.email "github-actions@github.com"
        git add .
        git diff-index --quiet HEAD || git commit -m "Merge branch 'main' of https://github.com/walinejs/waline"
        git push

    - name: Push changes to master branch
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        branch: master
```

```
jobs:
  update-version:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        ref: master

    - name: Get @waline/vercel version
      id: get_version
      run: |
        echo "Fetching @waline/vercel version from waline repository"
        version=$(curl -s https://raw.githubusercontent.com/walinejs/waline/main/packages/server/package.json | jq -r '.version')
        echo "version=$version" >> $GITHUB_ENV

    - name: Update package.json
      run: |
        echo "Updating @waline/vercel version to ${{ env.version }}"
        jq --arg version "${{ env.version }}" '.dependencies["@waline/vercel"] = $version' package.json > tmp.$$.json && mv tmp.$$.json package.json

    - name: Commit changes
      run: |
        git config user.name "GitHub Action"
        git config user.email "action@github.com"
        git add package.json
        git diff-index --quiet HEAD || git commit -m "v${{ env.version }}"
        git push
```
