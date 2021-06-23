*(draft: this entire page)*

# Compiler Extensions
Compiler extensions are Macron packages that hook onto the compiler and perform arbitrary actions.

Compiler extensions are split into two kinds: pre-gen and post-gen. Pre-gen extensions are only applicable via the `-E`/`--extension` flag (Packages are searched in the current directory, then in the paths specified by the `MACRON_EXT_PATH` environment variable.)

Post-gen extensions can be added using the flag, or using the `ext_add` intrinsic.

*(draft)*
