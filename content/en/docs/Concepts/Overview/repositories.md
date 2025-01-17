---
title: "Creating Luet repositories"
linkTitle: "Creating Luet repositories"
weight: 2
date: 2017-01-05
description: >
  How to create Luet repositories
---

After a set of packages has been built, a repository must be created in order to make them accessible by Luet clients. A Repository can be served either local files or via http(s) (at the moment of writing). Luet, by default, supports multiple-repositories with priorities.

## Create a repository

After issuing a `luet build`, the built packages are present in the output build directory. The `create-repo` step is needed to generate a portable tree, which is read by the clients, and a `repository.yaml` which contains the repository metadata.

Note that the output of `create-repo` is *additive* so it integrates with the current build content. The repository is composed by the packages generated by the `build` command (or `pack`) and the `create-repo` generated metadata. 

### Flags

Some of the relevant flags for `create-repo` are:

- **--descr**: Repository description
- **--name**: Repository name
- **--output**: Metadata output folder (while a different path can be specified, it's prefered to output the metadata files directly to the package directory).This most of the time matches the packages path for convenience.
- **--packages**: Directory where built packages are stored. This most of the time is also the output path.
- **--reset-revision**: Reset the repository revision number
- **--tree-path**: Specify a custom name for the tree path. (Defaults to tree.tar)
- **--tree-compression**: Specify a compression algorithm for the tree. (Available: gzip, Defaults: none)
- **--tree**: Path of the tree which was used to generate the packages and holds package metadatas
- **--type**: Repository type (http/local). It is just descriptive, the clients will be able to consume the repo in whatsoever way it is served.
- **--urls**: List of URIS where the repository is available

See `luet create-repo --help` for a full description.

## Example

Build a package and generate the repository metadata:

```
$> mkdir package

$> cat <<EOF > package/build.yaml
image: busybox
steps:
- echo "foo" > /foo
EOF

$> cat <<EOF > package/definition.yaml
name: "foo"
version: "0.1"
category: "bar" # optional!
EOF

$> luet build --all --destination $PWD/out/ --tree $PWD/package

📦  Selecting  foo 0.1
📦  Compiling foo version 0.1 .... ☕
🐋  Downloading image luet/cache-foo-bar-0.1-builder
🐋  Downloading image luet/cache-foo-bar-0.1
📦   foo Generating 🐋  definition for builder image from busybox
🐋  Building image luet/cache-foo-bar-0.1-builder
🐋  Building image luet/cache-foo-bar-0.1-builder done
 Sending build context to Docker daemon  4.096kB
 ...

$> luet create-repo --name "test" --output $PWD/out --packages $PWD/out --tree $PWD/package
 For repository test creating revision 1 and last update 1580641614...

$> ls out
foo-bar-0.1-builder.image.tar  foo-bar-0.1.image.tar  foo-bar-0.1.metadata.yaml  foo-bar-0.1.package.tar  repository.yaml  tree.tar

```

## Notes

- The tree of definition being used to build the repository, and the package directories must **not** be symlinks.
- To build a repository is not required to hold the packages artifacts, only the respective `metadata.yaml` file is required.