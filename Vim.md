# Vim

## Input

Enable insert mode key map to type in languages other than English.
Input language can be switched with `C-^` in insert mode.
```
nnoremap <Leader>k :set keymap=russian-jcukenwin<CR>
```

## Airline

Airline is useful as it is tightly integrated into various plugins like ALE and
git-gutter.
```
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
```

Add current working directory to airline status and disable fancy Unicode
symbols because not all console fonts support them (I use bitmap font that
doesn't):
```
let g:airline_section_c='%<%f%m (%{getcwd()})'
let g:airline_symbols_ascii = 1
```

## Themes

```
Plugin 'rakr/vim-one'
Plugin 'flazz/vim-colorschemes'
Plugin 'rafi/awesome-vim-colorschemes'
Plugin 'challenger-deep-theme/vim'
Plugin 'drewtempelmeyer/palenight.vim'
```

## Programming languages

ALE supports most of the lint tools and compiler straight out of the box.
I don't even have custom configuration for it, with an exception of few hotkeys.
```
Plugin 'dense-analysis/ale'

Plugin 'vlime/vlime'
Plugin 'elixir-lang/vim-elixir'
Plugin 'JuliaEditorSupport/julia-vim'
```

Structured editing for Lisps:
```
Plugin 'guns/vim-sexp'
Plugin 'tpope/vim-sexp-mappings-for-regular-people'
```

This custom hook is useful for adding new custom lisp words and enabling
highlighting for custom macros:
```
function s:ClojureHook()
  setlocal lispwords+=match
  syn match clojureMacro "match"
endfunction
autocmd FileType clojure call s:ClojureHook()
```

## Journaling

Inspired by [this post](https://peppe.rs/posts/plain_text_journaling/).
Plain text journals can be prefixed with a simple extension, like `.j`
```
function s:JournalFiletype()
  set formatprg=sort\ -V
  syn match JournalAll /.*/
  syn match JournalDone /^-.*/
  syn match JournalTodo /^!.*/
  syn match JournalEvent /^@.*/
  syn match JournalNote /^\..*/
  syn match JournalMoved /^>.*/
  hi JournalAll ctermfg=None
  hi JournalDone ctermfg=8
  hi JournalTodo ctermfg=15
  hi JournalEvent ctermfg=9
  hi JournalMoved ctermfg=12
  hi JournalNote ctermfg=10
endfunction
au BufRead,BufNewFile *.j call s:JournalFiletype()
```

Creating a new monthly journal entry is a bit tedious and can be automated with a bash script.
I saved this script as a `.local/bin/j` and can edit my journal by typing `j` in terminal.
```bash
#!/usr/bin/bash
J="$(date "+$HOME/journal/%Y/%m.j")"
if [ ! -e "$(dirname "$J")" ]; then
  mkdir -p "$(dirname "$J")"
fi
if [ ! -e "$J" ]; then
  vi "$J" +"read "'!LC_TIME="" '"cal -m $(LC_TIME="" date +%b)" +'%s/\s\+$//g'
else
  vi "$J"
fi
```

## Hosting a wiki

I use `vimwiki` plugin to write short work-related articles that can be
publicly accessed via the intranet. Regular markdown files are used for
short-lived notes.
```
Plugin 'vimwiki/vimwiki'
Plugin 'godlygeek/tabular'
Plugin 'plasticboy/vim-markdown'
```

Among other things markdown plugin supports concealing. However it does not
define bold and italic styles, which visually makes emphasised text look like
regular. Also folding causes problems with code blocks.
```
set conceallevel=2
highlight htmlItalic cterm=italic
highlight htmlBold cterm=bold
let g:vim_markdown_folding_disabled = 1
```

To enable spelling in wiki, markdown and git commit files by default:
```
autocmd FileType vimwiki setlocal spell
autocmd FileType markdown setlocal spell
autocmd FileType gitcommit setlocal spell
```

Wiki configuration:
```
let g:vimwiki_list = [{
      \ 'name': 'work',
      \ 'path': '~/vimwiki/',
      \ 'template_ext': '.template',
      \ 'template_default': 'default',
      \ 'css_name': 'sakura-dark.css',
      \ 'nested_syntaxes': {'bash': 'bash', 'java': 'java'}
      \ }]
```

Generated content is stored to `vimwiki_html` directory next to the source
path. This folder can be simply served by a nginx web server. My configuration
in `/etc/nginx/sites-available/vimwiki` looks like this:
```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  root /home/kozlov/vimwiki_html;
  index index.html;
  server_name _;
  location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
  }
}
```

My default template includes a [CSS theme](https://github.com/oxalorg/sakura),
[highlight.js](https://highlightjs.org/) package,
and [MathJax](https://www.mathjax.org/).
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="" xml:lang="">
  <head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes"/>
    <title>%title%</title>
    <style type="text/css">
code{white-space: pre-wrap;}
span.smallcaps{font-variant: small-caps;}
span.underline{text-decoration: underline;}
div.column{display: inline-block; vertical-align: top; width: 50%;}
    </style>
    <link rel="stylesheet" type="text/css" href="%root_path%sakura-dark.css" />
    <link rel="stylesheet" type="text/css" href="%root_path%atom-one-dark.min.css"/>
    <script src="%root_path%highlight.min.js"></script>
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
  </head>
  <body>
    <header id="title-block-header">
      <h1 class="title">%title%</h1>
    </header>
    %content%
  </body>
  <script>
    document.querySelectorAll('pre').forEach(el => hljs.highlightElement(el));
  </script>
</html>
```
