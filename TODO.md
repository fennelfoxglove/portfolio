# To Do

## Tasks
- [X] Figure out git tag versioning
- [X] Overwrite old release with new one as soon as it becomes available
- [.] Add PDF download link to `_config.yml`
  - [.] What's the proper syntax for adding a yaml hyperlink?

## Notes
Git has two types of tags, lightweight and annotated. The former is like a
bookmark and the latter carries more information useful for making a release.

The more I learn about this, the less I may need the `generate tag` step in the
action. The question is, how can I access the tag in my md to pdf script? Right
now, it's dependent on the date. But how can I still push this workflow but have
it call on the local/remote tag instead?

https://stackoverflow.com/questions/58177786/get-the-current-pushed-tag-in-github-actions

So I gotta figure how to access `${{ github.ref }}` `${{ github.ref_name }}`

Answer:

```yaml
      # 3: Create release
      - name: push tag
        env:
          version: ${{ github.ref }}
        uses: actions/github-script@v9
        with:
          script: |
            github.rest.git.createRef({
              owner: context.repo.owner,
              repo: context.repo.repo,
              ref: `${version}`,
              sha: context.sha
            })
```

## git tags
How do git tags work with releases and creating refs with github actions

So what does the flow look like? 
1. Make changes on `dev`
2. Validate generated draft locally and on
3. Squash all commits
4. Merge `dev` into `main`
5. Push to `main`
6. Create tag `main` and push tag
7. Only build on tag push to `main`

So only build the next release when we push the tag to `main`

That issue is solved.

## updating refs
Next we need to figure how to get variable from one step to the other. Here's
the problem:

I have a `step` that generates a timestamp with: 

```yaml
run: | 
  echo "timestamp=$(date +%F)" >> "GITHUB_ENV"
```

Normally I can pull this into my next `step`:

```yaml
      # 4: Push tag
      - name: push tag
        env:
          timestamp: "string"
          VERSION: ${{ github.ref }}
          TIMESTAMP: ${{ github.env.timestamp }}
        uses: actions/github-script@v9
        with:
          script: |
            const version = process.env.VERSION
            const timestamp = process.env.TIMESTAMP
            github.rest.git.createRef({
              owner: context.repo.owner,
              repo: context.repo.repo,
              ref: `${version}-${timestamp}`,
              sha: context.sha
            })
```

But the problem I'm running into is that `env` will overwrite the `GITHUB_ENV`
variable without importing from the previous `step`.

Is there a way to import the `$GITHUB_ENV` from a previous `step`?
Alternatively, is there a way to `run` a shell command from within the env?

Deemed this unnecessary. 

## Drafts no longer actually release
Is the draft deal deleting all the drafts before they publish?
Why can't I see them? Why did this break?

Github was having issues.

Now I can get back to actually debugging this.

## How to pass iformation between job steps?
https://stackoverflow.com/questions/57819539/how-to-share-a-calculated-value-between-job-steps

```yaml
# Example 1 - Asker example

name: Test, Build and Deploy
on:
  push:
    branches:
      - master
jobs:
  build_and_push:
    name: Build and Push
    runs-on: ubuntu-latest
    steps:
      - name: Set tag var
        id: vars
        run: echo "docker_tag=$(echo ${GITHUB_REF} | cut -d'/' -f3)-${GITHUB_SHA}" >> $GITHUB_OUTPUT
      - name: Docker Build
        uses: "actions/docker/cli@master"
        with:
          args: build . --file Dockerfile -t cflynnus/blog:${{ steps.vars.outputs.docker_tag }}
      - name: Docker Tag Latest
        uses: "actions/docker/cli@master"
        with:
          args: tag cflynnus/blog:${{ steps.vars.outputs.docker_tag }} cflynnus/blog:latest

# Example 2 - Multiple vars

      - name: Set output variables
        id: vars
        run: |
          pr_title="[Test] Add report file $(date +%d-%m-%Y)"
          pr_body="This PR was auto-generated on $(date +%d-%m-%Y) \
            by [create-pull-request](https://github.com/peter-evans/create-pull-request)."
          echo "pr_title=$pr_title" >> $GITHUB_OUTPUT
          echo "pr_body=$pr_body" >> $GITHUB_OUTPUT
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v4
        with:
          title: ${{ steps.vars.outputs.pr_title }}
          body: ${{ steps.vars.outputs.pr_body }}

# Example 3 - Environment variables

      - name: Set environment variables
        run: |
          echo "PR_TITLE=[Test] Add report file $(date +%d-%m-%Y)" >> $GITHUB_ENV
          echo "PR_BODY=This PR was auto-generated on $(date +%d-%m-%Y) by [create-pull-request](https://github.com/peter-evans/create-pull-request)." >> $GITHUB_ENV
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v4
        with:
          title: ${{ env.PR_TITLE }}
          body: ${{ env.PR_BODY }}
```
