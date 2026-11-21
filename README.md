# iproject-ibrain-packs

Public ibrain packs for the infinity-* applications. Every infinity-* project
keeps its project rules here, so ibrain owns the versioning of each pack the
same way it does for the library packs.

## Packs

| Pack | Use when |
| --- | --- |
| `infinity-qrcode` | The project is [infinity-qrcode](https://github.com/FortyTwo-Dev/infinity-qrcode) |
| `infinity-quiz` | The project is [infinity-quiz](https://github.com/FortyTwo-Dev/infinity-quiz) |
| `infinity-url` | The project is `infinity-url-api` or `infinity-url-web` |

New infinity-* projects add their pack here, one directory per project.
A pack directory is named after the `pack:` name in its manifest.

## Install

```sh
ibrain add https://github.com/FortyTwo-Dev/iproject-ibrain-packs --pack infinity-qrcode --ref main
```

Or declare the pack as a git source in a project's `ibrain.yaml`:

```yaml
packs:
  - source: git
    url: "https://github.com/FortyTwo-Dev/iproject-ibrain-packs"
    ref: "main"
    pack: "infinity-qrcode"
```

Then run `ibrain sync` from the project root. The project pins the pack commit
in its `ibrain.lock.yaml`; update it with `ibrain pull` and `ibrain sync`.

## Project layer

A project whose rules must **replace** a generic rule, instead of adding to it,
declares its pack as the project layer:

```yaml
packs:
  - source: git
    url: "https://github.com/FortyTwo-Dev/iproject-ibrain-packs"
    ref: "main"
    pack: "infinity-quiz"
    layer: project
```

- No `priority` on that entry: ibrain rejects it, the project layer already
  ranks above every pack.
- A rule replaces a generic one by carrying its id (`id: vue/testing`); without
  an id it adds to the resolved set.
- Author the rules in this clone. ibrain refuses `ibrain new rule` on a
  git-sourced project layer: it asks for a path checkout of the repository.
- One pack name per project: declaring the same pack by path and by git fails
  with `two packs declare the name ...`.

## Conventions

- One pack per project, in a directory matching the project name.
- Rules are written in English and scoped by the `paths:` frontmatter.
- Commits follow Conventional Commits, in English.
- A pack used as a plain dependency, without `layer: project`, does not replace
  a generic rule: its rules are added, ordered by the `priority` of the
  declaration. Use the project layer when a generic rule must be replaced.
