FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=0005c79e09bae5f4cc2bd8ccd468d7dafe24a455
RUN git clone https://github.com/pallets/werkzeug . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# Pinned from the repo's uv.lock (`uv export --frozen --no-default-groups --group <tests>`) at
# BASE_SHA, resolved for Linux / CPython 3.12 (the mars-base default interpreter), so the
# image matches upstream CI for this commit and does not drift when PyPI moves.
RUN pip install --no-cache-dir \
        "cffi==2.1.0" \
        "cryptography==49.0.0" \
        "ephemeral-port-reserve==1.1.4" \
        "iniconfig==2.3.0" \
        "markupsafe==3.0.3" \
        "packaging==26.2" \
        "pluggy==1.6.0" \
        "pycparser==3.0" \
        "pygments==2.20.0" \
        "pytest-timeout==2.4.0" \
        "pytest==9.1.1" \
        "watchdog==6.0.0" \
    && pip install --no-cache-dir --no-deps -e . \
    && test -z "$(git status --porcelain)"

# v1.1 node-id scoring: pytest ships a native JUnit XML reporter (--junitxml),
# so no extra reporter dependency is required.

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]
