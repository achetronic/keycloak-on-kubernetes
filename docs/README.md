# Full documentation

## Description

Full documentation about Keycloak on Kubernetes

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

    ```console
    pip install -r requirements.txt
    ```

3. Deploy in local environment

    ```console
    mkdocs serve
    ```

4. Build static site

    ```console
    mkdocs build --clean
    ```
