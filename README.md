# Docker GH action test
A small test/POC for the `docker` daemon in GitHub actions.

See [test.yaml](.github/workflows/test.yml)
```yaml
jobs:
  info:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.17
      - name: Updating and upgrading brew
        if: matrix.os == 'macos-latest'
        run: |
          ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
          brew --version
      - name: Install and start docker
        if: matrix.os == 'macos-latest'
        # https://github.com/docker/for-mac/issues/2359#issuecomment-943131345
        run: |
          brew install --cask docker
          sudo /Applications/Docker.app/Contents/MacOS/Docker --unattended --install-privileged-components
          open -a /Applications/Docker.app --args --unattended --accept-license
          while ! /Applications/Docker.app/Contents/Resources/bin/docker info &>/dev/null; do sleep 1; done
      - name: Test docker
        run: |
          docker version
          docker info
```

## Why
- I wanted to run `docker` in `windows`, `macos` and `linux` in GitHub actions to have a minimal POC.
- Currently it's working for `docker` in version `v20.10.8` for `macos` with the code from this [comment](https://github.com/docker/for-mac/issues/2359#issuecomment-943131345).
- You may wonder why there are still scripts in the `ci-scripts` folder. The reason is: I wanted to keep them if the `Docker.app` changes its install flags, etc. in the future.
