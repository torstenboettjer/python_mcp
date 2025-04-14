# MCP CLI

This repo adapt the installation procedure for the Python MCP CLI to NixOS using the [automatic shell activation](https://devenv.sh/automatic-shell-activation/) with devenv.sh and direnv. 

## Prepare a Repository

Create a repository on github, clone the it into a local folder. Within the local folder prepare the devevlopment environment

```sh
devenv init
```

Update the greet variable and add the following packages in  devenv.nix 

```sh
  # https://devenv.sh/basics/
  env.GREET = "MCP CLI";

  # https://devenv.sh/packages/
  packages = [
    pkgs.git
    pkgs.nodejs
  ];
```

Add the python environment parameter to `devenv.nix`

```nix
  # Python environment
  languages.python = {
    enable = true;
    # https://devenv.sh/languages/python/
    # packages = [ pkgs.python39Packages.pip ];
    venv.enable = true;
    venv.requirements = ''
      requests
      pip
      # torch
    '';
    # venv.packages = [ pkgs.python39Packages.pip ];
    # venv.packages = [ pkgs.python39Packages.pip ];
    uv.enable = true;
  };
```

After saving the devenv configuration python and uv should be available.

```sh
python --version && uv --version
```

Add the dotfiles to `.gitignore`

```sh
echo -e ".direnv\n.envrc" >> .gitignore
```

## Setup

Install the client via pip

```sh
pip install mcp-client-cli
```

Create a ~/.llm/config.json file to configure your LLM and MCP servers

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

## Run the CLI using UV

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
