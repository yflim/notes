
# Table of Contents

1.  [Evaluating Emacs-Lisp](#org9ba1342)
2.  [Editing](#org246273d)
    1.  [Selecting](#org973dea7)
    2.  [Project search](#org68502c0)
    3.  [Project search and replace](#org0284235)
    4.  [Miscellaneous](#orga94733f)
3.  [Modes](#org14b4edd)
4.  [Org-mode](#org79a0851)
    1.  [Tables](#org6e071c6)
    2.  [Org roam](#org53bbea6)
5.  [Navigation](#org430504f)
6.  [Cider](#org50cc42e)
    1.  [Debugger](#org1264937)
7.  [Files](#org64480f3)
8.  [Dired](#orgbf64bee)
9.  [Buffers](#orgcb88473)
10. [Windows](#org2cc21d0)
11. [Environment](#org916146e)
12. [Help](#org6da04cc)
13. [Search, query, and replace](#orgbc66a5a)
    1.  [rg: Emacs UI for ripgrep](#org4e41be1)
        1.  [`rg` search & replace example:](#orgfc119a7)
14. [Basics](#orga186620)
15. [In case of emergency](#orgdca67bf)
16. [Doom](#orgace697c)
17. [Maybe useful someday](#org74b7ac8)
18. [map cider-repl-history-search-backward (in cider-repl-history-mode)?](#orgbaff003)

**Note:** *M-x and SPC : interchangeable in most or all cases*


<a id="org9ba1342"></a>

# Evaluating Emacs-Lisp

-   Read 1 expr in minibuffer, eval, print val in echo area: `SPC ;`
-   Eval expr before point, print val [in echo area?]: `SPC m e e`


<a id="org246273d"></a>

# Editing


<a id="org973dea7"></a>

## Selecting

-   At point
    -   Word: `viw`
    -   Block (by indentation): `vii`
    -   Block (by braces): `vib`
    -   Sentence: `vis`
    -   Paragraph: `vip`

-   Entire buffer: `C-x h`


<a id="org68502c0"></a>

## Project search

-   Search project (if !, include hidden files): `:pg[!] <query>`
-   `rg`


<a id="org0284235"></a>

## Project search and replace

-   Search project: `SPC s p`
-   Search another project: `SPC s P`
-   Search this directory: `SPC s d`
-   Search another directory: `SPC s D`


<a id="orga94733f"></a>

## Miscellaneous

-   String search (necessary in `cider-repl-history`): `SPC s s`
-   `insert-char`: `Ctrl+x 8 RET`
    -   E.g. zero-width space to escape special chars
-   Paste from register: `Ctrl-r p` (*r* for register)
-   Reindent selected region: `C-M-\`


<a id="org14b4edd"></a>

# Modes

-   Activate e.g.: M-x org-mode


<a id="org79a0851"></a>

# Org-mode

-   Toggle expand / collapse current section:
    1.  Go to previous visible heading: `C-c C-p`
    2.  `tab` on heading asterisk
-   Cycle through expanding / collapsing all sections (maximal expansion is followed by collapsing all): `shift-tab`
-   Add list item: `M-RET`

-   **Bold**, *italics*, <span class="underline">underline</span>, <del>strikethrough</del>
-   `code` and `verbatim` need to be the inner-most markers since their contents are interpreted literally.
-   Todo: any heading becomes a todo when started with `TODO`
    -   Add todo to heading: `C-c C-t` (will pull up menu) followed by `t`
    -   Toggle: `ret`
-   Preview: `M-x markdown-preview`
-   Export:
    -   Add backend to `org-export-backends` if necessary.
    -   Export e.g. `M-x org-md-export-to-markdown`. The exported file should be in the same directory.

-   Code block
    -   Init: type `<s-tab`
    -   Exec: `ret`
        
        -   sh example:
        
            cd ~
            ls
        
        -   elisp example:
        
            (format "hi there")
        
        -   python example: prints return value by default; remove `:results output` to restore.
        
            print("hello")
    -   Pop up buffer with code: `Ctrl-c '`
    -   Session
        
            v = 1 + 2
        
            print(v)
        
        -   Or set property: &#x2026; incomplete or doesn&rsquo;t work!!!
            
            :header-args: :session py2
            
                v = 1 + 2
            
                print(v)
            
                sum(range(5))
        
        -   Name block for referencing:
            
                def sum_range(n):
                    sum(range(n))

    hello


<a id="org6e071c6"></a>

## Tables

-   Creation
    -   Type &ldquo;|&rdquo; after any amount of white space
    -   `C-c |`: will be asked for table structure (rows and cols)
-   Navigation
    -   Next cell: `tab`
    -   Previous cell: `shift-tab`
    -   Next cell in same column: `ret`
        -   Will create new row if that cell doesn&rsquo;t exist, or is beyond a separator line
    -   Moving rows/columns
        -   Move current column to the left/right (command mode only): `M-<left|right>`
        -   Move current row to up/down (command or insert mode): `M-<up|down>`
-   Align right cell border to content: `tab`
-   Add row: `M-ret`
-   Examples
    -   Regular:
        
        <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
        
        
        <colgroup>
        <col  class="org-right" />
        
        <col  class="org-right" />
        </colgroup>
        <tbody>
        <tr>
        <td class="org-right">A</td>
        <td class="org-right">B</td>
        </tr>
        
        
        <tr>
        <td class="org-right">1</td>
        <td class="org-right">2</td>
        </tr>
        </tbody>
        </table>
    -   Bold header:
        
        <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
        
        
        <colgroup>
        <col  class="org-right" />
        
        <col  class="org-right" />
        </colgroup>
        <thead>
        <tr>
        <th scope="col" class="org-right">A</th>
        <th scope="col" class="org-right">B</th>
        </tr>
        </thead>
        
        <tbody>
        <tr>
        <td class="org-right">1</td>
        <td class="org-right">2</td>
        </tr>
        
        
        <tr>
        <td class="org-right">&#xa0;</td>
        <td class="org-right">&#xa0;</td>
        </tr>
        </tbody>
        </table>


<a id="org53bbea6"></a>

## Org roam

-   Select or create node: `org-roam-node-find`

-   Tag heading: see example right above


<a id="org430504f"></a>

# Navigation

-   Switch to buffer `SPC <`
-   Switch (toggle) buffer: `C-x O`
-   Switch to REPL buffer: `C-x p`
-   Previous buffer (within same window): `C-x <left>`
-   Jump to file in project (helm-projectile-find-file): `SPC SPC`
    -   To first refresh: `SPC u SPC SPC`
-   Open recent file (helm/workspace-mini): `SPC ,`
-   Find files in private config: `SPC f p`
-   Find files in project: `SPC p f`
-   helm-find-files: `SPC .`
-   helm-refresh: `C-c C-u` (helm-map)
-   Switch project: `SPC p p`
-   Toggle file explorer: `SPC o p`
-   Open recent file in any project: `SPC f r`
-   Open recent file in current project: `SPC f R`


<a id="org50cc42e"></a>

# Cider

-   `cider-inspector-next-page`: `C-j`
    -   previous page: `C-k`
-   History: `C-c M-p` (`cider-repl-history`)
-   Evaluate expression: `SPC m e e`
-   Evaluate region: `SPC m e r`
-   Comment section for inline evaluation, e.g.:
    
        (comment
          (js/Date.)
          (:all-messages (make-mailbox)))
-   `M-x cider-jack-in-clj`: `SPC m '`
-   `M-x cider-jack-in-cljs`: `SPC m "`
-   `M-x cider-connect`: `SPC m c`
-   Toggle between Clojure[Script] buffer and REPL: `C-c C-z`
-   Look up definition: `M-.`
-   Look up documentation: `C-c C-d C-d` (cider-mode-map), `SPC m h d` (clojure-mode-map)
-   Public symbols apropos: `C-c C-d a`
-   Documentation apropos: `C-c C-d f`
-   `cider-repl-set-~ns: ~SPC m n`
-   `cider-doc: ~SPC ~m h d`
-   Show references to fn. at point: `C-c C-? r`
-   Reload modified and unloaded namespaces on classpath: `cider-ns-refresh`.
    -   cider-repl-mode `SPC m r`, clojure-mode `SPC m r r`


<a id="org1264937"></a>

## Debugger

-   Eval current toplevel form; print result in minibuffer (`cider-eval-defun-at-point`): `SPC m e d`
    -   Also for instrumenting after adding `#break` statements
-   The above plus instrumentation by Cider, `cider-eval-defun-at-point` with `DEBUG-IT` prefix or its equivalent `cider-debug-defun-at-point`: `SPC m d d`
    -   Remove instrumentation by evlauating normally again, using `cider-eval-defun-at-point`


<a id="org64480f3"></a>

# Files

-   Move/Rename file: `SPC f R`
-   Delete file: `SPC f D`


<a id="orgbf64bee"></a>

# Dired

-   Toggle directory metadata display: `shift-9`
-   Move up a level: `-`
-   Select then delete a file/dir: `d x`
-   Sort by name/date in ascending order: `o`
-   Modify permissions: `shift+M`, then e.g. `u+x,g-w`
-   Select all directories: `* /`
-   Unselect all: `U`
-   Select: `m`
-   Unselect: `u`
-   Toggle selection: `t`
-   Move selected: `R`


<a id="orgcb88473"></a>

# Buffers

-   Refresh buffer: `C-x C-v`
-   list-buffers: `C-x C-b`
-   In list-buffers buffer:
    -   Mark buffer: `m`
    -   Mark for deletion (after `m`): `C-k`
    -   Execute deletion (after preceding keystrokes): `x`


<a id="org2cc21d0"></a>

# Windows

-   Open new window in vertical split: `C-x 3`
-   Open new frame: `SPC o f`
-   Enable line numbers: M-x linum-mode


<a id="org916146e"></a>

# Environment

-   Show variable value: `C-h v`
-   Show current major mode: `M-:` / `C-h v major-mode`
-   List and display docs of current major and minor modes: `C-h m`
-   Effect changes in .doom.d/init.el:

$ `doom sync`
In Emacs: M-x doom/reload i.e. `SPC h r r`


<a id="org6da04cc"></a>

# Help

-   List doom keybindings: SPC ?
-   Describe key sequence in current context: SPC h k
-   Show help for known fn: M-x <fn-name>
-   Describe fn: SPC h f <fn-name>
-   Find source of Emacs command: `M-x find-function RET fn-name`
-   See all keybindings for a major mode: SPC h b f
-   See all keybindings available in a buffer, in order of precedence (describe-bindings): SPC h b b
-   Functions and variables related to search terms: M-x apropos <term[s]> / C-x c a
-   Find man page: C-x c m
-   Display elisp function documentation: C-h f fn RET
-   Go to module&rsquo;s documentation: `M-x doom/help-modules` or `SPC h d m`


<a id="orgbc66a5a"></a>

# Search, query, and replace

-   org query language: [org-ql](https://github.com/alphapapa/org-ql)
-   Search and replace tools: `query-replace`, `multiple-cursors.el`, `iedit`, etc.


<a id="org4e41be1"></a>

## [rg](https://github.com/dajva/rg.el): Emacs UI for [ripgrep](https://github.com/BurntSushi/ripgrep)

`*rg*` (results) buffer keybindings, defined in `rg-mode-map`:

-   `n`, `next-error-no-select`: move to the next line with a match, show that file in other buffer and highlight the match
-   `p`, `previous-error-no-select`: ditto with the previous line with a match
-   `f`, `rg-rerun-change-files`
-   `r`, `rg-rerun-change-regexp`
-   go backward and forward in search history:
    -   `C-c <`, `rg-forward-history`
    -   `C-c >`, `rg-back-history`
-   `m`, `rg-menu`: pops up a menu
    -   E.g.: To use flag `--context` or `-C` for including lines before & after each match:
        -   hit `m`, then `-C`
        -   when the minibuffer displays `--context=`, hit `2` and `RET`
        -   hit `g` for `rg-recompile`
-   Bonus: with `org-mode`, e.g.
    ~ If you try to open the following org link (in a `org-mode` buffer), Emacs will request confirmation to execute it:
    `[[elisp:(rg-run "\\(defmacro with" "subr.el" "/tmp/emacs/" nil nil '("--context=2"))]]`
    If yes, Emacs runs a search and displays results in `*rg*`


<a id="orgfc119a7"></a>

### `rg` search & replace example:

-   Suppose we want `org-link-expand-abbrev` -> `org-link-RENAMED`
-   Search for the former, then in `*rg*`:
    -   Hit `e` for `wgrep-change-to-wgrep-mode`; the following happen:
        -   the matched lines are now editable in `*rg*`
        -   the keymap `wgrep-mode-map` becomes the local map
    -   Replace to heart&rsquo;s content
    -   Abort! Abort! `C-c C-k` for `wgrep-abort-changes`
    -   Which reverts the changes and `*rg*` to &ldquo;normal&rdquo;
    -   Note that until we explicitly run a command e.g. `wgrep-abort-changes` of `wgrep` package,
        nothing is reflected in the filesystem (nor in new file buffers)
    -   Hit `e` again, and edit
    -   To save changes in `*rg*`: `C-x C-s` `wgrep-finish-edit`
    -   `*rg*` is &ldquo;normal&rdquo; (not editable) again, and:
        -   The changes are visible *but not saved* in the corresponding file buffers
        -   Can be undone manually
    -   To save to file system: `M-x wgrep-save-all-buffers`
    -   To save automatically on `wgrep-finish-edit`:
        `(setq wgrep-auto-save-buffer t)`


<a id="orga186620"></a>

# Basics

-   Doom equivalent of `M-x`: `SPC :`


<a id="orgdca67bf"></a>

# In case of emergency

-   After a crash, files can be restored with `M-x` and `recover-file`, `recover-session`, or `recover-this-file`.
    -   `auto-save-default` makes copies of files in `\~/.emacs.d/.local/cache/{autosave,backup}`, then deletes them when the buffer is saved.


<a id="orgace697c"></a>

# Doom

-   Upgrading: Follow [instructions](https://github.com/doomemacs/doomemacs/blob/master/docs/getting_started.org#updowngrading-emacs); make sure to check for announcements and such about changes to the script.


<a id="org74b7ac8"></a>

# Maybe useful someday

-   Tiling window manager (edwina?): gives full control over where things open
-   Dedicated reading(???) mode: [olivetti](https://github.com/rnkn/olivetti)
-   Highlight PDF documents: pdf-tools, org-noter (&ldquo;does this job better&rdquo;).
-   Bibliographies:
    -   Export Zotero library to `.bib` format
    -   Manipulate the `.bib` file with packages e.g. Ivy-bibtex, org-roam-bibtex or org-ref


<a id="orgbaff003"></a>

# TODO map cider-repl-history-search-backward (in cider-repl-history-mode)?

