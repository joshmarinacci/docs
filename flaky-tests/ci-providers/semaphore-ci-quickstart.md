---
description: Configure Flaky Tests using Semaphore CI
---

# Semaphore CI

## Getting Started

You can use the Flaky Tests CLI within your [Semaphore CI](https://semaphoreci.com/) workflows to upload your test results.

{% hint style="info" %}
The Trunk Flaky Tests CLI currently only supports x86\_64 and arm64 for both Linux and macOS. If you have another use case, please get in touch with support at [https://slack.trunk.io](https://slack.trunk.io/). For the best results, you'll need to validate that your test invocation doesn't use cached test results and doesn't automatically retry failing tests.
{% endhint %}

1. Create a Semaphore workflow that runs the tests you want to monitor. In order for us to use the results, these tests **must** produce a test report in [JUnit XML](https://github.com/testmoapp/junitxml) format.

## Find Organization Slug and Token

Next you will need your Trunk **organization slug** and **token.** Navigate to [app.trunk.io](http://app.trunk.io). Once logged in navigate to **Settings** -> **Manage** -> **Organization**. Copy your organization slug. You can find your Trunk token by navigating to **Settings** → **Manage Organization** → **Organization API Token** and clicking "View." Copy this token.

In your Semaphore dashboard, store your Trunk token in a secret named TRUNK\_TOKEN. Update your Semaphore workflow to download and run the test uploader binary after you've run your tests.

{% @supademo/embed demoId="clvmr1w3d19ac769dnukc5ywg" url="https://app.supademo.com/demo/clvmr1w3d19ac769dnukc5ywg" %}

## Sample Semaphore workflow steps

You can upload test results to Flaky Tests with the [`trunk-analytics-cli`](https://github.com/trunk-io/analytics-cli) by running it in a stage after your tests are complete. There are four different OS/arch builds of the CLI in the latest release. Pick the one you need for your testing platform and be sure to download the release on every CI run. **Do not bake the CLI into a container or VM.** This ensures your CI runs are always using the latest build.

Portion of `repo/.semaphore/semaphore.yml`. See the complete file [here](https://github.com/mmatheson/SemaphoreFlakyTestExample/blob/main/.semaphore/semaphore.yml).

{% tabs %}
{% tab title="Linux x86_64" %}
{% code title="semaphore.yaml" overflow="wrap" lineNumbers="true" %}
```yaml
version: v1.0
name: Semaphore JavaScript Example Pipeline
blocks:
  - name: Tests
    task:
      secrets:
        - name: Trunk API Token
      env_vars:
        - name: NODE_ENV
          value: test
        - name: CI
          value: "true"
      prologue:
        commands:
          - checkout
          - nvm use
          - node --version
          - npm --version
      jobs:
        - name: Run Tests
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json),node-modules-$SEMAPHORE_GIT_BRANCH,node-modules-master
            - npm test
      epilogue:
        always:
          commands:
            # Publish results to Semaphore
            - test-results publish junit.xml
            # Upload results to trunk.io
            - curl -fsSL --retry 3 "https://github.com/trunk-io/analytics-cli/releases/latest/download/trunk-analytics-cli-x86_64-unknown-linux.tar.gz" | tar -xvz > ./trunk-analytics-cli
            - ./trunk-analytics-cli upload --junit-paths "junit.xml" --org-url-slug <TRUNK_ORG_SLUG> --token "${TRUNK_API_TOKEN}"

after_pipeline:
  task:
    jobs:
      - name: Publish Results
        commands:
          - test-results gen-pipeline-report
```
{% endcode %}
{% endtab %}

{% tab title="Linux arm64" %}
{% code title="semaphore.yaml" overflow="wrap" lineNumbers="true" %}
```yaml
version: v1.0
name: Semaphore JavaScript Example Pipeline
blocks:
  - name: Tests
    task:
      secrets:
        - name: Trunk API Token
      env_vars:
        - name: NODE_ENV
          value: test
        - name: CI
          value: "true"
      prologue:
        commands:
          - checkout
          - nvm use
          - node --version
          - npm --version
      jobs:
        - name: Run Tests
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json),node-modules-$SEMAPHORE_GIT_BRANCH,node-modules-master
            - npm test
      epilogue:
        always:
          commands:
            # Publish results to Semaphore
            - test-results publish junit.xml
            # Upload results to trunk.io
            - curl -fsSL --retry 3 "https://github.com/trunk-io/analytics-cli/releases/latest/download/trunk-analytics-cli-aarch64-unknown-linux.tar.gz" | tar -xvz > ./trunk-analytics-cli
            - ./trunk-analytics-cli upload --junit-paths "junit.xml" --org-url-slug <TRUNK_ORG_SLUG> --token "${TRUNK_API_TOKEN}"

after_pipeline:
  task:
    jobs:
      - name: Publish Results
        commands:
          - test-results gen-pipeline-report
```
{% endcode %}
{% endtab %}

{% tab title="macOS x86_64" %}
{% code title="semaphore.yaml" overflow="wrap" lineNumbers="true" %}
```yaml
version: v1.0
name: Semaphore JavaScript Example Pipeline
blocks:
  - name: Tests
    task:
      secrets:
        - name: Trunk API Token
      env_vars:
        - name: NODE_ENV
          value: test
        - name: CI
          value: "true"
      prologue:
        commands:
          - checkout
          - nvm use
          - node --version
          - npm --version
      jobs:
        - name: Run Tests
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json),node-modules-$SEMAPHORE_GIT_BRANCH,node-modules-master
            - npm test
      epilogue:
        always:
          commands:
            # Publish results to Semaphore
            - test-results publish junit.xml
            # Upload results to trunk.io
            - curl -fsSL --retry 3 "https://github.com/trunk-io/analytics-cli/releases/latest/download/trunk-analytics-cli-x86_64-apple-darwin.tar.gz" | tar -xvz > ./trunk-analytics-cli
            - ./trunk-analytics-cli upload --junit-paths "junit.xml" --org-url-slug <TRUNK_ORG_SLUG> --token "${TRUNK_API_TOKEN}"

after_pipeline:
  task:
    jobs:
      - name: Publish Results
        commands:
          - test-results gen-pipeline-report
```
{% endcode %}
{% endtab %}

{% tab title="macOS arm64" %}
{% code title="semaphore.yaml" overflow="wrap" lineNumbers="true" %}
```yaml
version: v1.0
name: Semaphore JavaScript Example Pipeline
blocks:
  - name: Tests
    task:
      secrets:
        - name: Trunk API Token
      env_vars:
        - name: NODE_ENV
          value: test
        - name: CI
          value: "true"
      prologue:
        commands:
          - checkout
          - nvm use
          - node --version
          - npm --version
      jobs:
        - name: Run Tests
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json),node-modules-$SEMAPHORE_GIT_BRANCH,node-modules-master
            - npm test
      epilogue:
        always:
          commands:
            # Publish results to Semaphore
            - test-results publish junit.xml
            # Upload results to trunk.io
            - curl -fsSL --retry 3 "https://github.com/trunk-io/analytics-cli/releases/latest/download/trunk-analytics-cli-aarch64-apple-darwin.tar.gz" | tar -xvz > ./trunk-analytics-cli
            - ./trunk-analytics-cli upload --junit-paths "junit.xml" --org-url-slug <TRUNK_ORG_SLUG> --token "${TRUNK_API_TOKEN}"

after_pipeline:
  task:
    jobs:
      - name: Publish Results
        commands:
          - test-results gen-pipeline-report
```
{% endcode %}
{% endtab %}
{% endtabs %}

The workflow above configures the cache and then runs `npm test` to actually generate the test output XML. The epilogue of the test block uses Semaphore's `test-results` command to publish the `junit.xml` file to Semaphore. Then it uses the curl command to download the latest version of the `trunk-analytics-cli`, make it executable, and finally run the uploader to send the `junit.xml` to Trunk.

The `trunk-analytics-cli` tool has several important arguments.

* `--junit-paths`is a comma separated list of paths.
* `--org-url-slug` is an identifier for the Trunk account you are using. This is the Organization Slug you copied from your Trunk settings above.
* `--token` is the Trunk API token you added as a Semaphore secret above.

## Hourly Tests

Running on a golden branch is how we detect that tests are flaky. Create a copy of your `semaphore.yml` file called `hourly.yml` . In your Semaphore dashboard, create a new task to run tests on the main branch, and link to the `hourly.yml` file. Schedule it to run **every hour, every day**. Click _Create_ then manually run the workflow to check that it's working. Now you can look at the data being collected in Trunk's [Flaky Tests dashboard](https://app.trunk.io/).

The source for the settings above are available in our [SemaphoreFlakyTestExample](https://github.com/mmatheson/SemaphoreFlakyTestExample) repo.

***

If you're interested in better understanding this binary or want to contribute to it, you can find the open source repo [here](https://github.com/trunk-io/analytics-cli).
