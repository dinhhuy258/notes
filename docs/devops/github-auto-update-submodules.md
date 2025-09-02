# Automatically updating git submodules using GitHub Actions

## Implementation Strategies

### 1. Repository Dispatch Method

For repositories you control, use `repository_dispatch` to trigger updates:

**Submodule Repository Workflow** (triggers on push):

```yaml
name: Dispatch Update

on:
  push:
    branches:
      - master

jobs:
  notify-infrastructure:
    runs-on: ubuntu-latest
    steps:
      - name: Notify parent-repo
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.PAT }}
          repository: owner/parent-repo
          event-type: submodule-update
          client-payload: |
            {
              "repository": "${{ github.repository }}"
            }
```

**Parent Repository Workflow** (receives dispatch):

```yaml
name: Update Submodules
on:
  repository_dispatch:
    types: [submodule-update]

jobs:
  update-submodule:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout infrastructure
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.PAT }}
          submodules: true

      - name: Update submodule
        run: |
          REPO_NAME=$(echo "${{ github.event.client_payload.repository }}" | cut -d'/' -f2)

          if [ "$REPO_NAME" = "xxx" ]; then
            SUBMODULE_PATH="xxx"
          elif [ "$REPO_NAME" = "yyy" ]; then
            SUBMODULE_PATH="yyy"
          else
            echo "Unknown repository: $REPO_NAME"
            exit 1
          fi

          echo "Updating submodule: $SUBMODULE_PATH"
          cd $SUBMODULE_PATH
          git fetch origin
          git checkout origin/master
          cd ../..

      - name: Commit submodule update
        run: |
          REPO_NAME=$(echo "${{ github.event.client_payload.repository }}" | cut -d'/' -f2)

          if [ "$REPO_NAME" = "xxx" ]; then
            SUBMODULE_PATH="xxx"
          else
            SUBMODULE_PATH="yyy"
          fi

          git config --local user.email "huy.duongdinh@gmail.com"
          git config --local user.name "Huy Duong"
          git add $SUBMODULE_PATH
          git commit -m "chore: update $REPO_NAME submodule to latest" || exit 0
          git push
```
