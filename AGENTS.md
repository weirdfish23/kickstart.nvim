AGENTS Guide

Purpose
- This repository is a Neovim configuration (Lua) based on kickstart.nvim and lazy.nvim.
- This guide standardizes how agentic coding agents operate here: commands, style rules, structure, and safety.
- Audience: automated/code agents and contributors working inside ~/.config/nvim.

Repo Layout
- init.lua — primary configuration with lazy.nvim plugin specs and setup
- lua/kickstart/plugins/*.lua — optional plugin modules (lint, autopairs, etc.)
- lua/custom/plugins/*.lua — your local plugin specs (currently empty). Import is commented in init.lua
- lazy-lock.json — plugin lockfile managed by lazy.nvim
- .stylua.toml — canonical Lua formatting rules
- .github/workflows/stylua.yml — example CI formatter check (gated to upstream repo)

System Requirements
- Neovim: latest stable or nightly
- External CLI tools used by plugins and config:
  - git, make, unzip, a C compiler (for native plugin builds)
  - ripgrep and fd (used by Telescope)
  - Optional: Nerd Font; set vim.g.have_nerd_font = true in init.lua if installed

Build / Sync / Health
- Bootstrap/install plugins (first run or after changes):
  - Launch Neovim: nvim
  - Open lazy UI: :Lazy, then press s to sync (or run headless command below)
- Headless plugin sync (CI-friendly):
  - nvim --headless "+Lazy! sync" +qa
- Update plugins interactively: :Lazy update
- Health checks:
  - Full: nvim --headless "+checkhealth" +qa
  - Single section (e.g., telescope): nvim --headless "+checkhealth telescope" +qa
- Smoke test (load config, exit non-zero on startup errors):
  - nvim --headless +qa

Format / Lint / Test Commands
- Lua formatting (canonical; enforced by .stylua.toml):
  - Format repo: stylua .
  - Check only: stylua --check .
  - Single file check: stylua --check path/to/file.lua
- Markdown lint (nvim-lint runs in-editor; CLI shown for CI/manual):
  - Repo: markdownlint .
  - Single file: markdownlint README.md
  - Note: lua/kickstart/plugins/lint.lua configures nvim-lint to run markdownlint on markdown buffers automatically.
- Tests: there are no unit tests defined for this config today.
  - Health as tests: nvim --headless "+checkhealth" +qa (treat failures as test failures)
  - Single "test" pattern: run checkhealth for exactly one provider, e.g. telescope or mason (see above)
  - If you add Lua unit tests (e.g., with busted), typical commands:
    - All: busted
    - Single file: busted spec/some_spec.lua
    - Single test by name (pattern): busted -m _spec -t "name substring"

CI Notes
- .github/workflows/stylua.yml runs a Stylua check but only on the upstream repo due to:
  - if: github.repository == 'nvim-lua/kickstart.nvim'
- For forks/local work, run stylua locally as shown above.

Adding Plugins
- Preferred locations:
  1) Add to init.lua inside require('lazy').setup({...}) alongside existing specs; or
  2) Place plugin specs in lua/custom/plugins/*.lua and enable that import by uncommenting
     the line in init.lua: { import = 'custom.plugins' }
- Use lazy.nvim style:
  - Minimal spec: { 'author/repo', opts = {} }
  - Full control: { 'author/repo', config = function() ... end }
  - Defer/lazy-load via event/ft/keys/cmd; use cond when external tools are required

Keymaps and Discoverability
- Define keymaps with descriptions for which-key and help:
  - vim.keymap.set(mode, lhs, rhs, { desc = 'Meaningful description' })
- Group leader mappings consistently; see which-key setup in init.lua for examples of [S]earch, [T]oggle groups
- Avoid hard-coding terminal-ambiguous chords; prefer defaults mapped in config

Autocommands
- Always create an augroup per feature to avoid duplicates:
  - local grp = vim.api.nvim_create_augroup('feature-name', { clear = true })
  - vim.api.nvim_create_autocmd({ 'BufWritePost' }, { group = grp, callback = function() ... end })
- Keep callbacks small; lift logic into a local function where non-trivial

Diagnostics & UX
- Central diagnostic behavior is configured in init.lua via vim.diagnostic.config(...)
- Respect existing defaults (no updates in insert, severity_sort, rounded floats)
- Use vim.diagnostic.* helpers for navigation and lists; do not rebind conflicting defaults

LSP Conventions
- Capabilities: extend server capabilities using require('blink.cmp').get_lsp_capabilities()
- Server setup pattern (see init.lua):
  - Define servers table, then iterate: server.capabilities = vim.tbl_deep_extend('force', {}, caps, server.capabilities or {})
  - Configure lua_ls with workspace/library settings as shown
- On attach:
  - Define buffer-local keymaps via a helper map() that adds { buffer = event.buf, desc = 'LSP: ...' }
  - Implement document highlight and inlay hints toggles only if client supports the method

Completion & Snippets
- Completion via saghen/blink.cmp; preset 'default' is expected
- Snippets via LuaSnip; install jsregexp when possible; keep mappings aligned with blink preset

Treesitter
- This config opts into a focused set of filetypes and starts treesitter on FileType
- When adding languages, prefer adding to the list used by require('nvim-treesitter').install(...)

Formatting on Save
- stevearc/conform.nvim is configured:
  - Uses stylua for Lua; python uses isort then black
  - lsp_format = 'fallback' with 500ms timeout
  - Disabled for c/cpp by default; follow existing pattern for other non-standardized languages

Imports and Modules
- Use require(...) at top for core dependencies; keep requires local to config blocks when scope-specific
- For optional modules/extensions, guard with pcall to avoid startup errors:
  - local ok, mod = pcall(require, 'telescope') if ok then ... end
- Return tables from modules placed under lua/**; prefer snake_case filenames

Naming Conventions
- Variables and locals: lower_snake_case (e.g., lazy_path, statusline)
- Augroups and autocommands: kebab or snake, consistent and descriptive (e.g., 'kickstart-lsp-attach')
- Keymap descriptions: Sentence case with prefixes where appropriate (e.g., 'LSP: Go to Definition')

Type Annotations (EmmyLua)
- Use Neovim/LSP-friendly annotations to aid tooling:
  - ---@type vim.Opt or specific plugin configs (see init.lua examples)
  - ---@param, ---@return for public/local helpers where non-trivial

Error Handling & Safety
- Avoid throwing hard errors in plugin configs; prefer:
  - pcall(...) for optional dependencies
  - Checks for external tools: if vim.fn.executable('make') == 1 then ... end
  - vim.notify('message', vim.log.levels.WARN) for recoverable issues
- Do not modify user environment or shell state; keep changes scoped to Neovim

Styling Rules (from .stylua.toml)
- column_width = 160 (wrap prose/comments accordingly)
- line_endings = Unix
- indent_type = Spaces; indent_width = 2
- quote_style = AutoPreferSingle (default to single quotes when possible)
- call_parentheses = None (omit parens for simple calls when allowed)
- collapse_simple_statement = Always

Git & Commit Hygiene
- Keep changes focused; mirror existing patterns; do not commit secrets
- If adding lua/custom/plugins, consider enabling its import in init.lua in the same change for atomicity

Cursor / Copilot Rules
- No Cursor rules (.cursor/rules/ or .cursorrules) found
- No Copilot instructions (.github/copilot-instructions.md) found
- If added later, agents must surface those rules in reviews and follow them

Contributing Tips for Agents
- Respect existing lazy.nvim structure and events; prefer event-based loading to reduce startup time
- Always include desc in keymaps for which-key visibility
- Use augroups for any autocmds you create; keep group names unique
- Keep user toggles consistent with existing <leader>t* patterns
- Update this AGENTS.md if you change any core workflows (format, lint, health, plugin import path)

Quick Reference
- Sync plugins: nvim --headless "+Lazy! sync" +qa
- Health (all): nvim --headless "+checkhealth" +qa
- Health (one): nvim --headless "+checkhealth telescope" +qa
- Format (write): stylua .
- Format (check): stylua --check .
- Lint markdown (all): markdownlint .
- Lint markdown (one): markdownlint README.md
- Smoke test config: nvim --headless +qa
