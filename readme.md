# Terraform CDK a.k.a. cdktf - Github actions

The actions are based on Terraform directly, using cdktf only to synthesize the Terraform code.
These actions support multiple stacks generated by one cdktf project.

## Stages

The actions support different environments called stages.
The cdktf stacks should be named <stack-name>:<stage-name>

## Usage

### Comment all plans of multiple stacks of one environment on a PR

```yml
name: "Comment a Plan on a PR"

on: [pull_request]

permissions:
  contents: read
  pull-requests: write

jobs:
  terraform:
    name: "Terraform CDK Diff"
    runs-on: ubuntu-latest
    strategy:
      matrix:
        stage: [dev, stage, prod]

    steps:
    - uses: actions/checkout@v3
    - uses: pnpm/action-setup@v2  
    - uses: actions/setup-node@v3
      with:
        node-version: 16
        cache: "pnpm"
        registry-url: https://npm.pkg.github.com/
    
    - uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: eu-central-1

    - name: Synthesize Terraform code
      run: |
        pnpm install
        pnpm cdktf get
        pnpm build
        pnpm cdktf synth

    - uses: danielBreitlauch/cdktf-actions/plan@v1
      with:
        GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
        stage: ${{ matrix.stage }}
        working-directory: ./Infrastructure
```

### Apply all stacks of one environment after a PR is merged

```yml
name: "Deploy stacks after PR is Merged"

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  deploy:
    if: github.event.pull_request.merged
    runs-on: ubuntu-latest    
    strategy:
      matrix:
        stage: [dev, stage, prod]

    steps:
    - uses: actions/checkout@v3
    - uses: pnpm/action-setup@v2  
    - uses: actions/setup-node@v3
      with:
        node-version: 16
        cache: "pnpm"
        registry-url: https://npm.pkg.github.com/
    
    - uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: eu-central-1

    - name: Synthesize Terraform code
      run: |
        pnpm install
        pnpm cdktf get
        pnpm build
        pnpm cdktf synth

    - uses: danielBreitlauch/cdktf-actions/deploy@v1
      with:
        stage: ${{ matrix.stage }}
        working-directory: ./Infrastructure
```