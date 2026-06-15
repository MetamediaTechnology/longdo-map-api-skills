# Longdo Map API Skills

An AI-agent skill for building web maps with the [Longdo Map][longdo] v3
JavaScript API and its REST APIs.

The skill lives in **[`SKILL.md`](./SKILL.md)** — point your AI coding agent
(Claude Code, etc.) at it, and it picks up the patterns and gotchas needed to
write correct Longdo Map code: drift-free HTML markers, directional/cluster
markers, click handling, popups, UI components, layer control, geolocation,
place tags, and the search/geocode/routing/traffic REST APIs.

## For humans

If you're a developer looking to learn the Longdo Map API directly:

- 🗺️ **Live examples:** https://mapdemo.longdo.com/
- 💡 **Official tips blog:** https://map-blog.longdo.com/longdo-map-api-tips/
- 📖 **JavaScript API docs:** https://map.longdo.com/docs/javascript/
- 🔌 **REST API docs:** https://map.longdo.com/docs/rest
- 🔑 **Get a free API key:** https://map.longdo.com/console/

## Using the skill

Reference `SKILL.md` from your agent's instructions, or drop it into a skills
directory your agent reads. It is self-contained Markdown — no setup, no
dependencies, no secrets.

> **Note:** never commit a real API key. The key is exposed to the browser by
> design, so load it from an environment variable / server-side template and
> restrict it to your domain(s) in the Longdo console.

## License

[Apache License 2.0](./LICENSE)

[longdo]: https://map.longdo.com/
