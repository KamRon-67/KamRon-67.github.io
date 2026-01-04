# Local Testing Guide

This guide explains how to set up and run this Jekyll site on your local machine for development and testing.

## Prerequisites

Before you begin, ensure you have the following installed:
- **Ruby** (Version 3.0 or higher recommended)
- **Bundler** (`gem install bundler`)

## Setup

1. **Install Dependencies**:
   Open your terminal in the project root and run:
   ```bash
   bundle install
   ```

2. **(Optional) Install Node.js dependencies**:
   If you plan to modify JavaScript or CSS source files:
   ```bash
   npm install
   ```

## Running the Site

### Method 1: Using the provided script (Recommended)
This script runs the server and enables live reloading:
```bash
./tools/run.sh
```

### Method 2: Manual command
```bash
bundle exec jekyll serve
```

Once the server is running, you can view the site at:
[http://localhost:4000](http://localhost:4000)

## Common Commands

- **Build once**: `bundle exec jekyll build`
- **Check for configuration issues**: `bundle exec jekyll doctor`
- **Clean the build cache**: `bundle exec jekyll clean`

## Troubleshooting

- **SASS Deprecation Warnings**: You may see warnings about `@import`. These are expected due to the theme version but do not prevent the site from building.
- **Port already in use**: If `4000` is taken, use `bundle exec jekyll serve --port 4001`.
- **Changes not showing**: Run `bundle exec jekyll clean` and then restart the server.
