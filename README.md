# sccache-action (Forgejo fork)

This is a Forgejo-optimised fork of Mozilla's
[sccache-action](https://github.com/mozilla-actions/sccache-action) that eliminates GitHub API
dependencies and rate limiting issues. It's designed specifically for Forgejo environments and
won't work on GitHub - use the original Mozilla action for GitHub workflows.

The action integrates [sccache](https://github.com/mozilla/sccache/) into your Forgejo Actions
workflows to cache compilation results, significantly reducing build times across subsequent runs.

## How it works

Rather than downloading binaries directly from GitHub (which requires authentication and hits rate
limits), this fork:

1. **Mirrors sccache releases** automatically using a Forgejo workflow that checks for updates and
   stores binaries in your instance's package registry
2. **Detects versions dynamically** by querying your Forgejo instance's package API to find the
   latest available version
3. **Downloads locally** from your own package registry, avoiding external dependencies entirely

The action automatically detects which Forgejo instance it's running on and uses the appropriate
package registry - no configuration needed.

## Usage

Add this step to your Forgejo Actions workflow:

### Automatic version detection (recommended)

The action automatically finds the latest sccache version available in your package registry:

```yaml
- name: Setup sccache
  uses: your-forgejo-instance/owner/sccache-action@v1
```

### Specify a particular version

If you need a specific version:

```yaml
- name: Setup sccache
  uses: your-forgejo-instance/owner/sccache-action@v1
  with:
    version: "v0.10.0"
```

### Custom registry configuration

To use a different Forgejo instance or repository for sccache packages:

```yaml
- name: Setup sccache
  uses: your-forgejo-instance/owner/sccache-action@v1
  with:
    server: "https://your-forgejo.example.com"
    owner: "your-owner"
```

**For action maintainers**: You can change the defaults by editing the `action.yml` file and
updating the `server` and `owner` default values. This allows you to fork the
action and point it to your own package registry without requiring users to specify these inputs.

### Conditional caching

Only enable caching for certain workflow types:

```yaml
- name: Setup sccache for development builds
  if: github.event_name != 'release'
  uses: your-forgejo-instance/owner/sccache-action@v1

- name: Configure Rust to use sccache
  if: github.event_name != 'release'
  run: |
    echo "SCCACHE_GHA_ENABLED=true" >> $GITHUB_ENV
    echo "RUSTC_WRAPPER=sccache" >> $GITHUB_ENV
```

### View compilation statistics

The action automatically creates a post-run step to show stats. You can also check stats manually:

```yaml
- name: Show sccache statistics
  run: ${SCCACHE_PATH} --show-stats
```

### Disable automatic stats reporting

```yaml
- name: Setup sccache
  uses: your-forgejo-instance/owner/sccache-action@v1
  with:
    disable_annotations: true
```

## Language-specific configuration

### Rust projects

Enable sccache for Rust compilation:

```yaml
env:
  SCCACHE_GHA_ENABLED: "true"
  RUSTC_WRAPPER: "sccache"
```

### C/C++ projects

For C/C++ projects, enable the cache and configure your build system:

```yaml
env:
  SCCACHE_GHA_ENABLED: "true"
```

**With CMake:**

```bash
cmake -DCMAKE_C_COMPILER_LAUNCHER=sccache -DCMAKE_CXX_COMPILER_LAUNCHER=sccache
```

**With configure scripts:**

```bash
# Using GCC
./configure CC="sccache gcc" CXX="sccache g++"
# Using Clang
./configure CC="sccache clang" CXX="sccache clang++"
```

## Setting up the mirroring workflow

To use this action, you'll need the mirroring workflow that downloads sccache releases and stores
them in your package registry. The workflow is included in this repository at
`.forgejo/workflows/mirror-sccache.yml`.

1. Fork this repository to your Forgejo instance
2. The mirroring workflow runs automatically daily and whenever you push changes
3. It checks for new sccache releases and mirrors missing versions to your package registry
4. Your workflows can then use the action with automatic version detection

The mirroring workflow requires a `FORGEJO_TOKEN` secret with package write permissions.

## License

Apache-2.0 (just like sccache)
