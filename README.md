```
; Add Codeium to load-path
(add-to-list 'load-path "~/.emacs.d/codeium.el/codeium.el")
(load "~/.emacs.d/codeium.el/codeium.el")
(add-to-list 'completion-at-point-functions #'codeium-completion-at-point)
;; Disable toolbar and scrollbar
(tool-bar-mode -1)
(scroll-bar-mode -1)
(delete-selection-mode 1)

;; Enable projectile mode
(require 'projectile)
(projectile-mode +1)
;; Enable Vertico mode
(vertico-mode 1)

(use-package global-company-mode
  :hook (after-init . global-company-mode))

(use-package lsp-mode
  :commands (lsp lsp-deferred)
  :init
  (setq lsp-headerline-breadcrumb-enable nil)
  (setq lsp-keymap-prefix "C-c l")  ;; Or 'C-l', 's-l'
  :config
  (lsp-enable-which-key-integration t)
  )

(use-package prog-mode
  :hook (prog-mode . (lambda ()
                       (setq tab-width 4)
                       (setq indent-tabs-mode nil))))

(use-package company
  :after lsp-mode
  :hook (prog-mode . company-mode)
  :bind (:map company-active-map
         ("<tab>" . company-complete-selection))
        (:map lsp-mode-map
         ("<tab>" . company-indent-or-complete-common))
  :custom
  (company-minimum-prefix-length 2)
  (company-idle-delay 0.25))

(use-package company-box
  :hook (company-mode . company-box-mode))

;; Display line numbers globally
(global-display-line-numbers-mode 1)

;; Centaur tabs config

(require 'centaur-tabs)

(use-package centaur-tabs
  :defer t
  :config
  (centaur-tabs-mode t)
  :hook
  (eshell-mode . centaur-tabs-local-mode)
  (term-mode . centaur-tabs-local-mode))

;; Set Tab Height
(setq centaur-tabs-height 36)
;; Make unselected tabs greyed out
(setq centaur-tabs-gray-out-icons 'buffer)
;; Modified marker in the tab
(setq centaur-tabs-set-modified-marker t)

;; Use `tab-bar-mode` keybindings with Centaur Tabs
(global-set-key (kbd "C-x t 2") 'centaur-tabs--create-new-tab)   ;; Same as `tab-bar-new-tab`
(global-set-key (kbd "C-x t 0") 'centaur-tabs--kill-this-buffer-dont-ask) ;; Same as `tab-bar-close-tab`
(global-set-key (kbd "C-<tab>") 'centaur-tabs-ace-jump)
(global-set-key (kbd "<backtab>") 'centaur-tabs-counsel-switch-group)
(global-set-key (kbd "C-x t 3") 'centaur-tabs-kill-other-buffers-in-current-group)
(global-set-key (kbd "C-x t 4") 'centaur-tabs-kill-unmodified-buffers-in-current-group)

;; Group Centaur Tabs by projectile project
(centaur-tabs-group-by-projectile-project)

;; Find all function usages
(global-set-key (kbd "<f12>") 'dumb-jump-go)
(global-set-key (kbd "M-<f12>") 'dumb-jump-quick-look)
(global-set-key (kbd "C-<f12>") 'dumb-jump-back)

;; Multiple cursors key bindings
(global-set-key (kbd "C-S-<down>") 'mc/mark-next-like-this)
(global-set-key (kbd "C-S-<up>") 'mc/mark-previous-like-this)

;; Custom copy and paste binds for pasting whole lines
(defun my-copy-line-or-region ()
  "Copy the entire line if no region is selected."
  (interactive)
  (if (use-region-p)
      (copy-region-as-kill (region-beginning) (region-end))
    (kill-ring-save (line-beginning-position) (line-end-position)))
  (message "Line or region copied"))

(defun my-paste-line-above ()
  "Paste the last copied line above the current line."
  (interactive)
  (let ((line (car kill-ring)))
    (save-excursion
      (beginning-of-line)
      (newline)
      (previous-line)
      (insert line)))
  (message "Pasted line above"))

;; Bind the custom functions to keys
(global-set-key (kbd "C-c c") 'my-copy-line-or-region)
(global-set-key (kbd "C-c y") 'my-paste-line-above)

;; Move line above/below binds

(defun move-line-up ()
  "Move the current line up."
  (interactive)
  (let ((col (current-column)))
    (beginning-of-line)
    (transpose-lines 1)
    (forward-line -2)
    (move-to-column col)))

(defun move-line-down ()
  "Move the current line down."
  (interactive)
  (let ((col (current-column)))
    (forward-line 1)
    (transpose-lines 1)
    (forward-line -1)
    (move-to-column col)))

;; Bind custom line-moving functions to M-<up> and M-<down>
(global-set-key (kbd "M-<up>") 'move-line-up)
(global-set-key (kbd "M-<down>") 'move-line-down)

;; New line below key binding
(defun my-newline-below ()
  "Insert a newline below the current line."
  (interactive)
  (end-of-line)
  (newline-and-indent))

(global-set-key (kbd "C-<return>") 'my-newline-below)

;; New line above key binding
(defun my-newline-above ()
  "Insert a newline above the current line."
  (interactive)
  (beginning-of-line)
  (newline)
  (forward-line -1)
  (indent-for-tab-command))

(global-set-key (kbd "C-S-<return>") 'my-newline-above)

;; Bind Projectile commands to keys
(defun my-projectile-bindings ()
  "Set up custom key bindings for Projectile commands."
  (interactive)
  ;; Basic Projectile commands
  (global-set-key (kbd "C-c p p") 'projectile-switch-project) ; Switch to another project
  (global-set-key (kbd "C-c p f") 'projectile-find-file)        ; Find file in project
  (global-set-key (kbd "C-c p s") 'projectile-find-file-in-directory) ; Find file in directory
  (global-set-key (kbd "C-c p r") 'projectile-recentf)          ; Find file by regex
  (global-set-key (kbd "C-c p c") 'projectile-run-async-shell-command-in-root) ; Run project command
  (global-set-key (kbd "C-c p i") 'projectile-add-known-project) ; Add file to project
  (global-set-key (kbd "C-c p d") 'projectile-remove-known-project) ; Remove file from project
  (global-set-key (kbd "C-c p a") 'projectile-add-known-project)

  ;; Advanced Projectile commands
  (global-set-key (kbd "C-c p t") 'projectile-find-tag)         ; Open tags file
  (global-set-key (kbd "C-c p e") 'projectile-find-file)         ; Find environment file
  (global-set-key (kbd "C-c p k") 'projectile-kill-buffers)      ; Kill project buffers
)

;; Apply the custom key bindings
(my-projectile-bindings)

(setq projectile-project-root-files
      '(".git" "Makefile" "package.json" "Cargo.toml" "build.gradle"))

;; Enable electric-indent mode
(electric-indent-mode 1)

(use-package smartparens
  :defer t
  :hook (prog-mode . smartparens-mode)
  :config
  (require 'smartparens-config)
  (show-smartparens-global-mode t))

;; Expand region key binding
(require 'expand-region)
(global-set-key (kbd "C-<") 'er/expand-region)

;; Configure Corfu
(use-package corfu
  :init
  (global-corfu-mode))

;; Configure package manager
(require 'package)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/") t)
(package-initialize)

;; Configure Treemacs
(use-package treemacs
  :defer t
  :init
  (with-eval-after-load 'winum
    (define-key winum-keymap (kbd "M-0") #'treemacs-select-window))
  :config
  (progn
    (setq treemacs-collapse-dirs                   (if treemacs-python-executable 3 0)
          treemacs-width                           25
          treemacs-workspace-switch-cleanup        nil)

    (treemacs-follow-mode t)
    (treemacs-filewatch-mode t)
    (treemacs-fringe-indicator-mode 'always)
    (when treemacs-python-executable
      (treemacs-git-commit-diff-mode t))

    (pcase (cons (not (null (executable-find "git")))
                 (not (null treemacs-python-executable)))
      (`(t . t)
       (treemacs-git-mode 'deferred))
      (`(t . _)
       (treemacs-git-mode 'simple)))

    (treemacs-hide-gitignored-files-mode nil))
  :bind
  (:map global-map
        ("M-0"       . treemacs-select-window)
        ("C-x t 1"   . treemacs-delete-other-windows)
        ("C-x t t"   . treemacs)
        ("C-x t d"   . treemacs-select-directory)
        ("C-x t r"   . treemacs-refresh)
        ("C-x t B"   . treemacs-bookmark)
        ("C-x t C-t" . treemacs-find-file)
	("C-x 4 t"   . treemacs-add-and-display-current-project-exclusively) ;; Open Treemacs in a new window
        ("C-x t M-t" . treemacs-find-tag)))

(defun swiper-search-selection ()
  "Use swiper to search for the selected text."
  (interactive)
  (if (use-region-p)
      (let ((search-term (buffer-substring-no-properties (region-beginning) (region-end))))
	(deactivate-mark)
        (swiper search-term))
    (swiper)))

(use-package swiper
  :ensure t
  :bind (("C-s" . swiper-search-selection)))

(use-package treemacs-projectile
  :after (treemacs projectile))

;; Disable backup files
(setq make-backup-files nil)

(use-package lsp-ui
  :after lsp-mode
  :commands lsp-ui-mode)

(setq lsp-ui-doc-position 'bottom)

(setq lsp-ui-sideline-enable nil)
(setq lsp-ui-sideline-show-hover nil)

;; Enable Golden Ratio mode
(require 'golden-ratio)
(golden-ratio-mode 1)

(custom-set-variables
 ;; custom-set-variables was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 '(codeium/metadata/api_key "d9b0f82c-5cd5-4620-8a8c-7ac88d2ba605")
 '(custom-enabled-themes '(kaolin-valley-light))
 '(custom-safe-themes
   '("5a00018936fa1df1cd9d54bee02c8a64eafac941453ab48394e2ec2c498b834a" "06ed754b259cb54c30c658502f843937ff19f8b53597ac28577ec33bb084fa52" "a131602c676b904a5509fff82649a639061bf948a5205327e0f5d1559e04f5ed" "2ce76d65a813fae8cfee5c207f46f2a256bac69dacbb096051a7a8651aa252b0" "3c7a784b90f7abebb213869a21e84da462c26a1fda7e5bd0ffebf6ba12dbd041" "4363ac3323e57147141341a629a19f1398ea4c0b25c79a6661f20ffc44fdd2cb" \... "default"))
 '(load-theme 'kaolin-valley-light t)
 '(package-selected-packages
   '(kaolin-themes js2-refactor tide dashboard page-break-lines format-all toggle-term typescript-mode all-the-icons-ivy-rich all-the-icons-dired eshell-git-prompt eshell-prompt-extras eshell-did-you-mean eshell-command-not-found counsel-projectile night-owl-theme jedi jedi-core centaur-tabs lsp-ivy company-go js2-mode json-mode dap-mode corfu ag dumb-jump ripgrep smartparens expand-region \...)))

(custom-set-faces
 ;; custom-set-faces was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 )

;; Custom language servers

;; GO
(setq lsp-go-analyses '((shadow . t)
 (simplifycompositelit . :json-false)))

;; Set up before-save hooks to format buffer and add/delete imports.
;; Make sure you don't have other gofmt/goimports hooks enabled.
(defun lsp-go-install-save-hooks ()
  (add-hook 'before-save-hook #'lsp-format-buffer t t)
  (add-hook 'before-save-hook #'lsp-organize-imports t t))
(add-hook 'go-mode-hook #'lsp-go-install-save-hooks)

(use-package typescript-mode
  :mode "\\.ts\\'"
  :hook (typescript-mode . lsp-deferred)
  :config
  (setq typescript-indent-level 2))

;; Terminal mode hook
(dolist (mode '(eshell-mode-hook
                term-mode-hook
                org-mode-hook))
  (add-hook mode (lambda ()
                   (display-line-numbers-mode 0)
                   (centaur-tabs-local-mode 1))))

;; Night Owl Theme styling
(defun night-owl/ivy-format-function-line (cands)
  "Transform CANDS into a string for minibuffer."
  (let ((str (ivy-format-function-line cands)))
    (font-lock-append-text-property 0 (length str) 'face 'ivy-not-current str)
    str))

(setq ivy-format-function #'night-owl/ivy-format-function-line)

(which-key-setup-side-window-right)
(which-key-mode 1)
(use-package lsp-ivy)

;; Bind redo to C-.
(global-set-key (kbd "C-.") 'undo-redo)

(use-package toggle-term
  :bind (("M-o f" . toggle-term-find)
         ("M-o t" . toggle-term-term)
         ("M-o v" . toggle-term-vterm)
         ("M-o a" . toggle-term-eat)
         ("M-o s" . toggle-term-shell)
         ("M-o e" . toggle-term-eshell)
         ("M-o i" . toggle-term-ielm)
         ("M-o o" . toggle-term-toggle))
  :config
    (setq toggle-term-size 20)
    (setq toggle-term-switch-upon-toggle t))

(when (display-graphic-p)
  (require 'all-the-icons))
;; or
(use-package all-the-icons
  :if (display-graphic-p))

;; Configure Emacs Dashboard
(use-package dashboard
  :ensure t
  :config
  (dashboard-setup-startup-hook)
  (setq dashboard-startup-banner 'logo)
  (setq dashboard-items '((recents . 5)
                          (projects . 5)))
  (setq initial-buffer-choice (lambda () (get-buffer "*dashboard*")))
  ;; Adjusting some visual aspects of the dashboard
  (setq dashboard-center-content t)
  (setq dashboard-set-heading-icons t)
  (setq dashboard-set-file-icons t))

(setq gc-cons-threshold (* 100 1000 1000))  ;; 100MB during startup
(add-hook 'emacs-startup-hook
          (lambda () (setq gc-cons-threshold (* 20 1000 1000))))  ;; 20MB after startup

(defun setup-tide-mode ()
  (interactive)
  (tide-setup)
  (flycheck-mode +1)
  (setq flycheck-check-syntax-automatically '(save mode-enabled))
  (eldoc-mode +1)
  (tide-hl-identifier-mode +1)
  ;; company is an optional dependency. You have to
  ;; install it separately via package-install
  ;; `M-x package-install [ret] company`
  (company-mode +1))

;; aligns annotation to the right hand side
(setq company-tooltip-align-annotations t)

;; formats the buffer before saving
(add-hook 'before-save-hook 'tide-format-before-save)

;; if you use typescript-mode
(add-hook 'typescript-mode-hook #'setup-tide-mode)
;; if you use treesitter based typescript-ts-mode (emacs 29+)
(add-hook 'typescript-ts-mode-hook #'setup-tide-mode)


```
