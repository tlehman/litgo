# litgo

[![Go Reference](https://pkg.go.dev/badge/github.com/tlehman/litgo.svg)](https://pkg.go.dev/github.com/tlehman/litgo)
[![Go Report Card](https://goreportcard.com/badge/github.com/tlehman/litgo)](https://goreportcard.com/report/github.com/tlehman/litgo)
[![Go version](https://img.shields.io/github/go-mod/go-version/tlehman/litgo)](go.mod)
[![Release](https://img.shields.io/github/v/release/tlehman/litgo)](https://github.com/tlehman/litgo/releases/latest)
[![Build](https://github.com/tlehman/litgo/actions/workflows/pages.yml/badge.svg)](https://github.com/tlehman/litgo/actions/workflows/pages.yml)
[![License: MIT](https://img.shields.io/github/license/tlehman/litgo)](LICENSE)

Literate programming for Go. Write a program as Markdown; litgo tangles it to
source, weaves it to a page, proves what its `//@` annotations promise, and
runs it with every compiler error and every failed proof pointing at the
Markdown.

litgo is written in itself: [litgo.lit.md](litgo.lit.md) is the whole program
and its documentation. Every other file here is generated from it. Read it
woven at <https://tobilehman.com/litgo/>, which every push to `master` rebuilds.

![litgo Gopher logo](logo.jpg)

## Install

```sh
go install github.com/tlehman/litgo@latest
```

Or download a binary for Linux or macOS (amd64, arm64) from the
[latest release](https://github.com/tlehman/litgo/releases/latest).

## Use

```sh
litgo run    prog.lit.md   # tangle, prove, compile, run
litgo prove  prog.lit.md   # check the //@ annotations, and list what was proved
litgo check  prog.lit.md   # chunks, syntax, types and proofs, without writing anything
litgo tangle prog.lit.md   # write prog.go
litgo weave  prog.lit.md   # write prog.pdf (needs pandoc and typst)
```

The PDF's Mermaid diagrams are drawn by mermaid.js if `mmdc` is installed
(`npm install -g @mermaid-js/mermaid-cli`), and as box-character text if not.

Start from [examples/worker_pool.lit.md](examples/worker_pool.lit.md).

## Proofs

litgo checks [VeGo](https://arxiv.org/abs/2608.22630) annotations: contracts,
loop invariants and variants, measures for recursion, written as `//@`
comments in ordinary Go.

```go
//@ Requires n >= 0
func Isqrt(n int) (r int) {
	lo, hi := 0, n+1
	//@ Invariant 0 <= lo < hi
	//@ Invariant lo*lo <= n ^ n < hi*hi
	//@ Variant hi - lo
	for hi-lo > 1 {
		mid := lo + (hi-lo)/2
		if mid*mid <= n {
			lo = mid
		} else {
			hi = mid
		}
	}
	return lo
}
//@ Ensures r*r <= n ^ n < (r+1)*(r+1)
```

`litgo run` proves before it compiles, and a program that is not proved is not
run (`--no-prove` runs it anyway). The verifier is litgo's own, in pure Go with
no solver to install, since the paper's `vegop` is not published. It is sound
and deliberately small: integers, booleans, slices of integers that are read,
loops, calls and recursion. Anything else in an annotated function is reported
as not verified rather than passed. Integers are mathematical, so overflow is
not modelled.

[examples/isqrt.lit.md](examples/isqrt.lit.md) is the program above with its
argument written out, and
[examples/vego-tutorial.lit.md](examples/vego-tutorial.lit.md) goes through
every annotation, and [examples/heap.lit.md](examples/heap.lit.md) proves a
binary heap and a ring buffer. litgo proves the column arithmetic of its own source map
this way.

## Neovim

Renders LaTeX and Mermaid in the buffer, lines up the columns of Markdown
tables, shows type errors under the code as you type (`litgo lsp`), and adds
`:TangleCompileAndRun`, `:Prove`, `:Untangle`, `:RenameChunk`, `:Tangle`,
`:Weave`.

`//@` annotations are violet and their formulas are typeset like the math in
the prose, so `lo*lo <= n ^ n < hi*hi` reads `𝑙𝑜² ≤ 𝑛 ∧ 𝑛 < ℎ𝑖²` until the
cursor is on the line (`render.proofs = false` turns that off). Failed proofs
are violet too, and arrive as you type: red means the compiler will not take it, violet means it runs and
has not been shown to keep its promise. The groups are `LitgoProofKeyword`,
`LitgoProof`, `LitgoProofError`, `LitgoProofWarn` and `LitgoProofUnderline`.

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

When a run finishes, the output panel takes the cursor so that `q` closes it
and hands the window back; a run that ends at an error puts the cursor on the
error instead (`run.focus`, `run.jump`).

nvim-cmp and blink.cmp pick the server up by themselves. Without one of those,
the plugin turns Neovim's own completion popup on (`lsp.completion`: `"auto"`
by default, or `true`/`false` to decide it yourself).

A completion plugin offers the same sources everywhere in the buffer, so in a
Go block you also get Markdown snippets and words from the prose.
`require("litgo").in_go_block()` says whether the cursor is in a Go block,
which is all blink.cmp needs to leave them out:

```lua
sources = {
  default = function()
    if require("litgo").in_go_block() then
      return { "lsp", "path" }
    end
    return { "lsp", "path", "snippets", "buffer" }
  end,
}
```

## Hack

Edit `litgo.lit.md`, never the generated files.

```sh
go build -o bin/litgo .      # bootstrap from the committed sources
bin/litgo run litgo.lit.md   # tangle, prove, vet, test, rebuild bin/litgo
```

Setting a new `VERSION` in the command and pushing to `master` publishes that
release.

## License

[MIT](LICENSE)
