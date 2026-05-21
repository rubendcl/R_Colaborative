# Reproducible R Projects with renv + GitHub Across Linux and Windows

This setup is one of the best ways to make R projects reproducible across operating systems and across time.

The main goals are:

- Reproduce the same package environment
- Reproduce the same outputs (.md, .html, .pdf)
- Avoid "it works on my machine"
- Handle different R versions and system dependencies correctly
- Collaborate through GitHub safely

---

## 1. General Architecture

A typical reproducible project should look like this:

```text
project/
│
├── renv/
├── renv.lock
├── .Rprofile
├── DESCRIPTION            (optional but recommended)
├── analysis.R
├── report.Rmd
├── data/
├── output/
└── README.md
```

Main components:

| Component    | Purpose                            |
|--------------|------------------------------------|
| `renv.lock`  | Snapshot of exact package versions |
| `renv/`      | Local project package library      |
| `.Rprofile`  | Auto-activates renv                |
| GitHub repo  | Version control + collaboration    |
| RMarkdown    | Reproducible reports               |

---

## 2. Linux PC (Original Machine)

Suppose:

- Linux machine
- R 4.5.3
- Project already works correctly

---

## 3. Proper Setup on Linux (Source Machine)

### Step 1 — Initialize renv

Inside the project:

```r
# Environment: R console / RStudio console
install.packages("renv")
renv::init()
```

This:

- Creates `renv/`
- Creates `.Rprofile`
- Creates `renv.lock`

### Step 2 — Install All Needed Packages

Example:

```r
# Environment: R console / RStudio console
install.packages(c(
  "rmarkdown",
  "knitr",
  "ggplot2",
  "dplyr",
  "readr"
))
```

### Step 3 — Snapshot Environment

Very important:

```r
# Environment: R console / RStudio console
renv::snapshot()
```

This records:

- Package versions
- Sources
- R version
- Dependencies

into:

`renv.lock`

### Step 4 — Verify Status

```r
# Environment: R console / RStudio console
renv::status()
```

Should show synchronized environment.

### Step 5 — Git Ignore Correctly

Create `.gitignore`

Recommended:

```gitignore
# Environment: .gitignore file
.Rhistory
.RData
.Ruserdata

renv/library/
renv/staging/

output/
```

**IMPORTANT:**

DO NOT ignore:

- `renv.lock`
- `.Rprofile`
- `renv/activate.R`

These are essential.

### Step 6 — Commit to GitHub

```bash
# Environment: Linux / macOS / Windows Git Bash terminal
git init
git add .
git commit -m "Initial reproducible project"
git branch -M main
git remote add origin YOUR_REPO
git push -u origin main
```

---

## 4. What Happens on Windows PC

Suppose:

- Windows
- Different/newer R version
- Empty machine

Goal:

- Reproduce project
- Reproduce outputs
- Use same package versions

---

## 5. Clone Project from GitHub

### Option A — Terminal (First Time)

```bash
# Environment: Linux / macOS / Windows Git Bash terminal
git clone REPO_URL
```

### Option B — RStudio GUI (First Time)

`File → New Project → Version Control → Git`

### Option C — Update Existing Project (Already Cloned)

If you already have the project folder from a previous clone and just need to update it with the latest changes from GitHub:

```bash
# Environment: Linux / macOS / Windows Git Bash terminal
# Navigate into the existing project folder first
cd path/to/existing/project
git pull
```

Then restore any updated packages:

```r
# Environment: R console / RStudio console
renv::restore()
```

**When to use this option:**

- You already cloned the repository before and the folder exists locally
- You want to get the latest `renv.lock` and code changes from collaborators
- You are working on the same machine where the project was originally set up

**Important:** If the `renv.lock` file was updated in the pull, run `renv::restore()` to synchronize your local package library with the new lockfile.

---

## 6. Install Required Base Software on Windows

### Essential Components

| Component | Required? |
|-----------|-----------|
| R | YES |
| RStudio | Recommended |
| Git | Recommended |
| Rtools | Often YES |
| TinyTeX / TeXLive | For PDF |

---

## 7. About R Version Differences

This is one of the **MOST** important topics.

Suppose:

| Machine | R Version |
|---------|-----------|
| Linux | 4.5.3 |
| Windows | 4.6.0 |

---

## 8. What Does renv Actually Reproduce?

**renv reproduces:**

- package versions

**BUT NOT necessarily:**

- the exact R version
- system libraries
- compilers
- LaTeX installations

This is critical.

---

## 9. Recommended Options for R Versions

### OPTION 1 — Use Same R Version (BEST)

Install same R version on Windows:

- R 4.5.3

**Advantages:**

- Maximum reproducibility
- Fewer surprises
- Same numerical behavior
- Same package binaries

**Best for:**

- scientific research
- regulatory work
- pharmacometrics
- validated workflows

### OPTION 2 — Use Newer R Version

Usually works.

**BUT:**

Possible issues:

- packages unavailable
- binary incompatibilities
- numerical differences
- changed defaults
- deprecated functions

---

## 10. How renv Behaves with Different R Versions

When restoring:

```r
# Environment: R console / RStudio console
renv::restore()
```

### Possible situations:

**Case A — Compatible**

Everything installs normally.

**Case B — Old Packages Not Available**

You may see:

```
package not available for this version of R
```

**Solutions:**

- **Solution 1:** Install older R version.
- **Solution 2:** Allow source compilation.
- **Solution 3:** Update package versions.

---

## 11. BEST PRACTICE for R Versions

For strict reproducibility:

> Use identical:
> - R version
> - renv.lock
> - package versions

Especially important for:

- simulations
- statistical models
- numerical optimization
- reports for publication

---

## 12. Restoring Environment on Windows

Inside project:

```r
# Environment: R console / RStudio console
install.packages("renv")
renv::restore()
```

This reads:

`renv.lock`

and installs all packages.

### Check Library Details

To see a detailed overview of the project's library state — which packages are installed, which are used, and if anything needs to be synchronized:

```r
# Check if environment is in sync
renv::status()

# Show installed packages in the project library
renv::dependencies()

# Show detailed package status (installed vs used vs missing)
renv::diagnostics()
```

| Command | What it shows |
|---------|---------------|
| `renv::status()` | Whether the project is in sync or needs updates |
| `renv::dependencies()` | All R packages used in the project code |
| `renv::diagnostics()` | Detailed report of installed, used, and missing packages |

---

## 13. Important Observation About Windows

Windows often needs:

**Rtools**

Especially for source package compilation.

---

## 14. What is Rtools?

Rtools is a compiler toolchain for Windows.

Needed for:

- compiling source packages
- packages with C/C++/Fortran
- development packages

---

## 15. When is Rtools Needed?

Usually when:

- binary package unavailable
- package must compile from source
- using GitHub packages
- package has native code

---

## 16. Rtools Version Compatibility

| R Version | Recommended Rtools |
|-----------|--------------------|
| R 4.5.x | Rtools45 |
| R 4.4.x | Rtools44 |

Mismatch can create problems.

---

## 17. Options Regarding Rtools

### OPTION 1 — Install Matching Rtools (BEST)

Recommended.

### OPTION 2 — Use Only Binary Packages

Possible if binaries exist.

Then Rtools may not be necessary.

### OPTION 3 — Docker / Containers

Most reproducible approach.

Avoids many Windows issues.

More advanced.

---

## 18. Installing TinyTeX for PDF

For PDF generation:

```r
# Environment: R console / RStudio console
install.packages("tinytex")
tinytex::install_tinytex()
```

This installs:

- TinyTeX

---

## 19. PDF Problems Are VERY Common

Common errors:

| Error | Cause |
|-------|-------|
| xelatex not found | No LaTeX |
| missing sty files | incomplete installation |
| Unicode errors | pdflatex limitations |
| font issues | OS differences |

---

## 20. Recommended YAML for PDF

Use:

```yaml
# Environment: YAML front matter in .Rmd file
output:
  pdf_document:
    latex_engine: xelatex
```

because:

- better Unicode support
- better fonts
- cross-platform stability

---

## 21. Different Ways to Render Rmd

Suppose file:

`report.Rmd`

There are **MANY** ways.

---

## 22. Method 1 — RStudio Knit Button

Simplest.

Press:

`Knit`

Produces:

- HTML
- PDF
- Word
- MD

depending on YAML.

> **⚠️ Important:** The Knit button runs the `.Rmd` in a separate R session. To guarantee that `renv` is activated (and the correct packages are found), add this **setup chunk** at the top of your `.Rmd` file:
>
> ````markdown
> ```{r setup, include=FALSE}
> source("renv/activate.R")
> ```
> ````
>
> This ensures `renv` is loaded before any packages are used, regardless of how the file is rendered.

---

## 23. Method 2 — rmarkdown::render()

Most reproducible.

Example:

```r
# Environment: R console / RStudio console
rmarkdown::render("report.Rmd")
```

Specific output:

```r
# Environment: R console / RStudio console
rmarkdown::render(
  "report.Rmd",
  output_format = "pdf_document"
)
```

---

## 24. Method 3 — Render All Formats

YAML:

```yaml
# Environment: YAML front matter in .Rmd file
output:
  html_document: default
  pdf_document: default
  github_document: default
```

Then:

```r
# Environment: R console / RStudio console
rmarkdown::render("report.Rmd", output_format = "all")
```

---

## 25. Method 4 — Knit Function

Lower-level approach:

```r
# Environment: R console / RStudio console
knitr::knit("report.Rmd")
```

Produces markdown.

Usually not enough alone for HTML/PDF.

---

## 26. Method 5 — Terminal Command (Rscript)

**Linux:**

```bash
# Environment: Linux terminal (bash)
Rscript -e "rmarkdown::render('report.Rmd')"
```

**Windows CMD:**

```cmd
REM Environment: Windows Command Prompt (CMD)
Rscript -e "rmarkdown::render('report.Rmd')"
```

**PowerShell:**

```powershell
# Environment: Windows PowerShell
Rscript -e "rmarkdown::render('report.Rmd')"
```

Excellent for:

- automation
- CI/CD
- GitHub Actions

---

## 27. Method 6 — Direct pandoc

Possible but less recommended.

RStudio already includes Pandoc.

---

## 28. Using Visual Studio Code for R and RMarkdown

Visual Studio Code is an excellent alternative to RStudio for working with R and RMarkdown files. With the right extensions, you can use the Knit button and the integrated terminal just like in RStudio.

### 28.1 Required Extensions

Install these extensions from the VS Code marketplace:

| Extension | ID | Purpose |
|-----------|-----|---------|
| **R** (by REditorSupport) | `reditorsupport.r` | Language support, syntax highlighting, code completion |
| **R Markdown** (by Yuki Ueda) | `yukiyudi.r-markdown` | RMarkdown syntax highlighting and preview |
| **markdownlint** (by David Anson) | `davidanson.markdownlint` | Markdown linting and formatting |

To install them, press `Ctrl+Shift+X`, search for each extension, and click **Install**.

### 28.2 Configure R Path in VS Code

VS Code needs to know where R is installed.

**On Linux:**

R is usually in `/usr/bin/R` or `/usr/local/bin/R`. Verify with:

```bash
# Environment: Linux terminal (bash)
which R
```

**On Windows:**

R is typically installed in:

```
C:\Program Files\R\R-4.x.x\bin\x64\R.exe
```

**Configure in VS Code settings:**

1. Press `Ctrl+Shift+P` → `Preferences: Open Settings (JSON)`
2. Add or modify:

```json
# Environment: VS Code settings.json
{
  "r.rpath.windows": "C:\\Program Files\\R\\R-4.5.3\\bin\\x64\\R.exe",
  "r.rpath.linux": "/usr/bin/R",
  "r.alwaysUseActiveTerminal": true,
  "r.bracketedPaste": true,
  "r.sessionWatcher": true
}
```

Or use the Settings UI (`Ctrl+,`), search for `r.rpath`, and set the path there.

### 28.3 Configure RMarkdown Knit Button in VS Code

The **R Markdown** extension adds a **Knit** button to the editor toolbar when you open an `.Rmd` file.

**To verify it works:**

1. Open an `.Rmd` file in VS Code
2. Look for the **Knit** button in the top-right corner of the editor (or press `Ctrl+Shift+K`)
3. Click it to render the document

**If the Knit button does not appear:**

- Make sure both the **R** and **R Markdown** extensions are installed and enabled
- Reload VS Code (`Ctrl+Shift+P` → `Developer: Reload Window`)
- Check that the R path is correctly configured (see section 28.2)
- Ensure `rmarkdown` package is installed in R:

```r
# Environment: R console / RStudio console
install.packages("rmarkdown")
```

### 28.4 Alternative: Use Command Palette to Knit

If the Knit button is missing, you can still render via the command palette:

1. Open an `.Rmd` file
2. Press `Ctrl+Shift+P`
3. Type **R Markdown: Render Document** and press Enter

This achieves the same result as clicking the Knit button.

### 28.5 Important: Does the VS Code Knit Button Activate renv?

This is a critical question. The VS Code Knit button (from the R Markdown extension) runs `rmarkdown::render()` in a **separate R process** behind the scenes. Whether `renv` is activated depends on **where that R process starts**.

**✅ When it works correctly:**

If the R process is launched from the project's root directory (where `.Rprofile` lives), then `.Rprofile` is read, `renv` is activated, and packages are found in the `renv/library` folder. This is the expected behavior.

**⚠️ When it might fail:**

- If VS Code's integrated terminal is not open to the project root
- If the extension launches R from a different working directory
- If the `.Rprofile` file is missing or misconfigured

**🔍 How to verify that renv is active when using the Knit button:**

Add a temporary test chunk to your `.Rmd` file:

````markdown
```{r test-renv}
# Check if renv is active
renv::status()
# Check where packages are being loaded from
.libPaths()
```
````

When you click Knit, the output should show:
- `renv::status()` — confirms the project library is being used
- `.libPaths()` — should include the `renv/library` path

**🛡️ Best practice: Use the terminal instead for guaranteed renv activation**

If you want to be **100% sure** that `renv` is activated, use the integrated terminal instead of the Knit button:

```bash
# Environment: VS Code integrated terminal (any OS)
# This guarantees .Rprofile is read and renv is activated
Rscript -e "rmarkdown::render('report.Rmd')"
```

The integrated terminal inherits the project's working directory, so `.Rprofile` is always read correctly.

**💡 Pro tip: Configure the Knit button to use the active terminal**

In VS Code settings, enable:

```json
# Environment: VS Code settings.json
{
  "r.alwaysUseActiveTerminal": true
}
```

This tells the R extension to use the currently active R terminal (if one is open) instead of launching a new R process. If you have an R terminal open in the project root, `renv` will be active.

### 28.6 Using the VS Code Integrated Terminal for R

VS Code's integrated terminal (`Ctrl+`` ` or `View → Terminal`) is fully functional for running R commands.

**Open an R session in the terminal:**

```bash
# Environment: VS Code integrated terminal (Linux/macOS)
R
```

**Run R scripts directly:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript analysis.R
```

**Render RMarkdown from the terminal:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "rmarkdown::render('report.Rmd')"
```

**Render specific output format:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "rmarkdown::render('report.Rmd', output_format = 'pdf_document')"
```

**Render all formats defined in YAML:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "rmarkdown::render('report.Rmd', output_format = 'all')"
```

**Generate markdown only (for GitHub):**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "rmarkdown::render('report.Rmd', output_format = 'github_document')"
```

### 28.7 Using the VS Code Terminal with renv

The integrated terminal respects the project's `.Rprofile`, so `renv` is automatically activated when you run R from within the project directory.

**Restore environment:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "renv::restore()"
```

**Snapshot environment:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "renv::snapshot()"
```

**Check status:**

```bash
# Environment: VS Code integrated terminal (any OS)
Rscript -e "renv::status()"
```

### 28.8 Setting Up VS Code Tasks for Automation

You can create VS Code tasks to automate rendering with a keyboard shortcut.

Create `.vscode/tasks.json` in your project:

```json
# Environment: VS Code tasks.json configuration file
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Render RMarkdown (HTML)",
      "type": "shell",
      "command": "Rscript -e \"rmarkdown::render('${file}', output_format = 'html_document')\"",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": []
    },
    {
      "label": "Render RMarkdown (PDF)",
      "type": "shell",
      "command": "Rscript -e \"rmarkdown::render('${file}', output_format = 'pdf_document')\"",
      "group": "build",
      "problemMatcher": []
    },
    {
      "label": "Render RMarkdown (All Formats)",
      "type": "shell",
      "command": "Rscript -e \"rmarkdown::render('${file}', output_format = 'all')\"",
      "group": "build",
      "problemMatcher": []
    },
    {
      "label": "renv::restore()",
      "type": "shell",
      "command": "Rscript -e \"renv::restore()\"",
      "group": "none",
      "problemMatcher": []
    },
    {
      "label": "renv::snapshot()",
      "type": "shell",
      "command": "Rscript -e \"renv::snapshot()\"",
      "group": "none",
      "problemMatcher": []
    }
  ]
}
```

Then run a task via `Ctrl+Shift+P` → `Tasks: Run Task` → select the task.

### 28.9 VS Code Workspace Settings for R Projects

Create `.vscode/settings.json` in your project for consistent settings across collaborators:

```json
# Environment: VS Code settings.json configuration file
{
  "r.rpath.linux": "/usr/bin/R",
  "r.rpath.windows": "C:\\Program Files\\R\\R-4.5.3\\bin\\x64\\R.exe",
  "r.alwaysUseActiveTerminal": true,
  "r.bracketedPaste": true,
  "r.sessionWatcher": true,
  "files.exclude": {
    "renv/library": true,
    "renv/staging": true,
    "renv/python": true,
    "renv/sandbox": true
  },
  "editor.renderWhitespace": "boundary",
  "files.encoding": "utf8"
}
```

### 28.10 VS Code + Git Integration

VS Code has built-in Git support:

- **Source Control** tab (`Ctrl+Shift+G`) shows changes
- Stage files with `+` icon
- Write commit messages and press `Ctrl+Enter` to commit
- Push/Pull via the `...` menu

This is especially useful for committing `renv.lock` after `renv::snapshot()`.

### 28.11 Summary: VS Code vs RStudio for Reproducible R Projects

| Feature | RStudio | VS Code |
|---------|---------|---------|
| Knit button | Built-in | Via R Markdown extension |
| Integrated terminal | Built-in | Built-in |
| Git integration | Built-in | Built-in |
| renv support | Automatic | Automatic (via terminal) |
| Tasks automation | Limited | Powerful (tasks.json) |
| Extensions | Limited | Rich ecosystem |
| R debugging | Excellent | Good (with R extension) |

Both work well. VS Code is particularly useful if you already use it for other languages and want a unified editor.

---

## 29. Important Difference Between knit and render

| Function | Purpose |
|----------|---------|
| `knitr::knit()` | Converts Rmd → md |
| `rmarkdown::render()` | Full rendering pipeline |

Usually prefer:

`rmarkdown::render()`

---

## 30. Generating Markdown Only

Useful for GitHub README.

YAML:

```yaml
# Environment: YAML front matter in .Rmd file
output:
  github_document: default
```

Then:

```r
# Environment: R console / RStudio console
rmarkdown::render("report.Rmd")
```

Produces:

`report.md`

---

## 31. HTML Generation

Most portable.

```yaml
# Environment: YAML front matter in .Rmd file
output:
  html_document:
    self_contained: true
```

**Advantages:**

- fewer dependencies
- easier sharing

---

## 32. PDF Generation Challenges

PDF is hardest because of:

- LaTeX
- fonts
- Unicode
- OS differences

---

## 33. Cross-Platform Font Problems

Linux and Windows have different fonts.

Can affect:

- spacing
- line breaks
- plot rendering

---

## 34. Good Practice for Fonts

Explicitly define fonts when needed.

Or avoid OS-dependent fonts.

---

## 35. System Dependencies Problem

Some packages depend on OS libraries.

Examples:

| Package | System Dependency |
|---------|-------------------|
| xml2 | libxml2 |
| sf | GDAL |
| curl | libcurl |
| rJava | Java |

Linux and Windows differ greatly here.

---

## 36. Common Windows Challenges

| Problem | Cause |
|---------|-------|
| package compile fails | no Rtools |
| DLL locked | package loaded |
| path issues | spaces |
| encoding issues | locale |

---

## 37. Important Practice — Use Relative Paths

**BAD:**

```r
# Environment: R console / RStudio console
read.csv("C:/Users/...")
```

**GOOD:**

```r
# Environment: R console / RStudio console
read.csv("data/file.csv")
```

Or:

```r
# Environment: R console / RStudio console
here::here("data", "file.csv")
```

---

## 38. Recommended Package for Paths

`here`

Example:

```r
# Environment: R console / RStudio console
here::here("output", "plot.png")
```

Very important for GitHub collaboration.

---

## 39. Lockfile Maintenance

Whenever packages change:

```r
# Environment: R console / RStudio console
renv::snapshot()
```

Then commit:

```bash
# Environment: Linux / macOS / Windows Git Bash terminal
git add renv.lock
git commit -m "Update packages"
```

---

## 40. NEVER Manually Edit renv.lock

Always use:

```r
# Environment: R console / RStudio console
renv::snapshot()
```

---

## 41. Recommended Workflow

### On Linux

```r
# Environment: R console / RStudio console (on Linux)
renv::snapshot()
```

Push to GitHub.

### On Windows

```bash
# Environment: Linux / macOS / Windows Git Bash terminal
git pull
```

```r
# Environment: R console / RStudio console (on Windows)
renv::restore()
```

---

## 42. About Reproducing Results Exactly

Even with same packages:

Possible differences:

- BLAS/LAPACK
- floating point behavior
- random seeds
- parallelization
- CPU architecture

---

## 43. For Exact Numerical Reproducibility

Set seeds:

```r
# Environment: R console / RStudio console
set.seed(123)
```

Avoid uncontrolled parallelism.

Document:

- OS
- R version
- package versions

---

## 44. Best Practice — Session Info

Save:

```r
# Environment: R console / RStudio console
sessionInfo()
```

or:

```r
# Environment: R console / RStudio console
devtools::session_info()
```

Useful for debugging.

---

## 45. Advanced Option — Docker

Most reproducible approach.

**Advantages:**

- same OS
- same R
- same libraries
- same compilers

**Disadvantages:**

- complexity
- learning curve

Very valuable for:

- production
- research
- validated pipelines

---

## 46. Extremely Recommended Additions

Add `README.md`

Document:

- required R version
- how to restore
- how to render
- dependencies

---

## 47. Recommended README Example

```markdown
# Environment: README.md file (Markdown)
# Project

## Restore environment

```r
install.packages("renv")
renv::restore()
```

## Render report

```r
rmarkdown::render("report.Rmd")
```
```

---

## 48. Continuous Integration (Advanced)

You can automate rendering using GitHub Actions.

Useful for:

- automatic reports
- testing reproducibility
- validation

---

## 49. Recommended Minimal Workflow

### Linux machine

```r
# Environment: R console / RStudio console (on Linux)
renv::snapshot()
```

Commit:

```bash
# Environment: Linux / macOS / Windows Git Bash terminal
git add .
git commit -m "update"
git push
```

### Windows machine

```r
# Environment: R console / RStudio console (on Windows)
install.packages("renv")
renv::restore()

rmarkdown::render("report.Rmd")
```

---

## 50. Most Common Causes of Failure

| Problem | Typical Fix |
|---------|-------------|
| package not found | install older R |
| compile error | install Rtools |
| PDF fails | install TinyTeX |
| paths fail | use relative paths |
| different results | set.seed |
| encoding errors | UTF-8 |

---

## 51. Final Recommendations

### For the BEST reproducibility

**Strong Recommendation**

Use identical:

- R version
- renv.lock
- package versions
- TinyTeX version

and preferably:

- same OS

### Practical Recommendation

If exact reproducibility is not critical:

- newer R is usually acceptable
- `renv::restore()` solves most issues

### Professional / Scientific Recommendation

Use:

- renv
- GitHub
- RMarkdown
- Docker (optional but excellent)

This is currently one of the standard professional workflows in modern R projects.
