AGENTS Guide

Purpose
- Neovim configuration (Lua) based on kickstart.nvim + lazy.nvim living in ~/.config/nvim
- Standardizes how agentic coding agents operate here: commands, style rules, structure, and safety
- Audience: automated/code agents and human contributors

Repo Layout
- init.lua — primary configuration with lazy.nvim plugin specs and setup
- lua/kickstart/plugins/*.lua — optional plugin modules (lint, autopairs, debug, neo-tree, etc.)
- lua/custom/plugins/*.lua — local plugin specs; this repo includes lua/custom/plugins/init.lua and it is imported by init.lua
- lazy-lock.json — plugin lockfile managed by lazy.nvim (do not hand-edit)
- .stylua.toml — canonical Lua formatting rules for the repo
- .github/workflows/stylua.yml — example CI formatter check (gated to upstream repo)

System Requirements
- Neovim: latest stable or nightly
- External CLIs used by plugins/config:
  - git, make, unzip, a C compiler (for native plugin builds)
  - ripgrep and fd (used by Telescope)
  - Optional: Nerd Font; init.lua sets vim.g.have_nerd_font = true

Build / Sync / Health
- First run / after plugin changes:
  - nvim (boot) → :Lazy → press s to sync
  - Headless sync: nvim --headless "+Lazy! sync" +qa
- Update plugins: :Lazy update; clean unused: nvim --headless "+Lazy! clean" +qa
- Health checks:
  - All: nvim --headless "+checkhealth" +qa
  - Single provider: nvim --headless "+checkhealth telescope" +qa (replace provider name)
- Smoke test (load config, exit non-zero on startup errors): nvim --headless +qa

Format / Lint / Test Commands
- Lua format (enforced via .stylua.toml):
  - Format repo: stylua .
  - Check only: stylua --check .
  - Single file: stylua path/to/file.lua
- Markdown lint (wired via nvim-lint; CLI for manual/CI):
  - Repo: markdownlint .
  - Single file: markdownlint README.md
- Tests: no unit tests are defined for this config today.
  - Treat checkhealth as tests: nvim --headless "+checkhealth" +qa
  - Single "test" equivalent: nvim --headless "+checkhealth mason" +qa (or any provider)
  - If you add Lua tests with busted:
    - All: busted
    - Single file: busted spec/some_spec.lua
    - Single test by name (focus): busted -m _spec -t "name substring"

Debugging (DAP)
- Debugger plugins are present (nvim-dap + dap-ui + Go adapter). Useful keys:
  - <F5> continue; <F1> step into; <F2> step over; <F3> step out
  - <leader>b toggle breakpoint; <leader>B conditional breakpoint; <F7> toggle DAP UI

Plugin Management
- Where to add:
  1) Inside require('lazy').setup({...}) in init.lua; or
  2) In lua/custom/plugins/*.lua (preferred for local changes). init.lua already requires 'custom.plugins.init'
- Spec style (lazy.nvim):
  - Minimal: { 'author/repo', opts = {} }
  - Full: { 'author/repo', config = function() ... end }
  - Defer via `event`/`ft`/`keys`/`cmd`; use `cond` when external tools are required
- Lockfile policy: lazy-lock.json is source of truth; do not manually edit; re-sync to update

Keymaps & Discoverability
- Always provide descriptions for which-key/help: vim.keymap.set(mode, lhs, rhs, { desc = 'Sentence case description' })
- Follow existing leader groups ([S]earch, [T]oggle, Git [H]unk, etc.) defined in which-key config
- Honoring existing toggles: e.g., <leader>a toggles Aerial; '\\' reveals Neo-tree

Autocommands
- Create a unique augroup per feature to avoid duplicates:
  - local grp = vim.api.nvim_create_augroup('feature-name', { clear = true })
  - vim.api.nvim_create_autocmd({ 'BufWritePost' }, { group = grp, callback = function() ... end })
- Keep callbacks small; lift logic into locals when non-trivial

Diagnostics & UX
- Central behavior in init.lua via vim.diagnostic.config(...): no updates in insert, severity_sort, rounded floats
- Use vim.diagnostic.* helpers and keep default keymaps (plus provided <leader>q)

LSP Conventions
- Capabilities: extend via require('blink.cmp').get_lsp_capabilities()
- Setup pattern: define `servers` table, merge capabilities, then `vim.lsp.config(name, server)` and `vim.lsp.enable(name)`
- lua_ls: follow workspace/library/runtime settings in init.lua (do not override unless necessary)
- On attach: define buffer-local maps with helpful 'LSP: ...' descriptions; only enable features client supports (doc highlight, inlay hints)

Completion & Snippets
- Completion: saghen/blink.cmp preset 'default' expected; signature help enabled
- Snippets: LuaSnip with optional jsregexp build; friendly-snippets lazy-loaded; keep mappings aligned with preset

Treesitter
- This config opts into a small filetype list and starts Treesitter on FileType
- To add languages, extend the list passed to require('nvim-treesitter').install(...) in init.lua

Formatting on Save
- stevearc/conform.nvim:
  - lua → stylua; python → isort then black
  - lsp_format = 'fallback', timeout 500ms
  - Disabled for c/cpp by default; mirror pattern for other non-standardized languages

Code Style Guidelines
- Imports & modules:
  - Place require(...) at top for core deps; keep requires local inside plugin blocks when scope-specific
  - Guard optionals with pcall: local ok, t = pcall(require, 'telescope'); if ok then ... end
  - Return tables from modules under lua/**; prefer snake_case filenames and locals
- Formatting:
  - Follow .stylua.toml (column_width 160, single quotes preferred, no parens for simple calls)
  - Keep comments concise; avoid noise; wrap long prose to 160 cols
  - Default to ASCII in code/comments; UI plugins may use icons but prefer text fallbacks
- Types/annotations (EmmyLua):
  - Use ---@type for options tables; ---@param/---@return for public helpers
  - Use Neovim types when available (vim.Opt, plugin-specific)
- Naming:
  - Variables/locals lower_snake_case; augroups/autocmds kebab or snake (e.g., 'kickstart-lsp-attach')
  - Keymap descriptions in Sentence case with prefixes (e.g., 'LSP: Go to definition')
- Error handling & safety:
  - Avoid hard errors in plugin configs; prefer pcall, tool existence checks (vim.fn.executable('make') == 1)
  - Use vim.notify('message', vim.log.levels.WARN) for recoverable issues
  - Do not mutate user shell/environment; keep scope to Neovim

Git & Commit Hygiene
- Keep changes focused; mirror style/patterns in surrounding code; never commit secrets
- If you add lua/custom/plugins entries, ensure they are loaded (init.lua already requires custom/plugins/init.lua)
- Prefer one logical change per commit; include rationale in message body when non-obvious

Cursor / Copilot Rules
- No Cursor rules found (.cursor/rules/ or .cursorrules)
- No Copilot instructions found (.github/copilot-instructions.md)
- If such rules appear later, agents must surface and follow them

Quick Reference
- Sync plugins: nvim --headless "+Lazy! sync" +qa
- Update plugins: :Lazy update; Clean: nvim --headless "+Lazy! clean" +qa
- Health (all): nvim --headless "+checkhealth" +qa; Single: nvim --headless "+checkhealth telescope" +qa
- Format (write): stylua .; Check: stylua --check .; One file: stylua path/to/file.lua
- Lint markdown: markdownlint .; One file: markdownlint README.md
- Smoke test: nvim --headless +qa
