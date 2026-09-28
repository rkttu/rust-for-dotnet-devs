# Rust for C#/.NET Developers

The document sources can be found in the `src` directory. It is structured for
rendering with [mdBook].

[Install mdBook] locally or start-up [`devcontainer.json`]. If you've never
used Dev Containers, check out [Developing inside a Container using Visual
Studio Code Remote Development][vscode-dc].

Then to render for reading in a Web browser, run:

    mdbook serve

## Korean translation (work in progress)

A Korean edition is being prepared in [`ko/`](ko/). It currently includes the
introduction and getting started chapter. To preview the translated chapters,
run `mdbook serve ko` from the repository root. The original English book
remains in `src/`. The upstream translation discussion is in [issue #47].

[issue #47]: https://github.com/microsoft/rust-for-dotnet-devs/issues/47

  [mdBook]: https://rust-lang.github.io/mdBook/
  [Install mdBook]: https://rust-lang.github.io/mdBook/guide/installation.html
  [`devcontainer.json`]: .devcontainer/devcontainer.json
  [vscode-dc]: https://code.visualstudio.com/docs/devcontainers/containers
