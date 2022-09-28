# Stratis CI Container Action

This composite action executes a task in a newly built container
allowing a workflow to use a mix of steps that execute on the host or in
a container.

This makes it possible to disable the `systemd-udevd` daemon on the host
system and to run an instance local to the container for tests that
depend on udev functionality.

The container environment will include the specified Rust toolchain
version and components and has the GitHub workspace mounted at `/work`.
## Inputs

### `image`

**Required** The name of the container image to use.

### `components`

**Required** The list of toolchain components to use.

### `toolchain`

**Required** The version of the rust toolchain to use.

### `task`

**Required** The task to run in the container.

### `deps`

**Required** A command to install required dependencies in the container,
run before the container task.

## Examples

#### Typical Use Case

```yaml
  # TESTS WITH UDEV
  checks_with_udev:
    strategy:
      matrix:
        include:
          - task: RUST_LOG=stratisd=debug make -f Makefile test-loop
            toolchain: 1.64.0  # CURRENT DEVELOPMENT RUST TOOLCHAIN
            components: cargo
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v3
      - name: Stop host udevd
        run: |
          sudo systemctl stop systemd-udevd
          sudo systemctl disable systemd-udevd-kernel.socket
          sudo systemctl disable systemd-udevd-control.socket
      - uses: bmr-cymru/stratis-ci-container-action@v1
        with:
          image: fedora:36
          components: ${{ matrix.components }}
          toolchain: ${{ matrix.toolchain }}
          task: ${{ matrix.task }}
          deps: >
            dnf install -y
            asciidoc
            clang
            cryptsetup-devel
            curl
            device-mapper-persistent-data
            device-mapper-devel
            dbus-devel
            libblkid-devel
            make
            systemd-devel
            systemd-udev
            xfsprogs
```
