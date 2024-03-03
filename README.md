# Personal Website

This website has been forked from [antfu.me](https://antfu.me).

## Introduction

This project aims to create a personal website using modern web development tools and frameworks. It utilizes Vue.js as the main frontend framework along with Vite for fast development and Vue Router for routing. The website is designed to be highly responsive and user-friendly.

## Installation

To get started with this project, follow these steps:

1. Clone the repository: `git clone <repository-url>`
2. Navigate into the project directory: `cd <project-directory>`
3. Install dependencies: `npm install`

## Usage

### Development

To run the project in development mode, use the following command:

```bash
npm run dev
```

This will start a development server at http://localhost:3333/.

### Production Build

To build the project for production, use the following command:

```bash
npm run build
```

This command will generate optimized production-ready assets in the `dist` directory.

### Preview Production Build

To preview the production build locally, use the following command:

```bash
npm run build-and-preview
```

This will build the project and start a local server to preview the production build.

### Additional Scripts

- `npm run redirects`: Run TypeScript script to handle redirects.
- `npm run compress`: Compress images using a TypeScript script.
- `npm run spellcheck`: Spell check markdown files.
- `npm run lint:code`: Lint code using ESLint.
- `npm run lint:md`: Lint markdown files using markdownlint.

## Dependencies

This project relies on the following dependencies:

- **Vue.js**: Frontend framework for building user interfaces.
- **Vue Router**: Official router for Vue.js.
- **@vueuse/core**: Collection of essential Vue Composition API utils.
- **dayjs**: Date library for parsing, validating, manipulating, and formatting dates.
- **nprogress**: Slim progress bars for Ajax'y applications.
- **floating-vue**: Vue.js plugin for creating floating elements.
- **vue-router-better-scroller**: Better scroll behavior for Vue Router.

## Development Dependencies

This project utilizes various development dependencies for build, linting, and testing purposes. Some key ones include:

- **Vite**: Next-generation frontend tooling.
- **ESLint**: JavaScript and TypeScript linter.
- **markdownlint**: Markdown linting library.
- **rimraf**: UNIX command rm -rf for node.
- **simple-git-hooks**: A simple package to manage Git hooks.
- **lint-staged**: Run linters on git staged files.