# Rust for C#/.NET Developers

The document sources can be found in the `src` directory. It is structured for
rendering with [mdBook].

[Install mdBook] locally or start-up [`devcontainer.json`]. If you've never
used Dev Containers, check out [Developing inside a Container using Visual
Studio Code Remote Development][vscode-dc].

Then to render for reading in a Web browser, run:

    mdbook serve

## Korean translation

The complete Korean edition is in [`ko/`](ko/). To preview it, run
`mdbook serve ko` from the repository root. Its [translation notes] document
the source revision, review changes, and staged commit history. The original
English book remains in `src/`. The upstream translation discussion is in
[issue #47].

[issue #47]: https://github.com/microsoft/rust-for-dotnet-devs/issues/47
[translation notes]: ko/TRANSLATION_NOTES.md

  [mdBook]: https://rust-lang.github.io/mdBook/
  [Install mdBook]: https://rust-lang.github.io/mdBook/guide/installation.html
  [`devcontainer.json`]: .devcontainer/devcontainer.json
  [vscode-dc]: https://code.visualstudio.com/docs/devcontainers/containers
