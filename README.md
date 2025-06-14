# Typst Project Template with Automated Releases

This repository serves as a template for creating documents with [Typst](https://typst.app/), a modern, markup-based typesetting system. It includes a pre-configured GitHub Actions workflow that automatically compiles your document and creates a GitHub Release whenever you push a new version tag.

## Features

- **Automated PDF Compilation**: Automatically compiles `main.typ` into `main.pdf` on every run.
- **Dependency Caching**: Uses `typst-community/setup-typst` to cache dependencies listed in `requirements.typ`, speeding up subsequent builds.
- **Automatic Releases**: Creates a new GitHub Release with the compiled PDF attached whenever a tag matching the `v*.*.*` pattern is pushed.
- **Manual Workflow Trigger**: Allows for manual builds directly from the GitHub Actions tab, perfect for testing changes without creating a release.

## File Structure

```bash
.
├── .github/
│   └── workflows/
│       └── release.yml     # The GitHub Actions workflow definition.
├── main.typ                # Your main Typst document entry point.
├── requirements.typ        # List your Typst package dependencies here.
├── bibliography.yml        # A sample bibliography file (if needed).
└── README.md               # This file.
```

- **`main.typ`**: This is the heart of your document. All your content should start here.
- **`requirements.typ`**: If your project uses external Typst packages, you should specify them here. The workflow will automatically fetch and cache them. For example:

```typst
#import "@preview/fletcher:0.5.8"
#import "@preview/physica:0.9.5"
```

- **`.github/workflows/release.yml`**: This file defines the Continuous Integration/Continuous Deployment (CI/CD) pipeline. See the "Workflow Breakdown" section below for a detailed explanation.

## How to Use This Template

There are two primary ways to use the automation provided in this template.

### Method 1: Creating a Release/Tag (Recommended)

This is the standard way to publish a new version of your document. The workflow will automatically create a GitHub Release and attach your compiled PDF.

1. **Commit Your Changes**: Make sure all your latest changes to the `.typ` files are committed to your main branch.

```bash
git add .
git commit -m "Finalize version 1.0.0"
```

2. **Tag the Version**: Create a new Git tag that follows the semantic versioning format (e.g., `v1.0.0`, `v1.2.3`).

```bash
git tag v1.0.0
```

3. **Push the Tag**: Push your commits and the new tag to GitHub.

```bash git push origin main
git push origin v1.0.0
```

Once the tag is pushed, the "Release" workflow will automatically start. It will compile `main.typ` and create a new release on your repository's "Releases" page, named `v1.0.0 — <current_date>`, with `main.pdf` attached as an asset.

### Method 2: Manual Build for Testing

If you want to compile the PDF to see the result without creating a public release, you can trigger the workflow manually.

1. Navigate to the **Actions** tab in your GitHub repository.
2. In the left sidebar, click on the **Release** workflow.
3. You will see a message: "This workflow has a `workflow_dispatch` event trigger." Click the **Run workflow** dropdown button.
4. You will be prompted to enter a `version` string. This is for informational purposes in the run log; you can enter any value (e.g., `test-build`).
5. Click the green **Run workflow** button.

The workflow will run, but the final "Release" step will be skipped. You can download the compiled `main.pdf` from the "Artifacts" section on the summary page for that workflow run.

## Workflow Breakdown (`release.yml`)

The automation is powered by the `.github/workflows/release.yml` file. Here is a step-by-step explanation of what it does.

### Triggers

The workflow is triggered by two events:

1. **`push: tags:`**: Runs automatically when a tag matching the pattern `v[0-9]+.[0-9]+.[0-9]+*` is pushed.
2. **`workflow_dispatch:`**: Allows the manual execution described above.

### Permissions

- `contents: write`: This is essential. It grants the workflow permission to create GitHub Releases and upload files (artifacts) to them.

### Job: `build`

The workflow consists of a single job named `build` that runs on an `ubuntu-latest` virtual machine.

- **`actions/checkout@v4`**: This step checks out your repository's code so the workflow can access your `.typ` files.

- **`typst-community/setup-typst@v4`**: This community action installs the specified version of Typst (`0.13`). The `cache-dependency-path` key is configured to look at `requirements.typ`, enabling caching of Typst packages to make future runs faster.

- **`Compile Typst files`**: A simple shell command that runs the Typst compiler, taking `main.typ` as input and producing `main.pdf`.

```bash
typst compile main.typ main.pdf
```

- **`Upload PDF file`**: This step uses `actions/upload-artifact@v4` to save the generated `main.pdf` as a workflow artifact. This is useful for every run, as it allows you to download the PDF even if a release isn't created.

- **`Get current date`**: Creates a timestamp that is used in the release name for uniqueness.

- **`softprops/action-gh-release@v1`**: This is the final step that creates the release.
  - `if: github.ref_type == 'tag'`: This crucial condition ensures this step **only runs if the workflow was triggered by a tag**. It is skipped during manual `workflow_dispatch` runs.
  - `name: "${{ github.ref_name }} — ${{ env.DATE }}"`: Sets the release title to the tag name (e.g., `v1.0.0`) plus the date.
  - `files: main.pdf`: Attaches the compiled `main.pdf` to the release.

## Customization

- **Typst Version**: To use a different version of Typst, simply change the `typst-version` in the `release.yml` file.
- **Main File**: If your main Typst file is not named `main.typ`, you will need to update the `Compile Typst files` and `Release` steps in `release.yml`.
- **Release Assets**: You can add more files to the release (e.g., a ZIP of the source code) by modifying the `files:` list in the `Release` step.
