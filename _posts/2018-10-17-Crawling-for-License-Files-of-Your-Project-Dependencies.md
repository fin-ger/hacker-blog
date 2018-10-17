---
title: Crawling for License Files of Your Project Dependencies
published: true
---

We are currently in the process of deploying some closed source software and
are facing the problem of accumulating all open source licenses for display
in our user interface. As we have a whole bunch of projects written in go and
nodejs I decided to write a small utility script that crawls all our projects
and collects all licenses of direct dependencies.

I started with go dependencies and tried using a combination of `find` and `yq`
to collect the dependencies from our `glide.yaml` files. This turned out to be
quite unsuccessful as `yq` is way to bare-metal and cannot handle my holy
requirements on a yaml-query language. So I ended up using `yq` to convert the
yaml files to json and proceed with `jq` which is way more fancy and can do a
whole lot of magic stuff.

To actually get the license content I am using the github API with some `curl`.
This has one disadvantage as not all our dependencies are using a github import
path. I am using some `sed` magic to convert these import paths to their github
project paths which was actually quite successful. As github has a rate limiting
on their API I have to login with my username and password to make all
necessary requests.

I ended up with the following one-liner which I will integrate in a fancy
utility script:

```bash
find -L "${FOLDER}" -name glide.yaml |
  sed -r \
      -e '/makefiles/d' \
      -e '/autobahnmeisterei/d' |
  xargs -i yq r {} 'import[*].package' -j |
  jq -r '.[]' 2>/dev/null |
  sort |
  uniq |
  sed -r \
      -e 's;golang\.org/x/(\w+);github.com/golang/\1;' \
      -e 's;k8s.io/(\w+);github.com/kubernetes/\1;' \
      -e '/robulab/d' -e '/EmbeddedEnterprises/d' |
  sed -r -e 's;github.com/([-A-Za-z0-9_/]+);\1;' |
  while read -r project
  do
      curl -u "${username}:${password}" "https://api.github.com/repos/${project}/license" |
          jq -r '.content' |
          base64 -d |
          jq --arg project "${project}" -asR '{ project: $project, license: . }'
  done |
  jq -s '.'
```

After that I needed the same output for all our nodejs projects. This turns out
to be way more complicated as node does not hold all dependencies in one field
but rather spread into multiple dependency types. I learned about the `//` and
`*` operator of `jq` for default values when a query results in null and
merging of objects. After collecting all dependencies of a node project I had to
find a way to get the content of the dependencies license files. I found no
better way than searching for `LICENSE` files in `node_modules`. I am currently
not using the SPDX license fields of the `package.json` as I need the license
content. However, not all dependencies have a `LICENSE` file located in their
repository and adding a fallback to the SPDX license text may be a valid option.
Currently I am just producing a placeholder text for those dependencies.

After a lot of digging into `jq` syntax I ended up with the following scipt:

```bash
find "${FOLDER}" -name package.json |
  sed -r \
      -e '/node_modules/d' \
      -e '/dist/d' \
      -e '/makefiles/d' \
      -e '/Infrastructure/d' |
  while read -r project_path
  do
    project_dir="$(dirname "${project_path}")"
    jq -r '.peerDependencies // {} * .devDependencies // {} * .dependencies // {} | keys[]' "${project_path}" |
    while read -r project
    do
      if [[ -f "${project_dir}/node_modules/${project}/LICENSE" ]]
      then
        license="${project_dir}/node_modules/${project}/LICENSE"
      elif [[ -f "${project_dir}/node_modules/${project}/LICENSE.md" ]]
      then
        license="${project_dir}/node_modules/${project}/LICENSE.md"
      elif [[ -f "${project_dir}/node_modules/${project}/LICENSE.txt" ]]
      then
        license="${project_dir}/node_modules/${project}/LICENSE.txt"
      else
        echo "{ \"project\": \"${project}\", \"license\": \"UNKNOWN\" }"
        continue
      fi

      jq --arg project "${project}" -asR '{ project: $project, license: . }' "${license}"
    done
  done |
  jq -s 'unique_by(.project)'
```

In the end this solutions is currently good enough. I will use the produced
json in my user interface to display a long list of open source licenses.
