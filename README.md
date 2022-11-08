# GitHub Action for dotnet-format

[![CI Workflow Status](https://github.com/eero-dev/dotnet-format/workflows/CI/badge.svg)](https://github.com/eero-dev/dotnet-format/actions?query=workflow%3ACI)

Run [dotnet-format](https://github.com/dotnet/format) as part of your workflow to report formatting errors or auto fix violations as part of your pull request workflow.

## Usage

### Running on `pull_request`, adding a commit to the PR.
Use this only when the PRs are coming from the same repository. You won't have permission to add commits to PRs coming from a forked repository.

```yml
nname: Format check on pull request
on: pull_request
jobs:
  dotnet-format:
    runs-on: windows-latest
    steps:
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '6.0.x'

      - name: Checkout repo
        uses: actions/checkout@v3
        with:
          ref: ${{ github.head_ref }}

      - name: Run dotnet format
        id: format
        uses: eero-dev/dotnet-format@d3f2aef1dee3a6e6ed48183722e118ec24635a67
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          action: "fix"
          workspace: "\"Project Collection.sln\""
          only-changed-files: true

      - name: Commit files
        if: steps.format.outputs.has-changes == 'true'
        run: |
          git config --local user.name "github-actions[bot]"
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git commit -a -m 'Automated dotnet-format update
          Co-authored-by: ${{ github.event.pull_request.user.login }} <${{ github.event.pull_request.user.id }}+${{ github.event.pull_request.user.login }}@users.noreply.github.com>'
      - name: Push changes
        if: steps.format.outputs.has-changes == 'true'
        uses: ad-m/github-push-action@552c074ed701137ebd2bf098e70c394ca293e87f
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.head_ref }}

```
## Options

### Required

Name | Allowed values | Description
-- | -- | --
`repo-token` | `GITHUB_TOKEN` or a custom value | The token used to call the GitHub api.

### Optional

Name | Allowed values | Description
-- | -- | --
`action` | `check` (default), `fix` | The primary action `dotnet format` should perform. Check doesn't make changes to your file, fix will do the actual formatting.
`only-changed-files` | `true`, `false` (default) | Only changed files in the current pull request should be formatted. Only works when the trigger is a pull request.
`fail-fast` | `true` (default), `false` | The job should fail if there's a formatting error. Only used with the `check` action.
`workspace` | `.` | The solution or project file to operate on. In case you want to process all files in a certain folder, set the root folder here and specify the `workspaceIsFolder` option.
`include` | `.` | The files to include, delimited by space. Cannot be used together with the `workspace` option.
`exclude` | `.` | Space delimited list of files and/or folders to ignore.
`log-level` | `q[uiet]`, `m[inimal]`, `n[ormal]`, `d[etailed]`,  `diag[nostic]` | Sets the logging verbosity of the dotnet format process
`fix-whitespace` | `true`, `false` (default) | Removes whitespaces according to formatting rules.
`fix-analyzers-level` | `info`, `warn`, `error` | Fixes styles from third-party analyzers. More on https://github.com/dotnet/format/blob/main/docs/3rd-party-analyzers.md.
`fix-style-level` | `info`, `warn`, `error` | Fixes styles according to formating rules.

## Outputs

Name | Description
-- | --
`has-changes` | If any files were found to have violations or had fixes applied. Will be a string value of `true` or `false`.

# Building from source

## Generating `dist/index.js`
`npm run release`

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

## Acknowledgements

Thank you @victor-alcazar, @xt0rted and @jfversluis for the first versions that I have forked this from
