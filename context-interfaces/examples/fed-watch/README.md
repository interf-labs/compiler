# Fed watch

> For agents: use only the commands, flags and fields on this page, exactly as written. Run `interf --version` first; if it prints a different version, use `interf --help` and `interf <command> --help` instead of this page. llms.txt at the repository root lists every page.
>
> Checked against `@interf/compiler` 0.51.0.

This Context Interface describes a source-linked Graph for following the Federal
Open Market Committee (FOMC) across meetings: what it decided, how it assessed
the economy, what guidance it gave and who dissented.

Its required entrypoint `fed-watch.md` holds one policy path that links every
covered meeting. The `meetings/` directory holds the meeting, decision,
assessment, guidance and dissent nodes, connected by the declared relationships.
Source selection and Build instructions belong to each Graph's approved Build Plan.

Digest: sha256:7671c166d970b176d373566adf9e3818da9d0404092538d2254d5d02e00b160d

It is designed for FOMC statements and minutes published by the Board of
Governors of the Federal Reserve System. Cite the Board as the source; see its
[website disclaimer](https://www.federalreserve.gov/disclaimer.htm). This file
contains no FOMC text. [Prepare your first Graph](../../../first-graph/README.md)
uses it with four 2024 FOMC statements.

Check it offline, without an installation:

```sh
npx @interf/compiler context-interface validate --file context-interface.json
```
