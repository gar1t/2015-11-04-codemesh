<style>
.reveal .ct-label {color:white; font-size:75%;}
</style>

### On the Functional Tao of Bash

---

```bash
find -type f | xargs -I F basename F
```

---

```bash
map() {
    local fun=$1
    local -a vals=("${@:2}")
    for val in "${vals[@]}"; do
        eval "$fun $val"
    done
}

incr() {
    echo $(($1 + 1))
}

map incr 1 2 3 4
```

---

## Bash is not a functional language

---

## A Real Life Story

- bash script
- 140 lines of code
- named `ci.sh`
- ci = "continuous integration"

---

## Why Bash

- It's everywhere
- Mostly portable across posix systems
- Zero impedance mismatch with shell scripting

---

<img src="installing-nginx-1.png">

---

<img src="installing-nginx-2.png">

---

## Back to `ci.sh`

---

<img src="any-language.png">

---

#### Bash

```bash
init-project() {
    echo "Initialzing project at $project_dir"
    $tiger_cmd init-project "$project_dir" "$tiger_template"
}

init-config() {
    resolve-config "$config_in" > "$project_dir/tiger.config"
}

init-cluster() {
    pushd-quiet "$project_dir"
    $tiger_cmd init
    popd-quiet
}
```

---

#### Python

```python
def load_server_config(file_name):
    config = dna.config.parse(file_name)
    if not config:
        raise ValueError("Config file not found: %s" % file_name)
    return dna.secrets.resolve_config_secrets(config)

def server_name_from_dir(server_dir):
    return os.path.basename(server_dir)

def match_servers(pattern, location='.'):
    matched_server_dirs = match_server_dirs(pattern, location)
    return [load_server(server_dir) for server_dir in matched_server_dirs]
```

---

#### Erlang

```erlang
encode_meta([]) -> <<>>;
encode_meta(Meta) -> erlang:term_to_binary(Meta).

date_to_timestamp(Date) when is_integer(Date) -> Date.

maybe_auto_ignore_panic(Key, State) ->
    maybe_auto_ignore_panic(key_matches_auto_ignore(Key), Key, State).

key_matches_auto_ignore(Key) ->
    key_matches(Key, auto_ignore_patterns()).

```

---

<img src="ocd.jpg">

---

<img src="obvious.jpg">

---

## Goals

- At-a-glance obviousness
- No surprises (non obvious)
- Clear intent over "correctness"

---

# `ci.sh`

- Our "continuous integration" script
- I need to make a relatively simple change
- At least it seems that way

---

# Remarks

---

#### `ci.sh` - part 1

```bash
#!/bin/bash -ex
# Used for CI Integration Test

PROVIDER=${1/-*/}
VENDOR=${1/*-/}
export CLUSTER_NAME=${2:-tiger-testing}
PROJECT_DIR=$(readlink -f .tiger)
SCRIPT_DIR=$(readlink -f .)

export TIGER_PROJECT_DIR=$PROJECT_DIR

export PATH=$SCRIPT_DIR:$PATH

export DEBIAN_FRONTEND=noninteractive
```
---

#### `ci.sh` - part 2

```bash
# Update the DEVELOPMENT tiger-cli Docker image
if [ -z ${SKIP_RELEASE+x} ]; then
    pushd containers/cli
      make clean
    popd

    pushd release
    ./release ./VERSIONS-dev $SCRIPT_DIR
    ln -fs $SCRIPT_DIR/tiger-DEVELOPMENT $SCRIPT_DIR/tiger
    popd
else
    # Use an existing release, TIGER_CLI_VERSION will have to be
    # provided by caller
    ln -fs $SCRIPT_DIR/containers/cli/tiger $SCRIPT_DIR/tiger
fi
```

---

#### `ci.sh` - part 2 (aside)

```bash
update-the-development-tiger-cli-docker-image() {
    ...
}
```

---

#### `ci.sh` - part 2 (aside)

```bash
update-the-development-tiger-cli-docker-image() {
    if [ -z ${SKIP_RELEASE+x} ]; then
        pushd containers/cli
          make clean
        popd

        pushd release
        ./release ./VERSIONS-dev $SCRIPT_DIR
        ln -fs $SCRIPT_DIR/tiger-DEVELOPMENT $SCRIPT_DIR/tiger
        popd
    else
        # Use an existing release, TIGER_CLI_VERSION will have to be
        # provided by caller
        ln -fs $SCRIPT_DIR/containers/cli/tiger $SCRIPT_DIR/tiger
    fi
}
```

---

#### `ci.sh` - part 2 (aside)

```bash
update-the-development-tiger-cli-docker-image() {
    if [ -z ${SKIP_RELEASE+x} ]; then
        pushd containers/cli
          make clean
        popd

        pushd release
        ./release ./VERSIONS-dev $SCRIPT_DIR
        ln -fs $SCRIPT_DIR/tiger-DEVELOPMENT $SCRIPT_DIR/tiger
        popd
    else
        use-an-existing-release
    fi
}
```

---

#### `ci.sh` - part 2 (aside)

```bash
update-the-development-tiger-cli-docker-image() {
    if [ -z ${SKIP_RELEASE+x} ]; then
        i-sure-wish-there-was-a-comment-here-heck-just-do-stuff
    else
        use-an-existing-release
    fi
}
```

---

#### `ci.sh` - part 2 (aside)

```bash
update-the-development-tiger-cli-docker-image() {
    if skip-release-env-is-not-set; then
        i-sure-wish-there-was-a-comment-here-heck-just-do-stuff
    else
        use-an-existing-release
    fi
}
```

---

#### `ci.sh` - part 2

```bash
# Update the DEVELOPMENT tiger-cli Docker image
if [ -z ${SKIP_RELEASE+x} ]; then
    pushd containers/cli
      make clean
    popd

    pushd release
    ./release ./VERSIONS-dev $SCRIPT_DIR
    ln -fs $SCRIPT_DIR/tiger-DEVELOPMENT $SCRIPT_DIR/tiger
    popd
else
    # Use an existing release, TIGER_CLI_VERSION will have to be
    # provided by caller
    ln -fs $SCRIPT_DIR/containers/cli/tiger $SCRIPT_DIR/tiger
fi
```

---

#### `ci.sh` - part 2 (aside)

```bash
create-tiger-release-i-think() {
    pushd containers/cli
      make clean
    popd

    pushd release
    ./release ./VERSIONS-dev $SCRIPT_DIR
    ln -fs $SCRIPT_DIR/tiger-DEVELOPMENT $SCRIPT_DIR/tiger
    popd
}
```

---

#### `ci.sh` - part 2 (aside)

```bash
create-tiger-release-im-pretty-sure-with-odd-cli-clean-first() {
    pushd containers/cli
      make clean
    popd

    pushd release
    ./release ./VERSIONS-dev $SCRIPT_DIR
    ln -fs $SCRIPT_DIR/tiger-DEVELOPMENT $SCRIPT_DIR/tiger
    popd
}
```

---

#### `ci.sh` - part 2 (aside)

```bash
update-the-development-tiger-cli-docker-image() {
   if skip-release-env-is-not-set; then
      create-tiger-release-im-pretty-sure-with-odd-cli-clean-first
   else
      use-an-existing-release
   fi
}
```

---

#### `ci.sh` - part 4

```bash
# Upgrade existing cluster
if [ -f $PROJECT_DIR/shared/tiger.config ]; then
  mv $PROJECT_DIR/shared/tiger.config $PROJECT_DIR/shared/project.config
fi

# pass on terraform and dna vars with credentials to Tiger cli container
vars=$(env | grep '^TF_VAR_\|^DNA_ENV_' | sed -e 's/=.*//')
export DOCKER_ARGS="$(for v in $vars; do echo -n "-e $v "; done) $DOCKER_ARGS"

echo "Init project: ${CLUSTER_NAME}"''
mkdir -p $PROJECT_DIR
pushd $PROJECT_DIR
tiger init-project --reinit . $PROVIDER
popd
```

---

#### `ci.sh` - part 5

```bash
echo "Creating configuration"
# server ip
#ip=$(ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p' | head -n 1)
# EC2 public ip
#ip=$(curl -sS ipecho.net/plain)
export EXTERNAL_IP=$(curl -sS ident.me)
pushd $PROJECT_DIR

echo ip is ${EXTERNAL_IP}
CONFIG=$SCRIPT_DIR/ci/$PROVIDER-$VENDOR.config
if [ ! -f $CONFIG ]; then
  CONFIG=$SCRIPT_DIR/ci/$PROVIDER.config
fi
cat $CONFIG | envsubst > ./tiger.config
```

---

#### `ci.sh` - part 6

```bash
for image in castle cjoc master router elasticsearch
do
  if [ -f $SCRIPT_DIR/versions/$image/VERSION ]; then
    tee -a ./tiger.config << EOF
[$image]
docker_version            = $(cat $SCRIPT_DIR/versions/$image/VERSION)

EOF

  fi
done
```

---

#### `ci.sh` - part 7

```bash
echo "Setting default credentials"
mkdir -p $PROJECT_DIR/shared
echo "marathon:tiger-admin" > $PROJECT_DIR/shared/marathon-creds
echo "tiger-admin" > $PROJECT_DIR/shared/router-pwd
echo "admin tiger-admin " > $PROJECT_DIR/shared/elasticsearch-creds

if [ -e ./.tiger-initialized ]; then
    echo "Running tiger upgrade"
    tiger upgrade
else
    echo "Running tiger init"
    tiger init
fi
export MARATHON_CREDENTIALS=$(cat $PROJECT_DIR/shared/marathon-creds)
echo Marathon credentials: $MARATHON_CREDENTIALS

popd
```

---

#### `ci.sh` - part 8

```bash
export MARATHON_CREDENTIALS=$MARATHON_CREDENTIALS
if [ "$PROVIDER" = "aws" ]; then
  LOAD_BALANCER_RESOURCE=elb
else
  LOAD_BALANCER_RESOURCE=controller-1
fi
export CLUSTER_URL=$(terraform output -state="$PROJECT_DIR/.terraform/$LOAD_BALANCER_RESOURCE/terraform.tfstate" hostname_base)
cat $SCRIPT_DIR/ci/index.html | envsubst > ./index.html

# wait for cjoc
set +ex
export CJOC_PING_MAX_RETRIES=25
for i in `seq 1 $CJOC_PING_MAX_RETRIES`; do
  echo "Waiting for tasks to be alive... attempt $i of $CJOC_PING_MAX_RETRIES"
  curl -fsSL --user $MARATHON_CREDENTIALS -H "Accept: application/json" http://marathon.$CLUSTER_URL/v2/apps/jce/cjoc/tasks | grep '"alive":true' && \
    (! curl -fsSL --user $MARATHON_CREDENTIALS -H "Accept: application/json" http://marathon.$CLUSTER_URL/v2/apps/jce/castle/tasks | grep -q '"alive":false') && \
    curl -fsSL --user $MARATHON_CREDENTIALS -H "Accept: application/json" http://marathon.$CLUSTER_URL/v2/apps/jce/elasticsearch/tasks | grep '"alive":true' && \
    curl -fsSL --user $MARATHON_CREDENTIALS -H "Accept: application/json" http://marathon.$CLUSTER_URL/v2/apps/masters/eval-master/tasks && \
    break;
  sleep 5
done
set -ex
```

---

#### `ci.sh` - part 9

```bash
# run tests
pushd $PROJECT_DIR
echo "Running tiger check"
tiger check
popd
```

---

## Concerns

- Clear/obvious intent
- State life cycle - create, read, modify
- Interfaces (vars, exported vars, cmd/function args)
- The rubber duck doesn't understand

---

### Rainbow Wolf

<img src="rainbow-wolf-2.jpg" height="500">

---

# `ci-init-cluster`

---

### `ci-init-cluster` Overview

- Header ceremony
- Variable definitions
- Include "libs"
- Function definitions
- Script implementation

---

#### `ci-init-cluster`

```bash
# Header ceremony
...
# Variable definitions + includes
...
# Function definitions
...

init-new-cluster
```

---

#### `ci-init-cluster` - Header, Vars, Libs

```bash
#!/bin/bash -eu
# Creates a Tiger cluster from scratch

set -o pipefail

if ${trace:-false}; then
    set -x
fi

ci_dir=$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )

. "$ci_dir/ci-support"

```

---

#### `ci-init-cluster` - Functions

```bash
init-new-cluster() {
    destroy-existing-cluster
    clean-project
    init-project
    init-config
    init-cluster
    test-cluster
}
```

---

#### `ci-init-cluster` - Functions

```bash
destroy-existing-cluster() {
    if can-destroy-cluster; then
        echo "Destroying Tiger cluster"
        pushd-quiet "$project_dir"
        $tiger_cmd refresh-project 2> /dev/null
        $tiger_cmd destroy -f
        popd-quiet
    fi
}

can-destroy-cluster() {
    test -e "$project_dir/.tiger-project" -a \
         -e "$project_dir/.dna-project" -a \
         -e "$project_dir/servers/tiger"
}
```

---

#### `ci-init-cluster` - Functions

```bash
init-project() {
    echo "Initialzing project at $project_dir"
    $tiger_cmd init-project "$project_dir" "$tiger_template"
}

init-config() {
    resolve-config "$config_in" > "$project_dir/tiger.config"
}

init-cluster() {
    pushd-quiet "$project_dir"
    $tiger_cmd init
    popd-quiet
}

test-cluster() {
    pushd-quiet "$project_dir"
    $tiger_cmd check default
    popd-quiet
}
```

---

# Done!

---

## Advice for Bash

---

<img src="globals-bad.jpg" height="500">

---

## Globals in Bash

- File system
- Commands (shell path)
- Shell environment

---

### Rules of Thumb for "Globals"

- Avoid required environment variables (programming through globals)
- Minimize script variables
- Define script variables using args
- Use functions to compute derivable values

---

```bash
#!/bin/bash -eu

project="${1:?usage: $0 PROJECT}"

scripts-dir() {
    echo "$project/scripts"
}

echo "Scripts dir is $(scripts-dir)"
```

---

# Other Patterns

---

## Pipelining

```bash
some-list() {
     echo 1
     echo 2
     echo 3
}

do-something() {
    echo "Got $1"
}

some-list | while read x; do
    do-something $x
done
```

---

## Boolean Checks

```bash
is-filename() {
    if [[ "${1:?}" =~ "/" ]]; then true; else false; fi
}
```

```bash
is-file() {
    test -e "${1:?}"
}
```

---

## Failing Fast

```bash
set -e
set -u
set -o pipefail
```

---

## Arguments

```bash
foo() {
    local required="${1:?}"
    local optional="${2:-default}"

    local watch_out_for_me="$(something-that-might-fail)"

    local safe_form
    safe_form="$(something-that-might-fail)"
}
```

---

## Use Local!

```
foo() {
    _arg1="hello"
    bar
    echo "_arg1=$_arg1"
}

bar() {
    _arg1="bye"
    # Do something with _arg1
}
```

---

## Wrapping Up

- Bash is not a functional language
- Programming is not either/or
- Everyone has his or her own personal issues
- They're wrong
- JK LOL OMG WTF

---

## Discussion

#### @gar1t on Twitter
