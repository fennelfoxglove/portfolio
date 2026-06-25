# To Do

## Tasks
- [.] Figure out git tag versioning
- [.] Overwrite old release with new one as soon as it becomes available
- [.] Add PDF download link to `_config.yml`

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
