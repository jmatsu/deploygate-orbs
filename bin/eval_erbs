#!/usr/bin/env bash

set -eux

TAG_PREFIX=$(basename $(pwd))
GIT_ROOT_DIR=$(basename $(git rev-parse --show-toplevel))

if [[ "$GIT_ROOT_DIR" == "$TAG_PREFIX" ]]; then
    echo "This script must be run from the inside of each orbs" 1>&2
    exit 1
fi

: "${TAG:-${CIRCLE_TAG}}"
export RELEASE_TAG=$(echo $TAG | sed -e "s/^$TAG_PREFIX-//")

if [[ ! $RELEASE_TAG =~ [0-9]+\.[0-9]+\.[0-9]+ ]]; then
    echo "tag name must follow the correct format. $RELEASE_TAG is wrong" 1>&2
    exit 1
fi

while read file_path; do
    ruby -rerb -e "puts ERB.new(File.read('erb/$file_path')).result" > $file_path

    if [[ "$file_path" =~ \.yml$ ]]; then
        cp $file_path $file_path.backup
        ruby -ryaml -rjson -e "puts YAML.load(YAML.load_file('$file_path.backup').to_json).to_yaml" > $file_path
        rm $file_path.backup
    fi

    if [[ "$file_path" =~ \.bash$ ]]; then
      chmod +x "$file_path"
    fi
done < <(ls -1 erb)