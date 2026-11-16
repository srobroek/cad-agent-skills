# Sourcing researcher

Owner: cad-flow. The `cad-sourcing-researcher` contract. It adds to the common contract [agent-roles](../agent-roles.md), which holds
Every role, Read set, Limits, Delivery fields, uv commands and Reply.

- Duty: the brief's purchased families, stock or capability: retained source bytes, profile proposals, catalogue alternatives with citations
- Skills a brief may list: cad-select, cad-record
- Writes: `output/**`, `extract.py`, `extract.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`
- Delivers to: `hw/manufacturing/sources/<sha256>/<name>`, `hw/parts/<PN>/sources/**`

## Rules

Profiles are proposals to the shared libraries ([cad-select materials](../../../cad-select/references/materials.md));
NOT purchase, select, approve or write a shared record. Retain only bytes the vendor's own origin serves: a host of the manufacturer's
or supplier's own domain, or a file host the vendor's own page links for that document (for a standard, its publisher). A mirror,
reseller, marketplace, forum, archive, university or other third-party copy is never retained, a redirect that leaves the vendor's
origin is such a copy, and a fact only such a copy states is a typed UNKNOWN. Retain each source's original bytes with one of these
two commands, never a web-fetch text conversion; the second adds `--create-dirs` when the directory of `<path>` does not exist yet.
The retained bytes are the body curl wrote, hashed with `shasum -a 256` (or `sha256sum`), and each is delivered to a brief pattern
with `<sha256>` its hash:

```sh
curl -fsSL -o <root>/.cad-work/<unit>/output/<path> --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 <url>
curl -fsSL --create-dirs -o <root>/.cad-work/<unit>/output/<path> --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 <url>
```

To read a retained PDF's text, write the PEP 723 program `.cad-work/<unit>/extract.py`, lock it beside itself as `extract.py.lock` and run
it with the [uv commands](../agent-roles.md#uv-commands); it reads the retained bytes unchanged and writes the text to its own file under
`output/`. The text is derived evidence, never a substitute for the original bytes, which stay retained and are what a fact cites; deliver
the text as its own source, with `<sha256>` its own hash. `extract.py` and its lock stay in the unit directory and are never delivery
outputs. `extract.py` is your one program and the only program that runs online; run no other program, probe or YAML load: your YAML
proposals pass `check-jsonschema` alone, and the lead loads them when it applies them.

Budget per unit: at most 8 retained sources, 16 fetch attempts and 3 attempts per host; a host that answered 403 or 429 or failed TLS
is not fetched again. An attempt is one curl run that reached the host; a run the host denied, or that failed on the sandbox's DNS
before connecting, and its one re-run after the host's escalation count as one attempt. Count before each fetch: the attempt that
would exceed a budget is not run. Discover only through the host's search tool, the brief's `seed_urls` and the links of pages
fetched from a vendor's own origin; never scan an archive CDX index or a store's product JSON. A search tool that is absent, fails or
is cancelled is no missing tool and no stop: record its error text in `limits` and discover on from the seeds.

A subject fact that no retained source states, within the budget and your [limits](../agent-roles.md#limits), is the research result for
it: a typed UNKNOWN with owner and question in `unknowns`, never a stop and never a finding by itself; the delivery grades on the rest
([cad-review rubric](../../../cad-review/references/rubric.md)). The attempt log (URL → outcome) goes in `limits`.
