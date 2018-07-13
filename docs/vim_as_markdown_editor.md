# Vim as Markdown Editor

<!-- vim-markdown-toc GFM -->

* [基本功能支持](#基本功能支持)
* [目录生成[TOC]](#目录生成toc)
* [大纲](#大纲)
* [预览](#预览)
* [Markdown 检查](#markdown-检查)
* [文件导出/转换](#文件导出转换)
* [总结](#总结)

<!-- vim-markdown-toc -->

> Jul 13, 2018 Fri | vim | markdown

Markdown 编辑器很多，尝试过 maxiang，vscode + plugins 之类的，都不错。可没法使用Vim 的快捷键，不够顺手。vscode 虽然可以安装vim 模拟插件，可是每次打开文件要等待一两秒后，插件才能生效，用起来也是不顺畅。

试了一圈其它工具后，重回 Vim 倒腾，发现 Vim 通过配置一些插件，还是基本能满足 Markdown 文件编辑需要的。

使用 Vundle 安装要使用的插件：

```sh
Plugin 'godlygeek/tabular'
Plugin 'plasticboy/vim-markdown'
Plugin 'mzlogin/vim-markdown-toc'
Plugin 'iamcco/mathjax-support-for-mkdp'
Plugin 'iamcco/markdown-preview.vim'
Plugin 'dhruvasagar/vim-table-mode'
Plugin 'majutsushi/tagbar'
```

## 基本功能支持

通过 [vim-markdown](https://github.com/plasticboy/vim-markdown) 插件支持 Markdown syntax hightlight、~~TOC~~ Outline、Folding 这些基本功能。

```sh
# enable conceal
let g:vim_markdown_conceal = 2
let s:filetype_dict = {
    \ 'c++': 'cpp',
    \ 'bash': 'sh'
\ }

```

使用 [vim-table-mode](https://github.com/dhruvasagar/vim-table-mode) 用来编辑表格。使用 '||' abbreviation 启用 Table Mode。非常好用。🎉

```sh
function! s:isAtStartOfLine(mapping)
  let text_before_cursor = getline('.')[0 : col('.')-1]
  let mapping_pattern = '\V' . escape(a:mapping, '\')
  let comment_pattern = '\V' . escape(substitute(&l:commentstring, '%s.*$', '', ''), '\')
  return (text_before_cursor =~? '^' . ('\v(' . comment_pattern . '\v)?') . '\s*\v' . mapping_pattern . '\v$')
endfunction

inoreabbrev <expr> <bar><bar>
          \ <SID>isAtStartOfLine('\|\|') ?
          \ '<c-o>:TableModeEnable<cr><bar><space><bar><left><left>' : '<bar><bar>'
inoreabbrev <expr> __
          \ <SID>isAtStartOfLine('__') ?
          \ '<c-o>:silent! TableModeDisable<cr>' : '__'
```

![Table](images\vim_as_markdown_editor_table.png)

## 目录生成[TOC]

使用 [vim-markdown-toc](https://github.com/mzlogin/vim-markdown-toc) 生成、更新TOC。

```sh
# Disable auto update. Auto updating is stupid.
let g:vmt_auto_update_on_save = 0

# Generate Github Flavered Markdown style TOC
:GenTocGFM

:UpdateToc
:RemoveToc
```

![GFM TOC](images\vim_as_markdown_editor_toc01.png)

## 大纲

Vim 可以有多种方式实现大纲功能。

1. 通过 [vim-markdown](https://github.com/plasticboy/vim-markdown) folding 功能，折叠2级标题以下的内容，让Markdown 只显示文章的结构。

![vim_as_markdown_editor_outline](images\vim_as_markdown_editor_outline01.png)

参考配置：

```sh
# <space> to toggle folding
autocmd Filetype markdown nmap <space> zA

let g:vim_markdown_folding_style_pythonic = 1
let g:vim_markdown_override_foldtext = 0

```

**问题**：启用后，编辑时出现随机折叠。还是建议关闭此功能： 

```sh
let g:vim_markdown_folding_disabled = 1
```

2. tagbar + markdown2ctags.py

下载 markdown2ctags.py

```sh
curl -o markdown2ctags.py https://raw.githubusercontent.com/tamlok/vimconf/master/markdown2ctags.py
```

配置参考：https://github.com/tamlok/vimconf/blob/master/.vimrc

```sh
let file_markdown2ctags='~/.vim/markdown2ctags.py'
if g:iswindows
    let file_markdown2ctags=fnameescape($VIMFILES."\\markdown2ctags.py")
endif
if executable("python")
    let g:tagbar_type_markdown = {
        \ 'ctagstype': 'markdown',
        \ 'ctagsbin' : file_markdown2ctags,
        \ 'ctagsargs' : '-f - --sort=yes',
        \ 'kinds' : [
            \ 's:sections',
            \ 'i:images'
        \ ],
        \ 'sro' : '|',
        \ 'kind2scope' : {
            \ 's' : 'section',
        \ },
        \ 'sort': 0,
    \ }
endif
let g:tagbar_left=1
autocmd FileType markdown nmap <F8> :TagbarToggle<CR>
```

**问题**：每次tag 更新都要调用 markdown2ctags.py。windows 下会经常跳出 cmd 窗口运行 tag 更新命令，有点干扰注意力。

3. 使用 [vim-markdown](https://github.com/plasticboy/vim-markdown) TOC 功能。

```sh
let g:vim_markdown_toc_autofit = 1
# Auto open
au VimEnter *.md,*.markdown Toc

# Open / Update TOC
:Toc
```

![vim_as_markdown_editor_outline](images\vim_as_markdown_editor_outline02.png)

**问题**：大纲不能实时更新，需要用 :Toc 命令手动更新。😅

相对来说，第3重方法对编辑文件没有干扰，实时显示大纲也不算刚需。推荐使用。

## 预览

[markdown-preview](https://github.com/iamcco/markdown-preview.vim) 是目前能找到的最好的 Vim Markdown 实时预览插件。

```sh
let g:mkdp_path_to_chrome = 'firefox'
let g:mkdp_auto_start = 0
let g:mkdp_auto_close = 0
let g:mkdp_refresh_slow = 1

autocmd FileType markdown nmap <silent> <F10> <Plug>MarkdownPreview<esc>:sleep 2<esc>
autocmd FileType markdown imap <silent> <F10> <Plug>MarkdownPreview<esc>
```

![Preview](images\vim_as_markdown_editor_preview01.png)

这个插件目前还不支持 flow chart。要在 Markdown 中使用流程图，可以使用 [vscode + MPE](https://shd101wyy.github.io/markdown-preview-enhanced/#/diagrams)。

**问题**：使用中，出现过卡死 🐞。Vim 对于 Markdown 预览这种最基础功能的支持还是不让人满意。

## Markdown 检查

vscode + markdownlint 很不错，尤其对于刚使用 Markdown 不久的人。

Vim 还没有这么好用的 lint。所以，再装个 vscode 还是有用的。😂

Markdown 语法本身很简洁，容易掌握，规范检查功能对于熟悉 Markdown 的人并不重要。规范是死的，编辑起来自己舒服最重要。

## 文件导出/转换

使用 [markdown-preview](https://github.com/iamcco/markdown-preview.vim) + 装有 markdown viewer 插件的 chrome，打开文件后，可以保存为 HTML。或是，Ctrl + P，再导出为PDF。

![Export to PDF](images\vim_as_markdown_editor_export_to_pdf.png)

借助 vscode + Markdown PDF / MPE 是个不错的导出方法。

或是，使用 pandoc。

## 总结

除了对实时预览功能不太满意，Vim 用于 Markdown 文件编辑是完全胜任的。😀
