# Module-aware GoDoc

This came about based on my struggles when trying to generate a GoDoc Docset for
[Dash](https://kapeli.com/dash)/[Zeal](https://zealdocs.org/) using
[wuudjac/godocdash](https://github.com/wuudjac/godocdash). godocdash, by default,
generates a Docset for all of the packages included in `$GOPATH/src`, However, if
you are using modules for you projects, a lot of packages are probably under
`$GOPATH/pkg/mod` instead i.e., the "module cache".

Since the module cache is structured differently, running `GOPATH=$HOME/pkg/mod godoc`
does not produce what you probably want, which is to also have third-party libraries'
documentation. Similarly, `GOPATH=$HOME/pkg/mod godocdash` results in no third-party
documentation being generated.

This all started because I wanted to be able to refer to local documentation for
libraries that I frequently use, such as [github.com/peterbourgon/ff/v3/ffcli/](https://pkg.go.dev/github.com/peterbourgon/ff/v3/ffcli?tab=overview),
but I didn't really want to waste space storing local documenation for _every_ third-party
package that I use. We will use `ffcli` as an example for "testing" purposes since
it supports modules and the latest version is `v3`.

The `godoc` binary is now module-aware. This basically just means that if you run
`godoc` in a directory with a `go.mod` file, it can server the GoDocs locally, which
is awesome! Otherwise, when run outside of a module, `godoc` serves, "non-versioned"
packages from `$GOPATH/src`.

This [GitHub issue](https://github.com/golang/go/issues/33655), specifically
[this comment](https://github.com/golang/go/issues/33655#issuecomment-534457813)
documents a nice way to specify the exact packages that you want to serve local
documentation for.

Finally, because `godoc` is module-aware, so is `godocdash`! This is because `godocdash`
[runs the `godoc` binary](https://github.com/wuudjac/godocdash/blob/4d71e6a68077b16033b64f313bba81b2db64e319/main.go#L187)
and calls that local GoDoc server when generating the Dash DocSet, perfect! To confirm,
you should see the following:

```bash
$ godocdash
using module mode; GOMOD=/home/mccurdyc/go/doc/go.mod
...
```

## Adding Docs

It's literally just adding a dependency to the module and running `godocdash`.

1. `go get github.com/peterbourgon/ff/v3/ffcli@latest`
2. `godocdash`
3. Zeal requires an app restart to reload the downloaded Docsets.

## Updating Docs

It's literally just updating dependencies and re-running `godocdash`.

1. `go get -u ./...`
2. `godocdash`
3. Zeal requires an app restart to reload the downloaded Docsets.

## Other Notes

I have a symlink that points `$HOME/.zeal/docsets/GoDoc.docset` to the `GoDoc.docset`
in my doc module path.

```bash
$ ln -snf $HOME/src/github.com/mccurdyc/godocmod/GoDoc.docset $HOME/.zeal/docsets/GoDoc.docset
```
