
# Table of Contents

1.  [Evaluating Emacs-Lisp](#org55d1292)
2.  [Editing](#orgc59b2d0)
    1.  [Select at point](#org584d3fe)
    2.  [Project search ex commands](#org37169f6)
    3.  [Project search and replace](#org92f4722)
3.  [Org-mode](#org7fd0cb0)
4.  [Navigation](#org371f8a6)
5.  [Clojure](#org0e4a0c7)
6.  [Cider](#org936469a)
7.  [Files](#orge3e73f5)
8.  [Dired](#org564fe0b)
9.  [Buffers](#org63a6184)
10. [Windows](#orgea2f4b3)
11. [Environment](#org50ccfd8)
12. [Help](#orgf71635c)
13. [Basics](#orgef4f083)
14. [map cider-repl-history-search-backward (in cider-repl-history-mode)?](#orgd3218f3)

**Note:** *M-x and SPC : interchangeable in most or all cases*


<a id="org55d1292"></a>

# Evaluating Emacs-Lisp

-   Read 1 expr in minibuffer, eval, print val in echo area: SPC ;
-   Eval expr before point, print val [in echo area?]: SPC C-e


<a id="orgc59b2d0"></a>

# Editing


<a id="org584d3fe"></a>

## Select at point

-   Word: viw
-   Block (by indentation): vii
-   Block (by braces): vib
-   Sentence: vis
-   Paragraph: vip


<a id="org37169f6"></a>

## Project search ex commands

-   Search project (if !, include hidden files): :pg[!] <query>


<a id="org92f4722"></a>

## Project search and replace

-   Search project: SPC s p
-   Search another project: SPC s P
-   Search this directory: SPC s d
-   Search another directory: SPC s D


<a id="org7fd0cb0"></a>

# Org-mode

-   **Bold**, *italics*, <span class="underline">underline</span>, <del>strikethrough</del>
-   `code` and `verbatim` need to be the inner-most markers since their contents are interpreted literally.
-   Export:
    -   Add backend to `org-export-backends` if necessary.
    -   Export e.g. `M-x org-md-export-to-markdown`. The exported file should be in the same directory.


<a id="org371f8a6"></a>

# Navigation

-   Switch to buffer SPC <
-   Jump to file in project: SPC SPC
-   Open recent file (helm/workspace-mini): SPC ,
-   Find files in private config: SPC f p
-   Find files in project: SPC p f
-   helm-find-files: SPC .
-   Switch project: SPC p p
-   Toggle file explorer: SPC o p
-   Open recent file in any project: SPC f r
-   Open recent file in current project: SPC f R


<a id="org0e4a0c7"></a>

# Clojure

-   Evaluate expression: SPC m e e
-   Evaluate region: SPC m e r
-   Comment section for inline evaluation, e.g.:
    
        (comment
          (js/Date.)
          (:all-messages (make-mailbox)))


<a id="org936469a"></a>

# Cider

-   M-x cider-jack-in-clj: SPC m
-   M-x cider-jack-in-cljs: SPC m &ldquo;
-   Toggle between Clojure[Script] buffer and REPL: C-c C-z
-   Look up definition: M-.
-   Look up documentation: C-c C-d C-d (cider-mode-map), SPC m h d (clojure-mode-map)
-   Public symbols apropos: C-c C-d a
-   Documentation apropos: C-c C-d f
-   cider-repl-set-ns: SPC m n
-   cider-doc: SPC m h d
-   Show references to fn. at point: C-c C-? r


<a id="orge3e73f5"></a>

# Files

-   Move/Rename file: SPC f R


<a id="org564fe0b"></a>

# Dired

-   Toggle directory metadata display: shift-9
-   Move up a level: -
-   Select then delete a file/dir: d x
-   Sort by name/date in ascending order: o
-   Modify permissions: shift+M, then e.g. u+x,g-w
-   Select all directories: \* /
-   Unselect all: U
-   Select: m
-   Unselect: u
-   Toggle selection: t
-   Move selected: R


<a id="org63a6184"></a>

# Buffers

-   Switch buffers: SPC <


<a id="orgea2f4b3"></a>

# Windows

-   Open new window in vertical split: C-x 3
-   Open new frame: SPC o f


<a id="org50ccfd8"></a>

# Environment

-   Show variable value: C-h v
-   Show current major mode: M-: / C-h v major-mode
-   List and display docs of current major and minor modes: C-h m
-   Effect changes in .doom.d/init.el:

$ doom sync
In Emacs: M-x doom/reload i.e. SPC h r r


<a id="orgf71635c"></a>

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


<a id="orgef4f083"></a>

# Basics

-   Doom equivalent of M-x: SPC :


<a id="orgd3218f3"></a>

# TODO map cider-repl-history-search-backward (in cider-repl-history-mode)?

