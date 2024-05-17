# 📦 SFDX Package Installation

This repository implements a simple GitHub composite action for installing packages on a specific Salesforce target org.

## Usage

After installing the SF CLI and authorizing the relevant org in your GitHub workflow, packages can be installed using this action as follows:

```
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Select Node Version
        uses: svierk/get-node-version@main

      - name: Install Dependencies
        run: npm ci
      
      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@main
        
      - name: Salesforce Org Login
        uses: svierk/sfdx-login@main
        with:
          sfdx-url: ${{ secrets.SFDX_AUTH_URL }}
    
      - name: Package Installation
        uses: svierk/sfdx-package-installation@main
        with:
          packages: "['04t6S000001UjutQAC','04t3y000000X0OaAAK']"
          wait: 30
          publish-wait: 20
```

If the packages you want to install are key-protected, you can simply append the required installation key for each package with an underscore, e.g. '04t6S000001UjutQAC_INSTALLATIONKEY'.

The following actions were also used in the example workflow to create the prerequisites for the package installation:

- [Get Node Version](https://github.com/svierk/get-node-version) | Pulls Node.js version to be used from the _package.json_ of the project
- [SFDX CLI Setup](https://github.com/svierk/sfdx-cli-setup) | Installs the Salesforce CLI and related plugins
- [SFDX Login](https://github.com/svierk/sfdx-login) | Handles Salesforce login using a Salesforce DX authorization URL

Of course, the package installation action can be used flexibly and the respective approach can vary.

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-package-installation/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-package-installation/blob/main/LICENSE).