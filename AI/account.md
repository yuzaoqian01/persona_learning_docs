```json
email:yiuawsvqff2691@hotmail.com
key:sk-ctVZP0q179K7iTBzaPzilXlkpKZ0vZ3B5KV7t3FGvnZZdvE9
domain:https://api.codexcn.com/query
```

```toml
notify = ["/Users/zql/.codex/computer-use/Codex Computer Use.app/Contents/SharedSupport/SkyComputerUseClient.app/Contents/MacOS/SkyComputerUseClient", "turn-ended"]
service_tier = "priority"
model = "gpt-5.5"
model_reasoning_effort = "medium"

[marketplaces.openai-bundled]
last_updated = "2026-06-23T02:22:40Z"
source_type = "local"
source = "/Users/zql/.codex/.tmp/bundled-marketplaces/openai-bundled"

[marketplaces.openai-primary-runtime]
last_updated = "2026-06-23T03:04:12Z"
source_type = "local"
source = "/Users/zql/.cache/codex-runtimes/codex-primary-runtime/plugins/openai-primary-runtime"

[plugins."documents@openai-primary-runtime"]
enabled = true

[plugins."spreadsheets@openai-primary-runtime"]
enabled = true

[plugins."presentations@openai-primary-runtime"]
enabled = true

[plugins."figma@openai-curated"]
enabled = true

[plugins."browser@openai-bundled"]
enabled = true

[plugins."pdf@openai-primary-runtime"]
enabled = true

[plugins."template-creator@openai-primary-runtime"]
enabled = true

[features]
js_repl = false

[mcp_servers.node_repl]
args = []
command = "/Applications/Codex.app/Contents/Resources/cua_node/bin/node_repl"
startup_timeout_sec = 120

[mcp_servers.node_repl.env]
NODE_REPL_NATIVE_PIPE_CONNECT_TIMEOUT_MS = "1000"
NODE_REPL_NODE_MODULE_DIRS = "/Applications/Codex.app/Contents/Resources/cua_node/lib/node_modules"
NODE_REPL_NODE_PATH = "/Applications/Codex.app/Contents/Resources/cua_node/bin/node"
NODE_REPL_TRUSTED_CODE_PATHS = "/Users/zql/.codex"
CODEX_HOME = "/Users/zql/.codex"
NODE_REPL_TRUSTED_BROWSER_CLIENT_SHA256S = "5ee3754ef68f803b9cebf40e30f94ea22aaf446de1a0a24b109180fa2ae0bd3e"
BROWSER_USE_AVAILABLE_BACKENDS = "chrome,iab"
NODE_REPL_INSTRUCTIONS_USE_CASE_BROWSER = "Control the in-app browser in conjunction with the Browser Plugin."
NODE_REPL_INSTRUCTIONS_USE_CASE_CHROME = "Control the Chrome browser in conjunction with the Chrome Plugin. Prefer this method of controlling Chrome over alternatives (such as Computer Use) unless the user explicitly mentions an alternative."
BROWSER_USE_CODEX_APP_BUILD_FLAVOR = "prod"
BROWSER_USE_CODEX_APP_VERSION = "26.616.71553"
CODEX_CLI_PATH = "/Applications/Codex.app/Contents/Resources/codex"

[desktop]
conversationDetailMode = "STEPS_COMMANDS"
ambient-suggestions-enabled = false

[desktop.open-in-target-preferences]
global = "vscode"

[desktop.open-in-target-preferences.perPath]
"/Users/zql/Works/Project/2016/06/finx_app" = "vscode"

[projects."/Users/zql/Works/Project/2016/06/finx_app"]
trust_level = "trusted"

[projects."/Users/zql/Works/Project/works_project/12/ftg-invest"]
trust_level = "trusted"

```

