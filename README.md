# MCP Server Prototype

This repo provides a Python based prototype for a MCP server on NixOS. It relies on the [automatic shell activation](https://devenv.sh/automatic-shell-activation/) with devenv.sh and direnv.

## Prepare a Repository

Create a repository on github, clone the it into a local folder. Within the local folder prepare the devevlopment environment

```sh
devenv init
```

Update the greet variable and add the following packages in  devenv.nix

```nix
    # https://devenv.sh/basics/
    env.GREET = "MCP Prototype";

    # https://devenv.sh/packages/
    packages = with pkgs; [
    nodejs
    python313Packages.gradio
    ];
```

Add the python environment parameter to `devenv.nix`

```nix
# https://devenv.sh/languages/
  languages.python = {
    enable = true;
    package = pkgs.python313;
    venv.enable = true;
    venv.requirements = ''
      requests
      pip
      # torch
    '';
    # venv.packages = [ pkgs.python313Packages.gradio ];
    uv.enable = true;
  };
```

After saving the devenv configuration python and uv should be available.

```sh
python --version && uv --version
pip show gradio
```

Add the dotfiles to `.gitignore`

```sh
echo -e ".envrc" >> .gitignore
```

## Setup

Create a `~/.llm/config.json` file to configure your LLM and MCP servers

```sh{
  "systemPrompt": "You are an AI assistant helping a software engineer...",
  "llm": {
    "provider": "openai",
    "model": "gpt-4",
    "api_key": "your-openai-api-key",
    "temperature": 0.7,
    "base_url": "https://api.openai.com/v1"  // Optional, for OpenRouter or other providers
  },
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"],
      "requires_confirmation": ["fetch"],
      "enabled": true,  // Optional, defaults to true
      "exclude_tools": []  // Optional, list of tool names to exclude
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your-brave-api-key"
      },
      "requires_confirmation": ["brave_web_search"]
    },
    "youtube": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/adhikasp/mcp-youtube", "mcp-youtube"]
    }
  }
}
```

## Run the CLI

```sh
llm "What is the capital city of North Sumatra?"
```


## Links

### Sources

* [MCP CLI on GitHub](https://github.com/adhikasp/mcp-client-cli)

### Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
