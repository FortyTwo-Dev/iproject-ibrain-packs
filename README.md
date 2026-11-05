# iproject-ibrain-packs

Public ibrain packs for the infinity-* applications. Every infinity-* project
keeps its project rules here, so ibrain owns the versioning of each pack the
same way it does for the library packs.

## Packs

| Pack | Use when |
| --- | --- |
| `infinity-qrcode` | The project is [infinity-qrcode](https://github.com/FortyTwo-Dev/infinity-qrcode) |

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

## Conventions

- One pack per project, in a directory matching the project name.
- Rules are written in English and scoped by the `paths:` frontmatter.
- Commits follow Conventional Commits, in English.
- Project packs override the generic packs (`global`, `vue`, ...) through a
  higher `priority` in the pack manifest.
