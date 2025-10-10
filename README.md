#!/usr/bin/env bash
# Usage: ./git-quick-update.sh <git-repo-url>
# Example: ./git-quick-update.sh git@github.com:username/repo.git
set -euo pipefail

REPO_URL="${1:-}"
if [[ -z "$REPO_URL" ]]; then
  echo "Usage: $0 <git-repo-url>"
  exit 2
fi

# configurable
BRANCH_PREFIX="auto-update"
COMMIT_MSG="chore: automatic README timestamp update"

# work in a temp dir
TMPDIR="$(mktemp -d)"
cleanup() { rm -rf "$TMPDIR"; }
trap cleanup EXIT

cd "$TMPDIR"

# clone (shallow)
git clone --depth 1 "$REPO_URL" repo
cd repo

# create new branch
TS="$(date -u +"%Y%m%dT%H%M%SZ")"
BRANCH="${BRANCH_PREFIX}/${TS}"
git checkout -b "$

# ensure README.md exists and append/update a timestamp line
README="README.md"
if ! [[ -f "$README" ]]; then
  echo "# Repository" > "$README"
fi

# Replace existing auto-update line or append
if grep -q '^<!-- AUTO-TIMESTAMP: .* -->$' "$README"; then
  sed -i "s/^<!-- AUTO-TIMESTAMP: .* -->$/<!-- AUTO-TIMESTAMP: ${TS} -->/" "$README"
else
  echo "" >> "$README"
  echo "<!-- AUTO-TIMESTAMP: ${TS} -->" >> "$README"
fi

git add "$README"
git commit -m "$COMMIT_MSG"

# push branch (assumes you have push rights and credential method configured)
git push -u origin "$BRANCH"

echo "Pushed branch: $BRANCH"
echo "Repo: $REPO_URL"


