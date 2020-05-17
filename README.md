# Team Rotation

A GitHub Action to determine the next team member in a rotation.

## Use

This GitHub Action is meant to be used to assist with creating issues or PRs for tasks that rotate through a team. For example:

```yaml
# .github/workflows/weekly-issue
on:
  schedule:
    - cron: 0 20 * * 2
name: Create weekly issue
jobs:
  stuff:
    steps:
      - uses: lee-dohm/last-assigned@v1
        with:
          query: 'label:weekly-issue'
          token: ${{ secrets.GITHUB_TOKEN }}
        id: assigned
      - uses: lee-dohm/team-rotation@v1
        with:
          last: ${{ steps.assigned.outputs.last }}
          include: octocat lee-dohm
        id: rotation
      - uses: JasonEtco/create-an-issue@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/weekly-issue-template.md
          assignees: ${{ steps.rotation.outputs.next }}
```

This workflow:

1. Gets the person that was assigned to the last issue opened that matches the query `label:weekly-issue`
1. Determines the next person in the team rotation after the person found in step 1
1. Creates a new issue based on the `.github/weekly-issue-template.md` template and assigns the person found in step 2

### Inputs

- `exclude` -- Space-separated list of GitHub handles to remove from the list
- `include` -- Space-separated list of GitHub handles to add to the list generated by `teamName`, if any
- `last` -- GitHub handle of the last person to be assigned from the rotation
- `teamName` -- The name of a GitHub team, ex: `@org-name/team-name`. If this parameter is specified, `token` is required.
- `token` -- Token to use to perform the query of members in `teamName`. **Please note:** Because the default `GITHUB_TOKEN` doesn't have access to query organization team members, you'll have to [create a personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) with access to organization information and [create a secret](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets#creating-encrypted-secrets) for it

### Outputs

- `next` -- GitHub handle of the next person in the rotation

### Logic

The next person in the rotation is determined by:

1. Getting the list of handles associated with `teamName`, if any
1. Adding any handles from `include`
1. Removing any handles in `exclude`
1. Removing any duplicates in the list
1. Sorting the list alphabetically
1. Finding the handle specified in `last` in the list
1. Taking the next handle in the list, if there are no more in the list, use the first handle in the list

## License

[MIT](LICENSE.md)
