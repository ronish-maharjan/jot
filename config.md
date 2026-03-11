```bash

# ---- jot ----
export JOT="$HOME/dev-root/jot"

# today's log
jl() {
  mkdir -p "$JOT/log"
  local f="$JOT/log/$(date +%Y-%m-%d).md"
  if [ ! -f "$f" ]; then
    printf "# %s %s\n\n## did\n\n\n## learned\n\n\n## thoughts\n\n" \
      "$(date +%Y-%m-%d)" "$(date +%A)" > "$f"
  fi
  cd "$JOT" && nvim "$f"
}

# open scratch
alias js='cd $JOT && nvim scratch.md'

# open ideas
alias ji='cd $JOT && nvim ideas.md'

# open refs
alias jr='cd $JOT && nvim refs.md'

# quick capture to scratch without opening editor
# usage: jq "fix the nginx timeout thing"
jq() { printf "%s\n" "$*" >> "$JOT/scratch.md" && echo "→ scratch"; }

# quick capture idea without opening editor
# usage: ja "cli tool that diffs two directory trees"
ja() { printf -- "- %s\n" "$*" >> "$JOT/ideas.md" && echo "→ ideas"; }

# search all notes
# usage: jg "deadlock"
jg() { cd "$JOT" && grep -ri --color=auto "$1" --include='*.md'; }

# save: pull first to avoid conflicts, then commit and push
jc() {
  cd "$JOT" || return
  git pull --rebase -q 2>/dev/null
  git add -A
  git commit -m "$(date '+%Y-%m-%d %H:%M')" || return
  git push
}

```

