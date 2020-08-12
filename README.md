# Nightfall DLP Action
![nightfalldlp](https://nightfall.ai/wp-content/uploads/2020/08/nightfall-dark-logo-tm-e1597263930794.png)
## [Nightfall](https://nightfall.ai) DLP Action: A code review tool that protects you from committing sensitive information to your repositories.

The Nightfall DLP Action scans your code commits upon Pull Request for sensitive information - like credentials & secrets, PII, credit card numbers & more - and posts review comments to your code hosting service automatically. The Nightfall DLP Action is intended to be used as a part of your CI to simplify the development process, improve your security, and ensure you never accidentally leak secrets or other sensitive information via an accidental commit.

## Example
Here's an example using the Nightfall DLP Github Action inside a Github Workflow:  
_**Note:** you must use the actions/checkout step as shown below before the running the nightfalldlp action in order for it to function properly_
```yaml
name: nightfalldlp
on:
  push:
    branches:
      - master
  pull_request:
jobs:
  run-nightfalldlp:
    name: nightfalldlp
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo Action
        uses: actions/checkout@v2

      - name: nightfallDLP action step
        uses: nightfallai/nightfall_dlp_action@CreateAction
        env:
          NIGHTFALL_API_KEY: ${{ secrets.NIGHTFALL_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          EVENT_BEFORE: ${{ github.event.before }}
```

## Usage
**1. Get a Nightfall API key.**

The Nightfall DLP Action is powered by the Nightfall DLP API. Learn more and request access to a free API key **[here](https://nightfall.ai/api/)**. Alternatively, you can email **[sales@nightfall.ai](mailto:sales@nightfall.ai)** to provision a free account.

**2. Set up config file to specify detectors.**

- Place a `.nightfalldlp/` directory within the root of your target repository, and inside it a `config.json` file in which you can configure your detectors (see `Detectors` section below for more information on Detectors)
- See `Additional Configuration` section for more advanced configuration options
- Below is a sample `.nightfalldlp/config.json` file:

```json
{
  "detectors": [
    "CREDIT_CARD_NUMBER",
    "PHONE_NUMBER",
    "API_KEY",
    "CRYPTOGRAPHIC_KEY",
    "RANDOMLY_GENERATED_TOKEN",
    "US_SOCIAL_SECURITY_NUMBER",
    "AMERICAN_BANKERS_CUSIP_ID",
    "US_BANK_ROUTING_MICR",
    "ICD9_CODE",
    "ICD10_CODE",
    "US_DRIVERS_LICENSE_NUMBER",
    "US_PASSPORT",
    "EMAIL_ADDRESS",
    "IP_ADDRESS"
  ]
}
```
**3. Set up a few environment variables.**     
These variables should be made available to the nightfall_dlp_action by adding them to the `env:` key in your workflow:

- `NIGHTFALL_API_KEY`
    - Get a free Nightfall DLP API Key by registering for an account with the [Nightfall API](https://nightfall.ai/api)
    - Add this variable to your target repository's "Github Secrets" and passed in to your Github Workflow's `env`.

- `GITHUB_TOKEN`
    - This is automatically injected by Github inside each Workflow (via the `secrets` context), you just need to set it to the `env` key. This variable should always point to `secrets.GITHUB_TOKEN`
    - This token is used to authenticate to Github to write Comments/Annotations to your Pull Requests and Pushes

- `EVENT_BEFORE` (*only required for Github Workflows running on a `push` event)
    - the value for this var lives on the `github` context object in a Workflow - EVENT_BEFORE should always point to `${{ github.event.before }}` (as seen in the example above)
    
## Supported GitHub Events
The Nightfall DLP Action can run in a Github Workflow triggered by the following events:

- `PULL_REQUEST`
- `PUSH`

The Nightfall DLP Action is currently unable to be used in forked GitHub repositories due to GitHub's disabling of secrets sharing when Workflows originate from forks.

## Detectors
Each detector represents a type of information you want to search for in your code scans. A few examples of detectors Nightfall supports includes:

- `API_KEY`: A freeform string used for user verification to access online program functions.
- `CREDIT_CARD_NUMBER`: A 12 to 19 digit number used for payments and other monetary transactions.
- `CRYPTOGRAPHIC_KEY`: A string of characters used by an encryption algorithm to generate seemingly random tokens.
- `RANDOMLY_GENERATED_TOKEN`: A pseudo-random string generated by an encryption algorithm. This detector is more general than the API_KEY detector.
- `US_SOCIAL_SECURITY_NUMBER`: A 9 digit numeric string often used as a unique identification number for United States citizens and residents.
- Many more.

**Find a full list of supported detectors in the Nightfall API Documentation, after creating your account (per the instructions above).**

## Additional Configuration
You can add additional fields to your config to ignore tokens and files as well as specify which files to exclusively scan on.

**Token Exclusion**

To ignore specific tokens, you can add the `tokenExclusionList` field to your config. The `tokenExclusionList` is a list of strings, and it mutes findings that match any of the given regex patterns.

Here's an example use case:

```tokenExclusionList: ["NF-gGpblN9cXW2ENcDBapUNaw3bPZMgcABs", "^127\\."]```

In the example above, findings with the API token `NF-gGpblN9cXW2ENcDBapUNaw3bPZMgcABs` as well as local IP addresses starting with `127.` would not be reported. For more information on how we match tokens, take a look at [regexp](https://golang.org/pkg/regexp/).

**File Inclusion/Exclusion**

To omit files from being scanned, you can add the `fileExclusionList` field to your config. In addition, to only scan on certain files, add the `fileInclusionList` to the config.

Here's an example use case:
```
    fileExclusionList: ["*/tests/*"],
    fileInclusionList: ["*.go", "*.json"]
```
In the example, we are ignoring all file paths with a `tests` subdirectory, and only scanning on `go` and `json` files.
Note: we are using [gobwas/glob](https://github.com/gobwas/glob) to match file path patterns. Unlike the token regex matching, file paths must be completely matched by the given pattern. e.g. If `tests` is a subdirectory, it will not be matched by `tests/*`, which is only a partial match.

## [Nightfall DLP API](https://nightfall.ai/api)
With the Nightfall API, you can inspect & classify your data, wherever it lives. Programmatically get structured results from Nightfall's deep learning-based detectors for things like credit card numbers, API keys, and more. Scan data easily in your own third-party apps, internal apps, and data silos. Leverage these classifications in your own workflows - for example, saving them to a data warehouse or pushing them to a SIEM. Request access & learn more **[here](https://nightfall.ai/api/)**.

## Versioning
The Nightfall DLP Action issues releases using semantic versioning.

## Support
For help, please email us at **[support@nightfall.ai](mailto:support@nightfall.ai)**.
