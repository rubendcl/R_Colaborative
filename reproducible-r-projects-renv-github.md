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

```
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

```

Main components:

| Component | Purpose |
|-----------|---------|
| `renv.lock` | Snapshot of exact package versions |
| `renv/` | Local project package library |
| `.Rprofile` | Auto-activates renv |
| GitHub repo | Version control + collaboration |
| RMarkdown | Reproducible reports |
```

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
renv::status()
```

Should show synchronized environment.

### Step 5 — Git Ignore Correctly

Create `.gitignore`

Recommended:

```
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

### Option A — Terminal

```bash
git clone REPO_URL
```

### Option B — RStudio GUI

`File → New Project → Version Control → Git`

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
install.packages("renv")
renv::restore()
```

This reads:

`renv.lock`

and installs all packages.

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

---

## 23. Method 2 — rmarkdown::render()

Most reproducible.

Example:

```r
rmarkdown::render("report.Rmd")
```

Specific output:

```r
rmarkdown::render(
  "report.Rmd",
  output_format = "pdf_document"
)
```

---

## 24. Method 3 — Render All Formats

YAML:

```yaml
output:
  html_document: default
  pdf_document: default
  github_document: default
```

Then:

```r
rmarkdown::render("report.Rmd", output_format = "all")
```

---

## 25. Method 4 — Knit Function

Lower-level approach:

```r
knitr::knit("report.Rmd")
```

Produces markdown.

Usually not enough alone for HTML/PDF.

---

## 26. Method 5 — Terminal Command (Rscript)

**Linux:**

```bash
Rscript -e "rmarkdown::render('report.Rmd')"
```

**Windows CMD:**

```cmd
Rscript -e "rmarkdown::render('report.Rmd')"
```

**PowerShell:**

```powershell
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
install.packages("rmarkdown")
```

### 28.4 Alternative: Use Command Palette to Knit

If the Knit button is missing, you can still render via the command palette:

1. Open an `.Rmd` file
2. Press `Ctrl+Shift+P`
3. Type **R Markdown: Render Document** and press Enter

This achieves the same result as clicking the Knit button.

### 28.5 Using the VS Code Integrated Terminal for R

VS Code's integrated terminal (`Ctrl+`` ` or `View → Terminal`) is fully functional for running R commands.

**Open an R session in the terminal:**

```bash
R
```

**Run R scripts directly:**

```bash
Rscript analysis.R
```

**Render RMarkdown from the terminal:**

```bash
Rscript -e "rmarkdown::render('report.Rmd')"
```

**Render specific output format:**

```bash
Rscript -e "rmarkdown::render('report.Rmd', output_format = 'pdf_document')"
```

**Render all formats defined in YAML:**

```bash
Rscript -e "rmarkdown::render('report.Rmd', output_format = 'all')"
```

**Generate markdown only (for GitHub):**

```bash
Rscript -e "rmarkdown::render('report.Rmd', output_format = 'github_document')"
```

### 28.6 Using the VS Code Terminal with renv

The integrated terminal respects the project's `.Rprofile`, so `renv` is automatically activated when you run R from within the project directory.

**Restore environment:**

```bash
Rscript -e "renv::restore()"
```

**Snapshot environment:**

```bash
Rscript -e "renv::snapshot()"
```

**Check status:**

```bash
Rscript -e "renv::status()"
```

### 28.7 Setting Up VS Code Tasks for Automation

You can create VS Code tasks to automate rendering with a keyboard shortcut.

Create `.vscode/tasks.json` in your project:

```json
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

### 28.8 VS Code Workspace Settings for R Projects

Create `.vscode/settings.json` in your project for consistent settings across collaborators:

```json
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

### 28.9 VS Code + Git Integration

VS Code has built-in Git support:

- **Source Control** tab (`Ctrl+Shift+G`) shows changes
- Stage files with `+` icon
- Write commit messages and press `Ctrl+Enter` to commit
- Push/Pull via the `...` menu

This is especially useful for committing `renv.lock` after `renv::snapshot()`.

### 28.10 Summary: VS Code vs RStudio for Reproducible R Projects

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
output:
  github_document: default
```

Then:

```r
rmarkdown::render("report.Rmd")
```

Produces:

`report.md`

---

## 31. HTML Generation

Most portable.

```yaml
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
read.csv("C:/Users/...")
```

**GOOD:**

```r
read.csv("data/file.csv")
```

Or:

```r
here::here("data", "file.csv")
```

---

## 38. Recommended Package for Paths

`here`

Example:

```r
here::here("output", "plot.png")
```

Very important for GitHub collaboration.

---

## 39. Lockfile Maintenance

Whenever packages change:

```r
renv::snapshot()
```

Then commit:

```bash
git add renv.lock
git commit -m "Update packages"
```

---

## 40. NEVER Manually Edit renv.lock

Always use:

```r
renv::snapshot()
```

---

## 41. Recommended Workflow

### On Linux

```r
renv::snapshot()
```

Push to GitHub.

### On Windows

```bash
git pull
```

```r
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
sessionInfo()
```

or:

```r
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
renv::snapshot()
```

Commit:

```bash
git add .
git commit -m "update"
git push
```

### Windows machine

```r
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
