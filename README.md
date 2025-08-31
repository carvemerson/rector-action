# Run Rector PHP (GitHub Action)

A simple composite action to run Rector on your PHP project with caching and flexible configuration.

It will:
- Check out your repository (actions/checkout@v4) with fetch-depth: 2.
- Set up PHP (default 8.3) and Composer v2.
- Install dependencies using `ramsey/composer-install` with cache.
- Cache Rector's `.rector-cache` directory keyed by OS, `rector.php`, and `composer.lock`.
- Show the Rector version and run Rector in check mode (dry-run) by default.

## Usage

Basic usage (checks for refactoring/applicability and fails the job on changes/violations):

```yaml
name: Lint (Rector)

on:
  push:
  pull_request:

jobs:
  rector:
    runs-on: ubuntu-latest
    steps:
      - name: Run Rector
        uses: carvemerson/rector-action@v1
```

Using inputs and running from a subdirectory:

```yaml
jobs:
  rector:
    runs-on: ubuntu-latest
    steps:
      - name: Run Rector
        uses: carvemerson/rector-action@v1
        with:
          php-version: '8.3'
          working-directory: ./backend
          rector-args: '--dry-run --ansi'   # default; change to e.g. "--no-diffs" or remove --dry-run to apply fixes
          composer-options: '--no-scripts'
          continue-on-error: 'false'        # set to 'true' to not fail the Rector step on findings
```

## Inputs

- php-version
  - Description: PHP version used to run Rector
  - Required: false
  - Default: 8.3

- working-directory
  - Description: Directory containing composer.json and rector.php
  - Required: false
  - Default: .

- rector-args
  - Description: Extra arguments passed to Rector (e.g., --no-diffs). By default the action runs in check mode without modifying files.
  - Required: false
  - Default: `--dry-run --ansi`

- composer-options
  - Description: Extra options for `composer install` (e.g., `--no-scripts`)
  - Required: false
  - Default: empty

- continue-on-error
  - Description: Whether to continue on error for the Rector run step in this action.
  - Required: false
  - Default: false

## Notes

- You do not need to add a separate actions/checkout step in your workflow; this action already checks out your repository (fetch-depth: 2).
- Requirements: your project must include Rector as a dependency, typically via `composer require rector/rector --dev`. The action expects `vendor/bin/rector` to exist within the working directory.
- The cache step stores `.rector-cache` to speed up subsequent runs; it is keyed by OS and the hashes of `rector.php` and `composer.lock`.
- Input continue-on-error controls whether the Rector step continues on error within this action. Alternatively, you can use GitHub Actions' step-level `continue-on-error` on the calling workflow step as shown below.
