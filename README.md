# Github Action for mapping variables by a specific key

![build-test](https://github.com/kanga333/variable-mapper/workflows/build-test/badge.svg)

Variable-Mapper action maps variables by regular expressions.

- The map argument is a configuration in json format.
  - The top-level key in JSON is a regular expression condition. They are evaluated in order from the top.
  - The value is the key-value pair of variables to be exported.
- The key argument is the key to match the map.

## Sample Workflows

### Export variables corresponding to regular expression-matched keys

```yaml
on: [push]
name: Export variables corresponding to regular expression-matched keys
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Extract github branch name
      run: echo "##[set-output name=branch;]$(echo ${GITHUB_REF#refs/heads/})"
      id: get_branch
    - uses: actions/checkout@v2
    - uses: chris-madsen/variable-mapper@v0.1.3
      with:
        key: ${{ steps.get_branch.outputs.branch }}
        map: |
          {
            "development": {
              "environment": "development",
              "AWS_ACCESS_KEY_ID": "${{ secrets.AWS_ACCESS_KEY_ID_DEV }}",
              "AWS_SECRET_ACCESS_KEY": "${{ secrets.AWS_SECRET_ACCESS_KEY_DEV }}"
            },
            "qa": {
              "environment": "qa",
              "AWS_ACCESS_KEY_ID": "${{ secrets.AWS_ACCESS_KEY_ID_QA }}",
              "AWS_SECRET_ACCESS_KEY": "${{ secrets.AWS_SECRET_ACCESS_KEY_QA }}"
            },
            "master": {
              "environment": "production",
              "AWS_ACCESS_KEY_ID": "${{ secrets.AWS_ACCESS_KEY_ID_PROD }}",
              "AWS_SECRET_ACCESS_KEY": "${{ secrets.AWS_SECRET_ACCESS_KEY_PROD }}"
            }
          }
          
    - name: Echo environment
      run: echo ${{ env.environment }}
```

The key is evaluated from the top and exports the first matched variables.

### Export variables to output and environment and log

```yaml
on: [push]
name: Export variables to output and environment and log
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Extract github branch name
      run: echo "##[set-output name=branch;]$(echo ${GITHUB_REF#refs/heads/})"
      id: get_branch
    - uses: actions/checkout@v2
    - uses: chris-madsen/variable-mapper@v0.1.3
      id: export
      with:
        key: ${{ steps.get_branch.outputs.branch }}
        map: |
          {
            "master": {
              "environment": "production"
            },
            ".*": {
              "environment": "development"
            }
          }
        export_to: env,log,output
        
    - name: Echo environment and output
      run: |
        echo ${{ env.environment }}
        echo ${{ steps.export.outputs.environment }}
```

The variables can be exported to log, env and output. (Default is `log,env`)
