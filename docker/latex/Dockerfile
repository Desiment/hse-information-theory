# syntax=docker/dockerfile:1.7
FROM debian:trixie-slim

ENV DEBIAN_FRONTEND=noninteractive

ARG TL_YEAR=2024
ARG TL_REPO=https://ftp.math.utah.edu/pub/tex/historic/systems/texlive/${TL_YEAR}/tlnet-final
ARG TL_DIR=/usr/local/texlive/${TL_YEAR}

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/lists,sharing=locked \
    set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      ca-certificates curl wget xz-utils \
      perl make git \
      python3 python3-pygments \
      fontconfig fonts-dejavu \
      cpanminus \
      libfontconfig1 libfreetype6 libharfbuzz0b \
      zlib1g libpng16-16 libjpeg62-turbo \
      libxml2 libxslt1.1 \
      inkscape \
    ; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/texlive
RUN --mount=type=cache,target=/var/cache/texlive-installer,sharing=locked \
    set -eux; \
    curl -fsSL --retry 5 --retry-delay 2 \
      -o /var/cache/texlive-installer/install-tl-unx.tar.gz \
      "${TL_REPO}/install-tl-unx.tar.gz"; \
    tar -xzf /var/cache/texlive-installer/install-tl-unx.tar.gz --strip-components=1; \
    printf "%s\n" \
      "selected_scheme scheme-small" \
      "TEXDIR ${TL_DIR}" \
      "TEXMFCONFIG ~/.texlive/texmf-config" \
      "TEXMFVAR ~/.texlive/texmf-var" \
      "option_doc 0" \
      "option_src 0" \
      > texlive.profile; \
    ./install-tl --profile=texlive.profile -repository "${TL_REPO}"; \
    ln -sfn "${TL_DIR}" /usr/local/texlive/current; \
    rm -rf /tmp/texlive

# Arch-agnostic: /usr/local/texlive/current/bin/<arch> link to  bin/platform
RUN set -eux; \
    TL_BIN_DIR="$(find /usr/local/texlive/current/bin -mindepth 1 -maxdepth 1 -type d | head -n 1)"; \
    test -n "$TL_BIN_DIR"; \
    ln -sfn "$TL_BIN_DIR" /usr/local/texlive/current/bin/platform

ENV PATH="/usr/local/texlive/current/bin/platform:${PATH}"

RUN set -eux; \
    tlmgr option repository "${TL_REPO}"; \
    tlmgr option autobackup 0

RUN set -eux; \
    tlmgr install \
      latexmk \
      luatex \
      collection-latexrecommended \
      collection-latexextra \
      collection-fontsrecommended \
      minted \
      ifplatform \
      xcolor \
      graphics \
      collection-langcyrillic \
      cm-unicode \
      biber \
      advice \
      collargs \
      memoize \
      pgfplots \
      tikz-cd \
      quiver \
      svg \
      xsim \
      tasks \
      todonotes \
      keytheorems \
      stackengine \
      mdframed \
      imakeidx \
      biblatex \
      csquotes \
      mathcommand \
      extarrows \
      esvect

RUN tlmgr path add

RUN cpanm --notest PDF::Builder PDF::API2

RUN set -eux; \
    command -v latexmk; latexmk -v | head -n 1; \
    lualatex --version | head -n 2

WORKDIR /work
