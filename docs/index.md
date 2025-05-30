# tdo - 🚧 WIP

Your entire life as a task tree, with leaves synced wherever you need them.

*[tdo.garden](https://tdo.garden)*

## Why?

- **Everything is a tree**: From personal goals to work projects, all your tasks branch from you
- **Distributed leaves**: tasks.md files live in git repos or local folders
- **Automatic sync**: Edit any leaf, your whole tree stays synchronized
- **Plain text files**: tasks.md files are human-readable, editable by anyone

## How it works

You are the root of your task tree. tdo maintains this complete tree centrally, while each folder contains only its relevant leaves (tasks.md files). 

tdo watches your tasks.md files for changes and automatically propagates updates to your central task repository (Taskwarrior). Edit any leaf, and your entire tree updates in real-time.

```
                        YOU
                         |
      ╭─────────┬────────┴────────┬──────────╮
      │         │                 │          │
    life    projects          personal   learning
      │         │                 │          │
   garden     tdo           writing    spanish
      │         │                 │          │
   plants    core              blog      verbs
   herbs     docs             ideas


Folders contain only the leaves (tasks.md):
──────────────────────────────────────────────
~/repos/tdo/            ~/life/garden/
├── tasks.md 🍃         └── plants/
└── core/                   └── tasks.md 🍃
    └── tasks.md 🍃    

Each task has a path: ~/repos/tdo/core/tasks.md → projects.tdo.core
```

*Note: tasks.md files work in both git repos and regular folders. See [FAQ](specification#faq) for details.*

## Documentation

See [specification](specification) for the full specification.

## License

MIT
