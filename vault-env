#!/bin/bash
set -e
#set -x
#printenv

# Apply all ENV from vault using waterfall style
# _ENV, _KEY are space seperated
export VAULT_ENV="development test"
export VAULT_ADDR=http://localhost:8200/v1
export VAULT_KEY=N3LrFRgtNqx251C7glizbu6oJfKfV/Lwc1MeSVoajW0=
export VAULT_TOKEN=80bfb376-c599-d28b-d6b3-5a3718b5baac



DISTRO_DEPS="curl jq"
ALPINE_DEPS=${DISTRO_DEPS}
DEBIAN_DEPS=${DISTRO_DEPS}
UNINSTALL_MSG="Add the following script if cleanup is required"
setupDebian () {
    apt-get update -y
    apt-get install -y --no-install-recommends ${DEBIAN_DEPS}
    rm -rf /var/cache/apk/*
    echo ${UNINSTALL_MSG}
    echo "apt-get purge -y --autoremove ${DEBIAN_DEPS}"
}

setupApline () {
    apk add --update ${ALPINE_DEPS}
    rm -rf /var/cache/apk/*
    echo ${UNINSTALL_MSG}
    echo "apk del ${DEBIAN_DEPS}"
}

setup () {
  if [ -f /etc/debian_version ]; then
    echo "Installing dependencies for Debian-based OSes..."
    installDebian
  elif [ -f /etc/alpine-release ]; then
    echo "Installing dependencies for Alpine-based OSes..."
    installAlpine
  elif [ -f /etc/redhat-release ]; then
    echo "RedHat-based OSes not supported"
  else
    # TODO add check if deps exits?
    echo "Sorry, I don't know how to bootstrap the required deps on your operating system!"
    exit 1
  fi
}

bootstrap () {
    # Check required ENV are set
    : "${VAULT_ENV:?ENV VAULT_ENV is required}"
    : "${VAULT_ADDR:?ENV VAULT_ADDR is required}"
    : "${VAULT_KEY:?ENV VAULT_KEY is required}"
    : "${VAULT_TOKEN:?ENV VAULT_TOKEN is required}"

    # TODO unseal
    for key in ${VAULT_KEY}; do
        curl -s \
            -H 'Content-Type: application/json' \
            -X PUT \
            -d "{\"key\":\"${key}\"}" \
            ${VAULT_ADDR}/sys/unseal > /dev/null
    done

    res_seal_status=$(curl -s ${VAULT_ADDR}/v1/sys/seal-status)
    sealed=$(echo $res_seal_status | jq -r '.sealed')
    if [ "$sealstatus" == "false" ]; then
        echo "Vault still sealed. $(echo $res_seal_status | jq -r '.progress') / $(echo $res_seal_status | jq -r '.t') Complete"
        exit 1;
    fi

    SECRET_PATH=/secret/
    for ENVIRONMENT in ${VAULT_ENV}; do
        SECRET_PATH=${SECRET_PATH}${ENVIRONMENT}/
        echo "SECRET_PATH=$SECRET_PATH"
        applyEnvVars ${SECRET_PATH}
    done

    # Sec clean up
    export VAULT_ENV=
    export VAULT_ADDR=
    export VAULT_KEY=
    export VAULT_TOKEN=
}

# bootstrap
vault_get () {
    curl \
        -H "X-Vault-Token: ${VAULT_TOKEN}" \
        -X GET \
        http://vault:8200$1
}

# bootstrap
applyGroupVars () {
    # $@ = /secret/${ENVIRONMENT}/${group}  # no trailing slash
    # Get group keys
    secrets=$(curl -s \
            -H "X-Vault-Token: ${VAULT_TOKEN}" \
            -X GET \
            ${VAULT_ADDR}${@} | jq -r '.data'
    )

    while IFS="=" read -r key value
    do
        export ${key}=$value
    done < <(echo $secrets | jq -r 'to_entries|map("\(.key)=\(.value)")|.[]')
}

# bootstrap
applyEnvVars ()  {
    # $@ = /secret/${ENVIRONMENT}/  # trailing slash
    keys=$(curl -s \
            -H "X-Vault-Token: ${VAULT_TOKEN}" \
            -X LIST \
            ${VAULT_ADDR}${@} \
            | jq -r '.data.keys|.[]'
    )
    for group in $keys; do
      if [[ "$group" != */ ]]; then
        echo "${@}${group}"
        applyGroupVars ${@}${group}
      fi
    done
}

# ROUTING
case $1 in
    setup)
        setup
        ;;
    bootstrap)
        bootstrap
        echo "=========="
        printenv
        ;;
    *)
        echo "Usage: {setup,bootstrap}"
        exit 1;
esac