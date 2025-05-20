# Create a basic javascript action

1. Create a new repository called `hello-world-javascript-action`
2. Clone the repository locally and open the folder in an editor.
3. Follow the [instructions](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-javascript-action#commit-tag-and-push-your-action) on how to setup a JavaScript action. you can skip step 2,3 and 4.
5. When you reach `commit, tag and push your action` you can keep follow the instructions, or add the provided ci/cd workflow below, or create your own ci/cd flow. Make sure to locate your index.js file in the `src/index.js` and also don´t forget to update the `action.yml`to point to `main:dist/index.js`

<details>
  <summary>Solution</summary>

### ci/cd workflow for javascript action 

````YML
name: 🚀 CICD

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

permissions:
  contents: write
  actions: write
  checks: write

env:
  VERSION: 1.0.${{ github.run_number }}

jobs:
  test-bundle-release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        id: checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        id: setup-node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install Dependencies
        id: npm-ci
        run: |
          npm install
          npm i -g @vercel/ncc

      - name: Bundle
        run: ncc build src/index.js --license licenses.txt

      - name: Check if new bundle is created and needs to be checked in
        id: gitChanges
        run: |
          git status
          if [[ -n "$(git status --porcelain ./dist)" ]]; then
            echo "changes=true" >> $GITHUB_OUTPUT
          else
            echo "changes=false" >> $GITHUB_OUTPUT
          fi

      - name: Commit changes
        if: steps.gitChanges.outputs.changes == 'true' && github.ref == 'refs/heads/main'
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'
          git add ./dist
          git commit -m "[skip ci]: Bundle and check javascript. Release $VERSION"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Create Release
        if: steps.gitChanges.outputs.changes == 'true' && github.ref == 'refs/heads/main'
        id: create_release
        run: |
          gh release create $VERSION ./dist/* --title "Release $VERSION" --notes "This is an automated release created by the CI workflow."
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Create tags
        if: steps.gitChanges.outputs.changes == 'true' && github.ref == 'refs/heads/main'
        run: |
          git tag $VERSION
          git push origin $VERSION
          git tag -fa v1 $VERSION -m "Update v1 tag to $VERSION"
          git push origin v1 --force

      - name: Warn if no new binaries
        if: steps.gitChanges.outputs.changes != 'true'
        run: |
          echo "::warning::No new binaries have been created and no new release tag will be created."

````

</details>

6. In the consume workflow we created earlier, add another job to call the javascript action by it´s reference.`organizaton/repository@v1`
