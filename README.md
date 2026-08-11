# jReference

*Just a Reference.*

A personal collection of buying doctrine, project design docs, and reusable code — the reference material for how things get built and bought around here.

## Structure

```
jReference/
├── doctrine/       Evergreen personal principles, not tied to any one project
├── projects/       Project-specific design docs, one folder per project
├── snippets/       Portable, reusable code — write once, use forever
└── templates/      Reusable structures for future docs/projects
```

### `doctrine/`
Standing rules for evaluating tools, parts, services, and designs across projects.

- [`no-paperweight-buying-guide.md`](doctrine/no-paperweight-buying-guide.md) — purchasing/sourcing principles: no proprietary consumables, no ongoing subscriptions (with a swappable-service exception), evidence of need before cost (blocking for the future is fine, buying for it isn't), avoid obscurity in sourcing, fewer failure points over more capability.

### `projects/`
One folder per project. Each holds that project's design docs, decisions, and open items.

- [`ro-system/`](projects/ro-system/) — bespoke under-sink reverse osmosis water filtration system, fully modular, no proprietary consumables.

### `snippets/`
Reusable code, organized by language first (matches how you'd actually search for it later — "I need a bash one-liner" not "I need something in the automation category"). Function-based subfolders get added inside a language folder only once it's crowded enough to need them.

### `templates/`
Blank starting structures extracted from things that already worked, so the next project doesn't start from a blank page.

## License
MIT — see [LICENSE](LICENSE).
