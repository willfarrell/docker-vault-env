# docker vault-env
script to bootstrap ENV from vault



## Sample Use
### Dockerfile
```Dockerfile
...
RUN cd /usr/bin \
    && wget https://raw.githubusercontent.com/willfarrell/docker-vault-env/master/vault-env \
    && chmod a+x vault-env\
    && vault-env install
...
```

### docker-entrypoint.sh
```bash
#!/bin/bash

vault-env bootstrap

exec "${@}"
```

### docker-compose.yml
```yml
version: "2"

services:
  service:
    ...
    links:
     - vault
    env_file:
     - vault-service.env
  
  vault:
    image: mycompany/vault:latest
    expose:
     - "8022"
    restart: always
    read_only: false
    security_opt:
     - "no-new-privileges"
    volumes:
     - "./log/vault:/var/log:rw"
```

### vault-service.env
```bash
VAULT_ENV="development ecom"
VAULT_ADDR=
VAULT_KEY=""
VAULT_TOKEN=
```

## Vault Setup

### Policies


### Docker Container Accounts


## TODO
- [ ] Docs: Policy setup
- [ ] Docs: Auth setup