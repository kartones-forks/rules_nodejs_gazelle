# Javascript & Typescript language support for Gazelle

Bazel [Gazelle](https://github.com/bazelbuild/bazel-gazelle) rule
that generates BUILD file content for javascript/typescript code.

## Setup

### Gazelle

To use this extension in a project, you'll need to add Gazelle and its dependencies to your `WORKSPACE` file.
Follow the instructions at https://github.com/bazelbuild/bazel-gazelle#running-gazelle-with-bazel

### rules_nodejs

Many of the rules generated by this extension also depend on rules_nodejs, instructions for installing rules_nodejs can be found at https://bazelbuild.github.io/rules_nodejs/install.html and be sure to install `jest` and `@bazel/typescript` with `npm`

### Repository rule

Then add this repository to your `WORKSPACE` file:

```starlark
http_archive(
    name = "com_github_benchsci_rules_nodejs_gazelle",
    sha256 = "####",
    strip_prefix = "rules_nodejs_gazelle-####",
    urls = [
        "https://github.com/benchsci/rules_nodejs_gazelle/archive/####.tar.gz",
    ],
)
```

### Gazelle rule

Once you have set up a `gazelle` rule by following the installation instructions in [`Gazelle`](#Gazelle), you can add this extension as a language.

It should look something like this:

```starlark
load("@bazel_gazelle//:def.bzl", "DEFAULT_LANGUAGES", "gazelle", "gazelle_binary")

gazelle(
    name = "gazelle",
    gazelle = ":gazelle_js",
)

gazelle_binary(
    name = "gazelle_js",
    languages = DEFAULT_LANGUAGES + [
        "@com_github_benchsci_rules_nodejs_gazelle//gazelle",
    ],
)
```

### NPM setup

If you are using NPM for your project, you will probably want to add at least these directives to your root `BUILD` file as well:

```starlark
# gazelle:exclude node_modules
# gazelle:js_package_file package.json
```

### Custom rule or macro implementations

If you have custom implementations of the rules generated by this extension, you may wish to re-map them with the `map_kind` directive:

```starlark
# gazelle:map_kind js_library js_library @build_bazel_rules_nodejs
# gazelle:map_kind ts_project ts_project @my_local_repo
```

## Directives

Gazelle can be configured with _directives_, which are written as top-level
comments in build files.

Directive comments have the form `# gazelle:key value`.
[More information here](https://github.com/bazelbuild/bazel-gazelle#directives)

Directives apply in the directory where they are set _and_ in subdirectories.
This means, for example, if you set `# gazelle:prefix` in the build file
in your project's root directory, it affects your whole project. If you
set it in a subdirectory, it only affects rules in that subtree.

Example of most of these directives can be found in [gazelle/testdata](gazelle/testdata)

The following directives are recognized by this plugin:

<table>
<thead>
  <tr>
    <th>Directive</th>
    <th>Default value</th>
  </tr>
</thead>
<tbody>

  <tr>
    <td><code># gazelle:js_extension</code></td>
    <td><code>enabled</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Controls whether the JS extension is enabled or not. Sub-packages inherit this value. Can be either "enabled" or "disabled".</p></td>
  </tr>

  <tr>
    <td><code># gazelle:npm_label</code></td>
    <td><code>@npm//</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Defines the label prefix for third npm dependencies e.g @npm// OR //:node_modules/ </p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_lookup_types true|false</code></td>
    <td><code>false</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Causes Gazelle to try and find a matching @npm//types dependency for each @npm dependency, including @types/node for Node.js builtins</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_package_file package.json</code></td>
    <td><code>none</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Instructs Gazelle to use a package.json file to lookup imports from dependencies and devDependencies</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_import_alias some_folder other</code></td>
    <td><code>none</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Specifies partial string substitutions applied to imports before resolving them. Eg. <code># gazelle:js_import_alias foo bar</code> means that <code>import "foo/module"</code> will resolve to the package <code>bar/module</code>. This directive can be used several times.</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_visibility label</code></td>
    <td><code>none</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">By default, internal packages are only visible to its siblings. This directive adds a label internal packages should be visible to additionally. This directive can be used several times, adding a list of labels.</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_root</code></td>
    <td><code>workspace root</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Specifies the current package (folder) as a JS root. Imports for JS and TS consider this folder the root level for relative and absolute imports. This is used on monorepos with multiple Python projects that don't share the top-level of the workspace as the root.</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_aggregate_modules true|false</code></td>
    <td><code>false</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Generate 1 js_library, or ts_project rule per package when a <code>index.ts</code> or <code>index.js</code> file is found, rather than 1 per file. The js_root pkg cannot be a module</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_aggregate_web_assets true|false</code></td>
    <td><code>false</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Causes Gazelle to generate 1 web_assets rule, rather than 1 per file</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_aggregate_all_assets true|false</code></td>
    <td><code>false</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Generates a <code>web_assets</code> rule in the configured <code>web_root</code> that refers to all of the <code>web_assets</code> rules in child packages using</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_web_asset .json,.css,.scss</code></td>
    <td><code>none</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Files with a matching suffix will have <code>web_assets</code> rules created for them</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_quiet true|false</code></td>
    <td><code>false</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Silence extension warnings about missing imports (overrides gazelle:js_verbose)</p></td>
  </tr>

  <tr>
    <td><code># gazelle:js_verbose true|false</code></td>
    <td><code>false</code></td>
  </tr>
  <tr>
    <td colspan="2"><p dir="auto">Print more information about missing imports (overrides gazelle:js_quiet)</p></td>
  </tr>

</tbody>
