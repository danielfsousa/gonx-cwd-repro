# gonx cwd repro

Minimal reproduction for `@naxodev/gonx` 4.1.1: every executor fails with `spawnSync go ENOENT`
when Nx runs from a project directory, although `go` is on `PATH`.

The workspace is `nx` 23.2.1, `@naxodev/gonx` 4.1.1 and one Go application, `apps/my-go-app`.

## Steps

```bash
npm ci

npx nx build apps/my-go-app --skip-nx-cache   # passes

cd apps/my-go-app
npx nx build --skip-nx-cache                  # fails: spawnSync go ENOENT
```

`--skip-nx-cache` matters: without it, the second build replays the first one's cached result.

The `repro` workflow runs both builds on `ubuntu-latest`; its last step fails.

## Cause

The executors pass the workspace-relative project root (`apps/my-go-app`) to `execFileSync` as
`cwd`, which Node resolves against the process's current directory. From `apps/my-go-app` that is
`apps/my-go-app/apps/my-go-app`, which does not exist, and Node reports a missing `cwd` as `ENOENT`
on the executable.
