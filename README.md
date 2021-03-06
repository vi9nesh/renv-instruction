Package Development with {renv}
================

## Project Setup

One user (perhaps yourself) should explicitly initialize `{renv}` in the
project, via following below steps:

### User Consent

In accordance with the [CRAN Repository
Policy](https://cran.r-project.org/web/packages/policies.html), `{renv}`
must first obtain consent from you (the user) before these actions can
be taken. Please provide consent to `{renv}`, allowing it to write and
update certain files on your filesystem. Generally, this is required for
the first time user.

``` r
renv::consent(TRUE)
```

### Infrastructure

By default, when building a package tarball, R will copy all files
within the package directory to a temporary build directory before
building the package. Unfortunately, this also implies copying the
`{renv}` library of a project. When that library is large, this can
dramatically increase the amount of time it takes to build your package.

One way to resolve this issue is to force `{renv}` to use a library path
that lives outside of your project.

``` r
Sys.setenv(RENV_PATHS_LIBRARY_ROOT = "~/mywarpscratch/.renv/library")
```

Use,

``` r
renv::init()
```

to write the infrastructure needed to ensure that newly-launched R
projects will load the project’s private library on launch, alongside
any other project-specific state recorded for the project.

The following files are written to and used by projects using `{renv}`:

| **File**          | **Usage**                                                                           |
| ----------------- | ----------------------------------------------------------------------------------- |
| `.Rprofile`       | Used to activate `{renv}` for new R sessions launched in the project.               |
| `renv.lock`       | The lockfile, describing the state of your project’s library at some point in time. |
| `renv/activate.R` | The activation script run by the project `.Rprofile`.                               |
| `renv/library`    | The private project library.                                                        |

`.Rprofile`, `renv.lock` and `renv/activate.R` files should be committed
to your version control system for development and collaboration.

### Global Cache

Use a global cache of R packages. When active, `{renv}` will install
packages into a global cache, and link packages from the cache into your
`{renv}` projects library as appropriate. This can greatly save on disk
space and installation time for R packages which are used across
multiple projects.

``` r
# Setup cache
Sys.setenv(RENV_PATHS_CACHE = "~/mywarpscratch/.cache/renv")
renv::settings$use.cache(TRUE)

# Install `{renv}` in global cache
renv::install("renv")
```

### Setup `.Rprofile` file

Store the following code inside your project `.Rprofile` file to reduce
problems with cache (Open `.Rprofile` with:
`usethis::edit_r_profile(scope = "project")`). Add code in the same
order:

``` r
# -- Set private project library --
Sys.setenv(RENV_PATHS_LIBRARY_ROOT = "~/mywarpscratch/.renv/library")

# -- Activate for newly launched R sessions --
source("renv/activate.R")

# -- Set global cache directory --
Sys.setenv(RENV_PATHS_CACHE = "~/mywarpscratch/.cache/renv")
renv::settings$use.cache(TRUE)
```

### Install Package Dependencies

Often, R packages will have other R packages as dependencies. For this,
one must declare their R package dependencies within the package
DESCRIPTION file. In order to prepare your environment for package
development, use:

``` r
renv::install()
```

to install the packages as declared in the package’s DESCRIPTION file.
It’s an best practice to always build and test your package against the
latest versions of packages available on CRAN. For this, you should
consider using:

``` r
renv::update()
```

to ensure your package dependencies are up-to-date, as appropriate.

### Save Project State to `renv.lock` File.

``` r
renv::snapshot()

# Add-on development packages
dev_pkgs <- c("renv", "knitr", "devtools", "roxygen2", 
              "usethis", "testthat", "covr", "pkgdown")
renv::snapshot(packages = dev_pkgs)
```

Finally, everything should be committed and pushed to your version
control system for collaboration.

## Collaboration

On every new R session, all collaborator should use the following block
in the same order to restore `{renv}` project library locally on their
machine.

``` r
renv::activate()
renv::restore()
```

### Updating the Lockfile

While working on a project, you or your collaborators may need to update
or install new packages in your project. When this occurs, you’ll also
want to ensure your collaborators are then using the same
newly-installed packages. In general, the process looks like this:

1.  A user installs, or updates, one or more packages in their local
    project library using `renv::install` or `renv::update`;
2.  That user calls `renv::snapshot()` to update the `renv.lock`
    lockfile;
3.  That user then shares the updated version of `renv.lock` with their
    collaborators;
4.  Other collaborators then call `renv::restore()` to install the
    packages specified in the newly-updated lockfile.
