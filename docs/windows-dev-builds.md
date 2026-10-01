# Windows development packages

The `Snow Shot Windows Dev Build` workflow builds Windows x64 packages on GitHub
without installing the native build toolchain locally. It only runs through
`workflow_dispatch`; pushes and pull requests do not start a build.

## Run and download

1. Commit and push your changes to the fork.
2. Open **Actions > Snow Shot Windows Dev Build > Run workflow**.
3. Select the branch containing your changes and start the workflow.
4. After success, download `snow-shot-windows-dev-<run number>` under **Artifacts**.

The artifact contains the online and offline Windows installers, a portable ZIP,
their SHA-256 checksums, and package manifests. The offline installer and portable
ZIP include the default OCR resources. Packages are retained for 14 days. The
workflow does not create a release or build macOS, Linux, or Snow Image Viewer.

## Build environment and caches

The workflow selects VS 2026/MSVC 14.51, installs Rust 1.97.1, CMake 4.2.3 and NSIS
3.12.0, and checks the native tools and LLVM/libclang before compiling dependencies.
The packaging scripts still perform the existing static dependency, Qt, license
and runtime audits. They also generate release support files internally; only
installers and the portable package are uploaded as downloadable packages.

A first build can take several hours because vcpkg builds native dependencies,
including ONNX Runtime, and the workflow builds static Qt. The job allows up to
six hours. Changing the compiler, Windows SDK, dependency configuration, or Qt
build script can require rebuilding affected dependencies.

vcpkg uses its supported `files` binary-cache provider, with archives restored
and saved using `actions/cache`. vcpkg checks package ABI hashes before reusing
individual binaries. Completed dependencies are saved before Qt is built; partial
packages are also saved when dependency installation fails. Qt is cached using
the toolchain fingerprint and the installed PNG/zlib ABI receipts, and is saved
as soon as its build succeeds. An application or packaging failure therefore does
not discard successfully prepared dependency caches. Cache eviction can still
cause later cold builds.

Rust dependencies and build outputs are cached separately, including the updater
workspace. Cache entries are subject to GitHub's branch visibility rules; run on
the same branch when testing cache reuse. These caches do not provide live local
validation, and the application is still built and packaged on the hosted runner.
Before packaging, the workflow also fetches the Cargo packages needed by the
offline third-party license audit, including packages used only by development
dependencies.

## Failures

Inspect the failed step in the Actions run. When available, the workflow uploads
vcpkg and CMake diagnostic files as `snow-shot-windows-dev-logs-<run number>`,
retained for seven days. A missing package or mismatched checksum fails validation
before the downloadable artifact is uploaded.
