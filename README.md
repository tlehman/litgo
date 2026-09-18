# litgo

Literate programming for Go. Write a program as Markdown; litgo tangles it to
source, weaves it to a page, and runs it with every compiler error pointing at
the Markdown.

litgo is written in itself: [litgo.lit.md](litgo.lit.md) is the whole program
and its documentation. Every other file here is generated from it.

## Install

```sh
go install github.com/tlehman/litgo@latest
```

## Use

```sh
litgo run    prog.lit.md   # tangle, compile, run
litgo check  prog.lit.md   # type-check without writing anything
litgo tangle prog.lit.md   # write prog.go
litgo weave  prog.lit.md   # write prog.pdf (needs pandoc and typst)
```

Start from [examples/worker_pool.lit.md](examples/worker_pool.lit.md).

## Neovim

Renders LaTeX and Mermaid in the buffer, lines up the columns of Markdown
tables, shows type errors under the code as you type (`litgo lsp`), and adds
`:TangleCompileAndRun`, `:Untangle`, `:RenameChunk`, `:Tangle`, `:Weave`.

With [gopls](https://go.dev/gopls/) installed (on `$PATH`, in `$GOPATH/bin`, or
by Mason), a Go block is a Go buffer: completion, hover, go to definition,
references, rename and signature help work across blocks and chunks, because
`litgo lsp` asks gopls about the tangled program and translates the answers
back to the Markdown. On a chunk reference the same keys navigate, preview and
rename the chunk.

```lua
{
  "tlehman/litgo",
  build = "go build -o bin/litgo .",
  event = { "BufReadPre *.lit.md", "BufNewFile *.lit.md" },
  opts = {},
}
```

## Hack

Edit `litgo.lit.md`, never the generated files.

```sh
go build -o bin/litgo .      # bootstrap from the committed sources
bin/litgo run litgo.lit.md   # tangle, vet, test, rebuild bin/litgo
```
