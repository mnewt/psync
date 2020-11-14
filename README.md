# psync - Quick directory sync tool that respects .gitignore.

`psync` uses git and rsync to ensure that changes in the local directory are synchronized with the remote directory. Instead of copying all files, `psync` only copies the files that are part of the repository (and files that are marked as dirty). Files ignored by git are also ignored by `psync`.

It is intended to be run frequently to ensure files are backed up between commits.

```
USAGE: psync [-hv] [ VERB ] [ SOURCE ] [ DESTINATION ]

Where VERB is one of:

clone [ SOURCE ] DESTINATION

Clone a (presumably remote) source repository into the (presumably
local) destination.

sync [ SOURCE ] [ DESTINATION ]

Synchronize a (presumably local) source repository with a (presumably
remote) destination.

A psync_config file is searched for at the source. If it is found
then its settings are loaded. SOURCE and DESTINATION specified on the
command line override these.

The psync_config file is used to configure the sync command so that
you can run it without specifying parameters at the command line. Its
options include:

local='the local repository'
remote='the remote repository'

If VERB is omitted, sync is assumed.

Options are:

-h       Print this help message

-v       Turn on verbose messages
```

I use a hook in my editor to sync every time I save a file. If you use Emacs, maybe it's of use to you.

```emacs-lisp
(add-to-list 'auto-mode-alist '("psync_config\\'" . sh-mode))

(defvar-local psync-directory nil
  "Cached directory for `psync'.

It is always buffer local.")

(defun psync-maybe ()
  "If we find a `psync_config' file then run `psync'.

See: https://github.com/mnewt/psync"
  (interactive)
  (when-let ((default-directory (and (not (file-remote-p default-directory))
                                     (locate-dominating-file default-directory
                                                             "psync_config"))))
    (setq psync-directory default-directory)
    (if (= 0 (call-process-shell-command "psync"))
        (message "psync complete in directory %s." default-directory)
      (error "Synchronization with psync failed in directory: %s"
             default-directory))))

(defun psync-clone (local remote)
  "Clone a new repository for use with `psync' from LOCAL to REMOTE."
  (interactive (list (read-directory-name "Local directory: ")
                     (read-directory-name "Remote directory: ")))
  (async-shell-command (format "psync -v clone '%s' '%s'" local remote)))

(add-hook 'after-save-hook #'psync-maybe)
```
