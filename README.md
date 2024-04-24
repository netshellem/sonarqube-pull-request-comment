# sonarqube-pull-request-comment

This GitHub Action adds a comment to your pull requests to quickly visualize the results of the Quality Gate.

## Requirements

* Defined the secrets **SONAR_HOST_URL** and **SONAR_TOKEN**.
* A previous step must have run an analysis on your code **with the "sonar.qualitygate.wait" option to true in the sonar-project.properties file or in the command options of the sonar-scanner**.

## Secrets

You can set the **SONAR_HOST_URL** and **SONAR_TOKEN** environment variable in the **Secrets** settings page of your repository, or you can add them at the level of your GitHub organization (recommended).

* **SONAR_HOST_URL** – Required this tells the scanner where SonarQube is hosted.
* **SONAR_TOKEN** – Required this is the token used to authenticate access to SonarQube. You can read more about security tokens [here](https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/).

## Usage

The workflow YAML file will usually look something like this

```yaml
on:
  # Trigger analysis when pushing in master or pull requests, and when creating
  # a pull request. 
  push:
    branches:
      - master
  pull_request:
      types: [opened, synchronize, reopened]

# Global permission here all other scopes will have access. 
# OR use Job permission, that will only apply to the job the permission is set on.
permissions:
  pull-requests: write

name: Main Workflow
jobs:
  sonarqube:
    runs-on: ubuntu-latest

    # Job permission, that will only apply to the job named "sonarqube". 
    # Write access is granted for the pull-requests scopes. All other scopes will have no access.
    permissions:
      pull-requests: write

    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0

    # Triggering SonarQube analysis as results of it are required by Quality Gate check.
    # ----------------------------------------------------------------------------------
    # With sonarqube-scan-action
    - name: SonarQube Scan
      uses: sonarsource/sonarqube-scan-action@master
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
    # ----------------------------------------------------------------------------------
    # Or if you have installed the Sonar Scanner command in a previous step.
    - name: SonarQube Scan
      run: sonar-scanner -Dsonar.login=${{ secrets.SONAR_TOKEN }} -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} -Dsonar.qualitygate.wait=true 
    # ----------------------------------------------------------------------------------

    - name: SonarQube Comment
      if: always() && github.event.action == 'pull_request'
      uses: ibex-code/sonarqube-pull-request-comment@v1
      with:
        sonar_token: ${{ secrets.SONAR_TOKEN }}
        sonar_host_url: ${{ secrets.SONAR_HOST_URL }}
        github_token: ${{ secrets.GITHUB_TOKEN }} 
```

## Generated comments

### On success

![success](https://github.com/ibex-code/sonarqube-pull-request-comment/blob/v1/images/success.webp?raw=true)

### On error

![error](https://github.com/ibex-code/sonarqube-pull-request-comment/blob/v1/images/error.webp?raw=true)

## License

This project are released under the LGPLv3 License, see LICENSE.txt.
