# MCP CLI

This repo adapt the installation procedure for the Python MCP CLI to NixOS using the [automatic shell activation](https://devenv.sh/automatic-shell-activation/) with devenv.sh and direnv. 

## Installation

Clone the repository

```sh
git clone https://github.com/chrishayuk/mcp-cli
cd mcp-cli
```

Initialise devenv

```sh
devenv init
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

Install dependencies

```sh
uv sync --reinstall
```

## Run the CLI using UV

```sh
uv run mcp-cli --help
```


## Links

### Sources

* [MCP CLI](https://modelcontextprotocol.io/quickstart/client](https://github.com/chrishayuk/mcp-cli)

### Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
