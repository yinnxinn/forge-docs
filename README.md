# Forge Docs — Official Documentation for Lingtarn Forge

This repository hosts the **official documentation** for [Lingtarn Forge](https://github.com/yinnxinn/forge-docs) (灵碳云铸). It uses a GitBook-style structure and is available in **English** (default) and **Chinese**.

## Documentation

- **English** (default): [gitbook/](gitbook/README.md) · [Table of contents](gitbook/SUMMARY.md)
- **中文**: [gitbook/zh/](gitbook/zh/README.md) · [目录](gitbook/zh/SUMMARY.md)

## Publish to GitBook (auto sync from GitHub)

To have [GitBook](https://www.gitbook.com/) build and host the docs and keep them in sync with this repo:

1. **Sign in** at [gitbook.com](https://www.gitbook.com/) and create or open a **Space** (your doc site).
2. Go to **Space settings** (gear icon) → **Integrations** (or **Git Sync**).
3. Choose **GitHub** and **Configure** / **Connect repository**.
4. **Authorize** GitBook in GitHub (install the GitBook app on your account/org if prompted).
5. Select **Repository**: `yinnxinn/forge-docs`, **Branch**: `main`.
6. **First sync**: choose **Import from GitHub** so GitBook uses the content from this repo. The repo root has a [`.gitbook.yaml`](.gitbook.yaml) that points the book root to `./gitbook/`, so the existing structure (including [LANGS.md](gitbook/LANGS.md) for EN/中文) is used.

After that, pushes to `main` will sync to GitBook as configured (GitBook will pull from GitHub). Edits can be made either in GitBook (they commit back to GitHub) or in this repo and pushed.

## Build locally

```bash
# Using HonKit (GitBook fork)
npx honkit build

# Or GitBook CLI
npx gitbook-cli build . ./_book
```

## License

MIT License — see [LICENSE](LICENSE).
