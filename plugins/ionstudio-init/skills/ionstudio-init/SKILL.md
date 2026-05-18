---
name: ionstudio-init
description: Set up a new ionstudioapps dev environment from scratch — GitHub access, repos, tools, env vars, and Claude skills. Run once on a new machine or when onboarding a new teammate.
---

# ionstudioapps Dev Environment Setup

Walk through each step in order. Report pass ✅ / fail ❌ / skipped ⏭ for each. At the end, print a summary of what's ready and what still needs manual action.

---

## Step 1 — Check prerequisites

```bash
echo "Node:  $(node -v 2>/dev/null || echo NOT FOUND)"
echo "npm:   $(npm -v 2>/dev/null || echo NOT FOUND)"
echo "git:   $(git --version 2>/dev/null || echo NOT FOUND)"
echo "gh:    $(gh --version 2>/dev/null | head -1 || echo NOT FOUND)"
echo "eas:   $(eas --version 2>/dev/null || echo NOT FOUND)"
```

If any are missing:
- **Node/npm**: install from https://nodejs.org (LTS)
- **git**: install via Xcode CLT — `xcode-select --install`
- **gh**: `brew install gh`
- **eas**: `npm install -g eas-cli`

Pause and ask the user to install missing tools before continuing.

---

## Step 2 — GitHub authentication

```bash
gh auth status 2>&1
```

If not logged in or wrong account:
```bash
# This must be run interactively — tell the user to run it in their terminal
echo "Run in your terminal: gh auth login --hostname github.com --web"
```

After login, confirm the authenticated user has access to `ionstudioapps`:
```bash
gh repo list ionstudioapps --limit 5 2>&1
```

If access is denied, ask them to request an invite at github.com/orgs/ionstudioapps/people from Sua (alexsuakim@gmail.com).

---

## Step 3 — SSH key setup for ionstudioapps

Check if an SSH key already works with GitHub:
```bash
ssh -T git@github.com 2>&1 | head -1
```

If the response doesn't say "Hi [username]", generate a new key:
```bash
# Ask the user for their GitHub email first
ssh-keygen -t ed25519 -C "USER_EMAIL" -f ~/.ssh/id_ed25519_ionstudio -N ""
cat ~/.ssh/id_ed25519_ionstudio.pub | pbcopy
echo "SSH public key copied to clipboard — add it at https://github.com/settings/ssh/new"
```

Add SSH config entry if not already present:
```bash
grep -q "github-ionstudio" ~/.ssh/config 2>/dev/null || cat >> ~/.ssh/config << 'EOF'

# ionstudioapps
Host github-ionstudio
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_ionstudio
EOF
echo "SSH config updated"
```

Wait for the user to add the key to GitHub, then test:
```bash
ssh -T git@github-ionstudio 2>&1
```

---

## Step 4 — Clone repos

Check which repos are already cloned, clone the rest:

```bash
PROJECTS_DIR="$HOME/Projects"
mkdir -p "$PROJECTS_DIR"

REPOS=("wheeltodo" "rocky" "studio")

for repo in "${REPOS[@]}"; do
  if [ -d "$PROJECTS_DIR/$repo/.git" ]; then
    echo "⏭ $repo already exists — pulling latest"
    git -C "$PROJECTS_DIR/$repo" pull
  else
    echo "Cloning $repo..."
    git clone git@github-ionstudio:ionstudioapps/$repo.git "$PROJECTS_DIR/$repo"
  fi
done
```

---

## Step 5 — Install dependencies

```bash
# wheeltodo
echo "Installing wheeltodo dependencies..."
npm install --prefix "$HOME/Projects/wheeltodo"

# rocky
echo "Installing rocky dependencies..."
npm install --prefix "$HOME/Projects/rocky"
```

---

## Step 6 — Set up environment secrets

Clone the private secrets repo and copy env files into place:

```bash
SECRETS_DIR="$HOME/.ionstudio/secrets"

# Clone or pull secrets repo
if [ -d "$SECRETS_DIR/.git" ]; then
  echo "Pulling latest secrets..."
  git -C "$SECRETS_DIR" pull
else
  echo "Cloning secrets repo..."
  git clone git@github-ionstudio:ionstudioapps/secrets.git "$SECRETS_DIR"
fi

# Create shared env dir
mkdir -p "$HOME/.ionstudio"

# Copy rocky.env to both the shared location and rocky project
cp "$SECRETS_DIR/rocky.env" "$HOME/.ionstudio/rocky.env"
cp "$SECRETS_DIR/rocky.env" "$HOME/Projects/rocky/.env"

echo "✅ Environment files installed"
```

If the clone fails (no org access yet), tell the user to ask Sua (alexsuakim@gmail.com) for access to ionstudioapps/secrets.

---

## Step 7 — EAS login (for mobile builds)

```bash
eas whoami 2>&1
```

If not logged in, tell the user:
```
Run in your terminal: eas login --browser
Log in with the ionstudio Expo account — ask Sua for credentials.
```

---

## Step 8 — Install Claude skills marketplace

Tell the user to run these slash commands in Claude Code:
```
/plugin marketplace add https://github.com/ionstudioapps/claude-skills
/plugin install code-review
/plugin install deploy
/plugin install rocky-debug
/plugin install pr-description
/plugin install project-status
/plugin install schedule-meeting
/plugin install ionstudio-init
/reload-plugins
```

---

## Final Summary

Print a checklist:
```
ionstudioapps Setup Summary
═══════════════════════════
✅/❌ Node/npm
✅/❌ git
✅/❌ gh CLI + authenticated
✅/❌ SSH key for ionstudioapps
✅/❌ wheeltodo cloned & deps installed
✅/❌ rocky cloned & deps installed
✅/❌ studio cloned
✅/❌ rocky/.env created
⚠️  rocky/.env needs manual values (list which ones)
✅/❌ EAS logged in
⚠️  Claude skills — install manually via /plugin commands above

You're ready to build 🚀
```
