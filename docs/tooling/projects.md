# Projects and official tools

Arbor projects keep production code and tests separate. A unified command coordinates
the compiler, formatter, linter, test runner, and documentation tools.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

The `arbor/` directory is the umbrella repository root. Official tools live in
separate repositories included here as Git submodules. The design baseline is maintained
in the umbrella repository as the shared design reference.

Illustrative layout; submodule paths and remote repositories are not yet fixed:

```text
arbor/
    DESIGN.md
    tools/
        compiler/          # Submodule
        formatter/         # Submodule
        linter/            # Submodule
        lsp/               # Submodule
        documentation/     # Submodule
        package-manager/   # Submodule: Cargo-like build/package/test interface
        toolchain-manager/ # Submodule: rustup-like installation and selection
    editors/
        tree-sitter/       # Submodule
        vscode/            # Submodule
        vim/               # Submodule: Vim/Neovim integration
        emacs/             # Submodule
```

Runtime, standard-library, and shared compiler-library repository boundaries
remain open. The layout above documents intent; it does not register submodules
or invent their remotes.

Official commands include:

```text
arbor build
arbor run
arbor test
arbor fmt
arbor lint
arbor doc
```

Language intelligence should be shared through compiler libraries and the LSP.
Editor integrations should avoid implementing independent semantic rules.

An ordinary Arbor project follows a strict production/test layout:

```text
manifest.yaml
src/
    main.arb
    lib.arb
tests/
    main_test.arb
```

Both entry files are shown as possible roles, not a requirement that every
project contain both. Module naming derives predictably from paths; exact import
and entry-point rules remain to be specified.

Production source cannot import tests, test-only APIs, or test dependencies,
including through intermediate modules. Test privileges and dependencies do not
become available merely because production source is compiled for a test run.

## Related reading

- [Testing](testing.md)
- [Packages and dependencies](packages.md)
- [Compilation and toolchains](compilation.md)
