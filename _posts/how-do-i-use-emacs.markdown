---
title: "How do i use Emacs"
date: 2014-12-29 10:41:58 +0800
categories: [tools]
tags: [emacs, english]
---

**Emacs** is a powerful programme editor.

In my job, I used **js2-mode** for NodeJS RESTful projects and used **markdown-mode** for its documents.
Before i used Emacs, I tried VIM, Another programme editor. At last i did not insisted on using it, Beause it's ugly customized, Just **config file** Not **development script**.

I shared my [.emacs.d](https://github.com/vietor/emacs.d) on github, it contained some script from [purcell's scripts](https://github.com/purcell/emacs.d).

### Customized Environment

I include some **script** from [Melpa](http://melpa.org), and customized them for **myself**.  
The script example in my **.emacs.d**.

Example:  
Diable markdown-mode's auto-indent and trailing whitespace clean  
*init-markdown.el*
``` lisp
(add-hook 'markdown-mode-hook
          (lambda ()
            (setq retain-trailing-whitespace t)
            (set (make-local-variable 'electric-indent-mode) nil)
            (define-key markdown-mode-map (kbd "RET") 'newline)))
```
Where is the variable 'retain-trailing-whitespace' defined?  
*init-extend.el*
``` lisp
(defvar retain-trailing-whitespace nil)
(make-variable-buffer-local 'retain-trailing-whitespace)

(defadvice delete-trailing-whitespace (around retain activate)
  (unless retain-trailing-whitespace
    ad-do-it))
```
Where is the function 'delete-trailing-whitespace' used?  
*init-extend.el*
``` lisp
(defvar buffer-indent-function nil)
(make-variable-buffer-local 'buffer-indent-function)

(defun indent-current-buffer ()
  (interactive)
  (if (functionp buffer-indent-function)
      (funcall buffer-indent-function)
    (save-excursion
      (delete-trailing-whitespace)
      (indent-region (point-min) (point-max)))))
```
Is the function 'indent-current-buffer' bind any key?  
*init-keybind.el*
``` lisp
(global-set-key (kbd "<f12>")   'indent-current-buffer)
```
**Yes! When i want, maybe i script it. LOL**

### Multiple instances for multiple project

In my script, I add a command line parameter '-project' for specified directory as project.  
In project directory, It has a '.emacs.p' directory for store specified files, like: session, desktop etc.

Command line usage
``` bash
emacs -project ~/work/test
```
I created a named 'OpenInEmnacsAsProject' script for Natilus's right menu in Linux.  
I modified registory for right menu in Windows.

### Writing something

A great programme editor has two features: awesome to use, awesome to customize.

Using Emacs for coding.  
Using Emacs for document.  
Using Emacs for bloging.  
etc.

