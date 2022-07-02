# Keycloak documentation

## Description
General documentation about the Keycloak for the internal meta-documentation

This repository is here as a way to add a global overview of the systems involved into authentication in
the [meta documentation](https://gitlab.s73cloud.com:29999/Infrastructure/mkdocs) repository of the company, 
which can be [accessed here](https://docs.s73cloud.com/)

## Requirements

 * Python >= 3.6
 * Virtualenv

## How to launch

1. Create virtualenv executing the following in the console:

```console
python3 -m venv ./.venv
source .venv/bin/activate
```

2. Install dependencies inside the virtualenv

```
pip install -r requirements.txt
```

3. Deploy in local environment

```
mkdocs serve
```

4. Build static site

```
mkdocs build --clean
```
