# Sambafied fork notes

This fork is an optional Windows backend candidate for
[Sambafied #194](https://github.com/sambafied/sambafied/issues/194).
WinFsp remains the primary virtual-volume backend; native SMB mapping is a
separate supported mode. No Dokan driver is installed by this fork change.

Baseline: upstream wrapper `ce2ec8565591d21ed5b58f8233e9d81a730823ad`,
crates `0.3.1+dokan230`, bundled Dokany submodule
`32ae6fa98beaf41b4a8e607dc11c4024f02b0ba3` (Dokan 2.3.0).
Pin the wrapper, native import library, DLL and installed driver together.
The build script currently recognizes x86 and x64, not ARM64.

The fork removes the build script's unconditional process/compiler environment
and compiler-argument dumps. Environment values can contain credentials and
must not be included in ordinary Cargo diagnostics. Native compiler error
output remains available.

Local validation on Windows x64 with Rust 1.98.1 (2026-10-11):

- `cargo build --locked --workspace --all-targets`: passed, including the
  bundled user-mode DLL and memfs example. Existing `extern "stdcall"` declarations
  produce future-incompatibility warnings on x64; address this separately before
  production adoption. The kernel driver was not built or installed.
- Four exact, non-mounting wrapper tests passed:
  `can_map_win32_error_to_ntstatus`,
  `data::find_data::tests::test_wrap_fill_data`,
  `test_map_kernel_to_user_create_file_flags`, `test_is_name_in_expression`.
- An earlier filtered run also selected `file_system::can_fail_to_mount`:
  four tests passed and that test failed with `DriverInstall` instead of `Mount`
  because the matching Dokan driver is absent. This is retained as a missing
  native prerequisite, not a passing mount result. Do not run the upstream
  mounting suite indiscriminately: it uses Z:, occupied on this PC.
- `cargo fmt --all -- --check` and `git diff --check`: passed.

Mount lifecycle, write protection, delete error returns, file sharing, account
isolation, Explorer integration and recovery are not qualified by a successful
wrapper build. `MountFlags::WRITE_PROTECT` is exposed, but requires actual Windows
acceptance tests in Sambafied's owned fixture.
