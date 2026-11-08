# Git history rewrite (author / trailer cleanup)
- Repo is private GitHub arcticoder/wsl, branch main. Git identity: Arcticoder <10162808+arcticoder@users.noreply.github.com>.
- User rule: no commits authored or co-authored as "Claude". Do NOT add a `Co-Authored-By: Claude` trailer to new commits either (user overrides the default attribution reminder).
- 2026-11-07: removed `Co-Authored-By: Claude Haiku 5.5` trailers from 5092a00 and 4000a35 via:
  `git filter-branch -f --msg-filter 'sed -e "/^Co-Authored-By: Claude/d"' <oldest-bad>^..main`
  Tree contents verified identical (`git diff --stat <old-head> main` empty). Force-pushed with
  `git push --force-with-lease=main:<old-head> origin main`. Old head was 9b4053d, new head de56273.
- Before any history rewrite: `git bundle create <scratchpad>/x.bundle --all`. Then delete leftover `refs/original/*` (filter-branch leaves it).
- Verify after: `git log --all --format=%B | grep -i claude` should print nothing.
