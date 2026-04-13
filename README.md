# Movie Explorer

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

## GitHub Pages Deployment

This repository is configured to deploy automatically to GitHub Pages from the `main` branch.

- Workflow: `.github/workflows/deploy-gh-pages.yml`
- Trigger: push to `main` (or manual `workflow_dispatch`)
- Build output: `dist/`

### Required repository configuration

In GitHub repository settings:

1. Go to **Settings -> Pages**
2. Under **Build and deployment**, set **Source** to **GitHub Actions**

The workflow sets `VITE_BASE_PATH` to `/<repo-name>/` so the app loads correctly from the project Pages URL.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Oxc](https://oxc.rs)
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/)

## React Compiler

The React Compiler is not enabled on this template because of its impact on dev & build performances. To add it, see [this documentation](https://react.dev/learn/react-compiler/installation).

## Expanding the ESLint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and [`typescript-eslint`](https://typescript-eslint.io) in your project.
