
# Table of Contents

1.  [Evaluating Emacs-Lisp](#orge6b6611)
2.  [Editing](#org176a681)
    1.  [Select at point](#orgc2083e0)
    2.  [Project search ex commands](#orgd05ef2e)
    3.  [Project search and replace](#org36fd84c)
    4.  [Miscellaneous](#org78d62f5)
3.  [Modes](#org7b99f97)
4.  [Org-mode](#org920a708):emacs:org_mode:
    1.  [Tables](#org898c972)
5.  [Navigation](#orga35b8a7)
6.  [Cider](#org47523d2)
    1.  [Debugger](#org8f0bbf4)
7.  [Files](#org24c534a)
8.  [Dired](#orgbe481ca)
9.  [Buffers](#org04f6deb)
10. [Windows](#org33bd02d)
11. [Environment](#orge0e255a)
12. [Help](#orgd6fd58d)
13. [Basics](#org713d313)
14. [map cider-repl-history-search-backward (in cider-repl-history-mode)?](#orgfa0ecc3)

**Note:** *M-x and SPC : interchangeable in most or all cases*


<a id="orge6b6611"></a>

# Evaluating Emacs-Lisp

-   Read 1 expr in minibuffer, eval, print val in echo area: `SPC ;`
-   Eval expr before point, print val [in echo area?]: `SPC C-e`


<a id="org176a681"></a>

# Editing


<a id="orgc2083e0"></a>

## Select at point

-   Word: `viw`
-   Block (by indentation): `vii`
-   Block (by braces): `vib`
-   Sentence: `vis`
-   Paragraph: `vip`


<a id="orgd05ef2e"></a>

## Project search ex commands

-   Search project (if !, include hidden files): `:pg[!] <query>`


<a id="org36fd84c"></a>

## Project search and replace

-   Search project: `SPC s p`
-   Search another project: `SPC s P`
-   Search this directory: `SPC s d`
-   Search another directory: `SPC s D`


<a id="org78d62f5"></a>

## Miscellaneous

-   String search (necessary in `cider-repl-history`): `SPC s s`
-   `insert-char`: `Ctrl+x 8 RET`
    -   E.g. zero-width space to escape special chars


<a id="org7b99f97"></a>

# Modes

-   Activate e.g.: M-x org-mode


<a id="org920a708"></a>

# Org-mode     :emacs:org_mode:

-   Tag heading: see example right above

-   Expand / Collapse: `shift`
-   Indent heading: `tab`
    -   Decrease indent: `shift-tab`
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

-   Init code block: type `<s-tab`
-   Exec code block: `ret`
    
        cd ~
        ls

    (format "hi there")

    v = 1 + 2
    return v


<a id="org898c972"></a>

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


<a id="orga35b8a7"></a>

# Navigation

-   Switch to buffer SPC <
-   Switch (toggle) buffer: `Ctrl+x O`
-   Switch to REPL buffer: `Ctrl+x p`
-   Jump to file in project (helm-projectile-find-file): SPC SPC
    -   To first refresh: SPC u SPC SPC
-   Open recent file (helm/workspace-mini): SPC ,
-   Find files in private config: SPC f p
-   Find files in project: SPC p f
-   helm-find-files: SPC .
-   helm-refresh: C-c C-u (helm-map)
-   Switch project: SPC p p
-   Toggle file explorer: SPC o p
-   Open recent file in any project: SPC f r
-   Open recent file in current project: SPC f R


<a id="org47523d2"></a>

# Cider

-   History: `C-c M-p` (`cider-repl-history`)
-   Evaluate expression: SPC m e e
-   Evaluate region: SPC m e r
-   Comment section for inline evaluation, e.g.:
    
        (comment
          (js/Date.)
          (:all-messages (make-mailbox)))
-   M-x cider-jack-in-clj: SPC m &rsquo;
-   M-x cider-jack-in-cljs: SPC m &ldquo;
-   M-x cider-connect: SPC m c
-   Toggle between Clojure[Script] buffer and REPL: C-c C-z
-   Look up definition: M-.
-   Look up documentation: C-c C-d C-d (cider-mode-map), SPC m h d (clojure-mode-map)
-   Public symbols apropos: C-c C-d a
-   Documentation apropos: C-c C-d f
-   cider-repl-set-ns: SPC m n
-   cider-doc: SPC m h d
-   Show references to fn. at point: C-c C-? r
-   Reload modified and unloaded namespaces on classpath: cider-ns-refresh.
    -   cider-repl-mode SPC m r, clojure-mode SPC m r r


<a id="org8f0bbf4"></a>

## Debugger

-   Eval current toplevel form; print result in minibuffer (`cider-eval-defun-at-point`): `SPC m e d`
    -   Also for instrumenting after adding `#break` statements
-   The above plus instrumentation by Cider, `cider-eval-defun-at-point` with `DEBUG-IT` prefix or its equivalent `cider-debug-defun-at-point`: `SPC m d d`
    -   Remove instrumentation by evlauating normally again, using `cider-eval-defun-at-point`


<a id="org24c534a"></a>

# Files

-   Move/Rename file: `SPC f R`
-   Delete file: `SPC f D`


<a id="orgbe481ca"></a>

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


<a id="org04f6deb"></a>

# Buffers

-   Refresh buffer: `C-x C-v`
-   list-buffers: `C-x C-b`
-   In list-buffers buffer:
    -   Mark buffer: `m`
    -   Mark for deletion (after `m`): `C-k`
    -   Execute deletion (after preceding keystrokes): `x`


<a id="org33bd02d"></a>

# Windows

-   Open new window in vertical split: `C-x 3`
-   Open new frame: `SPC o f`
-   Enable line numbers: M-x linum-mode


<a id="orge0e255a"></a>

# Environment

-   Show variable value: `C-h v`
-   Show current major mode: `M-:` / `C-h v major-mode`
-   List and display docs of current major and minor modes: `C-h m`
-   Effect changes in .doom.d/init.el:

$ doom sync
In Emacs: M-x doom/reload i.e. `SPC h r r`


<a id="orgd6fd58d"></a>

# Help

-   List doom keybindings: SPC ?
-   Describe key sequence in current context: SPC h k
-   Show help for known fn: M-x <fn-name>
-   Describe fn: SPC h f <fn-name>
-   See all keybindings for a major mode: SPC h b f
-   See all keybindings available in a buffer, in order of precedence (describe-bindings): SPC h b b
-   Functions and variables related to search terms: M-x apropos <term[s]> / C-x c a
-   Find man page: C-x c m
-   Display elisp function documentation: C-h f fn RET


<a id="org713d313"></a>

# Basics

-   Doom equivalent of M-x: SPC :


<a id="orgfa0ecc3"></a>

# TODO map cider-repl-history-search-backward (in cider-repl-history-mode)?

