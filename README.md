# 📦 SFDX Package Installation

This repository implements a simple GitHub composite action for installing packages on a specific Salesforce target org.

## Usage

After installing the SF CLI and authorizing the relevant org in your GitHub workflow, packages can be installed using this action as follows:

```yaml
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    permissions:
      contents: read # least privilege - the installation itself needs no write access
    steps:
      - name: Checkout
        uses: actions/checkout@v7.0.1
        with:
          persist-credentials: false # the GITHUB_TOKEN is not needed after the checkout - don't leave it in .git/config

      - name: Select Node Version
        uses: svierk/get-node-version@v1.5.2

      - name: Install Dependencies
        run: npm ci

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@v1.2.0

      - name: Salesforce Org Login
        uses: svierk/sfdx-login@v1.4.3
        with:
          sfdx-url: ${{ secrets.SFDX_AUTH_URL }}

      - name: Package Installation
        uses: svierk/sfdx-package-installation@v1.4.0
        with:
          packages: '04t6S000001UjutQAC,04t3y000000X0OaAAK'
          wait: 30
          publish-wait: 20
```

Every action above is pinned to an exact release tag instead of a moving reference like `@main`, so a workflow run is reproducible and an update is always an explicit, reviewable change.

### List input (comma-separated or multi-line)

The `packages` input accepts a comma-separated string as well as a multi-line block, and the two notations can be mixed freely within one step. A comma-separated string keeps short lists on a single line, a block stays readable once a list grows:

```yaml
- name: Package Installation
  uses: svierk/sfdx-package-installation@v1.4.0
  with:
    packages: | # multi-line block
      04t6S000001UjutQAC
      04t3y000000X0OaAAK
      MyPackage@1.2.0-1
```

Surrounding whitespace and blank lines are ignored, so `'04t6S000001UjutQAC, 04t3y000000X0OaAAK'` is read exactly like the tightly packed `'04t6S000001UjutQAC,04t3y000000X0OaAAK'`. Packages are installed one after another in the given order, so list dependencies before the packages that need them.

The JSON array notation used up to v1.2.x, e.g. `"['04t6S000001UjutQAC','04t3y000000X0OaAAK']"`, is still supported, so existing workflows keep working unchanged. A malformed input, such as an unclosed array, fails the step with an explicit error message instead of installing something unintended.

If the packages you want to install are key-protected, you can simply append the required installation key for each package with an underscore, e.g. `04t6S000001UjutQAC_INSTALLATIONKEY`. Since an installation key is a credential, keep it in a [GitHub Actions secret](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions) rather than in the workflow file:

```yaml
- name: Package Installation
  uses: svierk/sfdx-package-installation@v1.4.0
  with:
    packages: |
      04t6S000001UjutQAC_${{ secrets.SFDX_PACKAGE_KEY }}
      04t3y000000X0OaAAK
```

The action masks every installation key in the job log and never prints it, so keys stay hidden even if they are passed as a literal value. Since commas and line breaks separate the list entries, an installation key must not contain either of them.

### Pause between installations

Right after an installation, Salesforce can still hold locks on the metadata of that package, and the next installation then fails with an error like `Record Currently Unavailable, Details: The record you are attempting to edit, or one of its related records, is currently being modified by another user`. If you run into this, for example while provisioning a scratch org, the `install-delay` input adds a pause between two installations:

```yaml
- name: Package Installation
  uses: svierk/sfdx-package-installation@v1.4.0
  with:
    packages: '04t6S000001UjutQAC,04t3y000000X0OaAAK'
    wait: 30
    install-delay: 60 # seconds
```

There is no pause before the first or after the last package, so a single package never takes longer. The pause only helps if each installation has finished when it starts, so set `wait` high enough: with the default `wait` of `0`, the CLI returns while the installation is still running.

The following actions were also used in the example workflow to create the prerequisites for the package installation:

- [Get Node Version](https://github.com/svierk/get-node-version) | Pulls Node.js version to be used from the _package.json_ of the project
- [SFDX CLI Setup](https://github.com/svierk/sfdx-cli-setup) | Installs the Salesforce CLI and related plugins
- [SFDX Login](https://github.com/svierk/sfdx-login) | Handles Salesforce login using a Salesforce DX authorization URL

Of course, the package installation action can be used flexibly and the respective approach can vary.

## Inputs

| Name           | Required | Default | Description                                                                                                  |
| -------------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------- |
| `packages`     | yes      |         | Packages to install as a comma-separated or multi-line list of aliases or IDs, e.g. `04t...,04t...`. Append `_<key>` to a package for key-protected installs. The JSON array notation `"['04t...','04t...']"` is still supported. |
| `target-org`   | no       |         | Username or alias of the target org. Not required if the default org is set.                               |
| `api-version`  | no       |         | Override the api version used for api requests, e.g. `68.0`.                                               |
| `wait`         | no       | `0`     | Number of minutes to wait for installation status.                                                         |
| `publish-wait` | no       | `0`     | Maximum minutes to wait for the Subscriber Package Version ID to become available before canceling.        |
| `install-delay` | no      | `0`     | Number of seconds to pause between two package installations, e.g. to let metadata locks of the previous package clear. |
| `step-summary` | no       | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-package-installation/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-package-installation/blob/main/LICENSE).