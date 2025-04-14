# MCP CLI

This repo adapt the installation procedure for the Python MCP CLI to NixOS using the [automatic shell activation](https://devenv.sh/automatic-shell-activation/) with devenv.sh and direnv. 

## Development Environment

Go onto github and prepare a python repository, clone the rope into a local folder. Within the local folder prepare the devevlopment environment

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

Initiliaze and activate the virtual environment, leaving the virtual environment with `deactivate`.

```sh
uv init && uv venv
source .devenv/state/venv/bin/activate
```

Install dependencies

```sh
uv add mcp anthropic python-dotenv
```

## Setup the API Key

Next, we’ll need an Anthropic API key from the Anthropic Console, we create a .env file to store it:

```sh
touch .env
```

Add the key to the .env file:

```sh
ANTHROPIC_API_KEY=<your key here>
```

Add .env to your .gitignore:

```sh
echo ".env" >> .gitignore
```

## Create the Client

Create the client file

```sh
touch client.py
```

## Run the Client

To run the client with a Python based MCP server

```sh
uv run client.py path/to/server.py
```

## Links

### Sources

* [MCP CLI]([https://modelcontextprotocol.io/quickstart/client](https://github.com/chrishayuk/mcp-cli))

### Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
