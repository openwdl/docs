---
outline: [1, 2]
---

# Overview

The Workflow Description Language has a number of **versions** that are
constantly improving. This guide lists all major versions of the language—the
major improvements for each version, common pitfalls when upgrading, and how to
get help.

If you're interested in learning more about how versioning works and how you can
specify the version of your WDL documents, see the [Versioning section] within
the Language Guide.

::: tip
Changes are grouped in terms of their **impact**. The word "impact" here is a
combination of "how important is the change" along with "how many existing WDL
documents are likely to be affected by this change". They are subjective
determinations made by the upgrade guide curator during the curator process, but
they are often helpful in quickly scanning upgrade elements to see which might
apply to your situation.
:::

# WDL v1.3

WDL v1.3 introduces enumeration types, `else if`/`else` conditional clauses, new
standard library functions, and enhanced retry capabilities for tasks. All
changes are backwards compatible with previous `v1.x` releases. The full release
notes can be found [here][wdl-v1.3-release-notes].

### Checklist

Use this checklist to ensure you hit all of the sections.

#### Moderate impact changes

<input type="checkbox" /> New enumeration types ([link](#new-enumeration-types)).<br />
<input type="checkbox" /> New `else if` and `else` conditional clauses ([link](#new-else-if-and-else-conditional-clauses)).<br />
<input type="checkbox" /> New `split` standard library function ([link](#new-split-standard-library-function)).<br />
<input type="checkbox" /> Clarified path existence requirements ([link](#clarified-path-existence-requirements)).<br />

#### Low impact changes

<input type="checkbox" /> New `task.previous` variable ([link](#new-taskprevious-variable)).<br />
<input type="checkbox" /> New `task.max_retries` variable ([link](#new-taskmax_retries-variable)).<br />

## Moderate impact changes

### New enumeration types

Enumeration types (`enum`) were introduced in
[#695](https://github.com/openwdl/wdl/pull/695) to define closed sets of named
variants with associated values. Enums support explicit and implicit typing, and
variant values can be of any WDL type including primitives, compound types, and
user-defined types. A new `value()` standard library function was also added to
extract the inner value from an enum variant.

```wdl
version 1.3

enum PrimaryColor {
  Red = "#FF0000"
  Green = "#00FF00"
  Blue = "#0000FF"
}

workflow example {
  PrimaryColor color = PrimaryColor.Red
  String hex_code = value(color)  # "#FF0000"
}
```

Look for anything that has a closed set of choices—these are good candidates to
replace with enumerations.

### New `else if` and `else` conditional clauses

The `else if` and `else` clauses were introduced in
[#699](https://github.com/openwdl/wdl/pull/699) to provide more expressive
conditional logic within workflows. Previously, to achieve an `else`, you had to
write two `if` statements with negated conditions.

```wdl
version 1.3

workflow example {
  input {
    String mode
  }

  if (mode == "fast") {
    call fast_analysis
  } else if (mode == "thorough") {
    call thorough_analysis
  } else {
    call default_analysis
  }
}
```

Look for pairs of `if` statements where the second condition is the negation of
the first—these can be simplified using `else`.

### New `split` standard library function

The `split` function was introduced in
[#729](https://github.com/openwdl/wdl/pull/729) to split a string into an array
of substrings based on a delimiter pattern.

```wdl
Array[String] parts = split("a,b,c", ",")  # ["a", "b", "c"]
```

Look for command blocks using `awk`, `cut`, or `tr` to split strings—these can
be replaced with the native `split` function.

### Clarified path existence requirements

The behavior of relative paths in `File` and `Directory` declarations was
clarified in [#735](https://github.com/openwdl/wdl/pull/735). Relative paths are
resolved relative to the WDL document's parent directory outside the `output`
section, and relative to the task's execution directory inside the `output`
section. Additionally, `File` and `Directory` values must exist at declaration
evaluation time—if a path does not exist when the declaration is evaluated, a
runtime error will occur. Optional files evaluate to `None` if the path does not
exist.

Review any `File` or `Directory` declarations that reference paths which might
not exist yet—these should either exist before the workflow runs, be marked as
optional (`File?` or `Directory?`), or be declared in the `output` section. Also
ensure that relative paths are updated to be relative to the correct location.

## Low impact changes

### New `task.previous` variable

The `task.previous` variable was added in
[#734](https://github.com/openwdl/wdl/pull/734), enabling runtime access to the
previous attempt's computed requirements. This is useful for implementing retry
logic that adjusts resources based on previous failures.

If you have tasks that sometimes fail due to insufficient memory or CPU,
consider using `task.previous` to automatically scale up resources on retry.

### New `task.max_retries` variable

The `task.max_retries` variable was added in
[#733](https://github.com/openwdl/wdl/pull/733) to provide access to the maximum
number of retry attempts configured for a task.

If you have retry logic that needs different behavior on the final attempt, you
can now use `task.max_retries` to detect when you're on the last retry.

# WDL v1.2

WDL v1.2 introduces a great deal of new syntax and changes to the specification.
Notably, changes were backwards compatible with all previous `v1.x` releases.
This sections only covers the high points related to upgrading—the full release
notes can be found [here][wdl-v1.2-release-notes].

### Checklist

Use this checklist to ensure you hit all of the sections.

#### Moderate impact changes 

<input type="checkbox" /> New `Directory` type ([link](#new-directory-type)).<br />
<input type="checkbox" /> Replacement of `runtime` section with `requirements` and `hints` sections ([link](#replacement-of-runtime-with-requirements-and-hints)). <br />
<input type="checkbox" /> Deprecation of `runtime` section ([link](#deprecation-of-runtime-section)). <br />
<input type="checkbox" /> New standard library functions
([link](#new-standard-library-functions)). <br />
<input type="checkbox" /> Multi-line strings ([link](#multi-line-strings)). <br />
<input type="checkbox" /> Optional `input:` statement ([link](#optional-input-statement)). <br />

#### Low impact changes 

<input type="checkbox" /> Addition of workflow `hints` ([link](#addition-of-workflow-hints)). <br />
<input type="checkbox" /> New requirements and hints keys ([link](#new-requirements-and-hints-keys)). <br />
<input type="checkbox" /> Metadata sections for structs ([link](#metadata-sections-for-structs)). <br />
<input type="checkbox" /> Exponentiation operator ([link](#exponentiation-operator)). <br />

## Moderate impact changes

### New `Directory` type

The `Directory` type was introduced in
[#641](https://github.com/openwdl/wdl/pull/641) to better semantically indicate
the use of a directory. If the intention of any of your arguments is to be used
to refer to a directory on the filesystem, you are encouraged to update the
parameters to a `Directory` type.

### Deprecation of `runtime` section

The `runtime` section, which previously held both requirement constraints and
hints to the execution engine, has now been split into the `requirements`
([#540](https://github.com/openwdl/wdl/issues/540)) section and `hints`
([#541](https://github.com/openwdl/wdl/issues/541)) section respectively. You
should split out these keys based on the definitions in the
specification<sup>[1](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-requirements-section),
[2](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-hints-section)</sup>.

🗑️ This change deprecates the `runtime` section, which will be removed in WDL v2.0.
That being said, if desired, you can continue to use the `runtime` section in your WDL
documents—it's just not recommended.

### New standard library functions

The following are new standard library functions and their definitions. You are
encouraged to read through them and replace any custom functionality that would
now be duplicated in favor of these functions.

- `contains_key`: whether or not a `Map` or `Object` contain a specific member
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-contains_key)).
- `values`: get the values from a `Map`
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-values)).
- `find`: search for a regular expression in a string
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-find)).
- `matches`: whether a string match a regular expression
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-matches)).
- `chunk`: split an array into sub-arrays
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-chunk)).
- `join_paths`: join two or more paths
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-join_paths)).
- `contains`: whether an array contains a specified value
  ([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-contains)).


### Multi-line strings

Multi-line strings were introduced with the following syntax.

```wdl
String foo = <<<
  my
  multi-line
  string
>>>
```

### Optional `input:` statement

As noted in [#524](https://github.com/openwdl/wdl/pull/524), the `input:`
statement that precedes call bodies is unnecessary historical boilerplate. This
statement is now optional in `call` bodies. You are encouraged to remove these
from your `call` bodies.

## Low impact changes

### Addition of workflow `hints`

Workflows gained a `hints` section in
[#543](https://github.com/openwdl/wdl/issues/543). You are encouraged to go read
the supported workflow
hints<sup>[1](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#workflow-hints)</sup>
and apply them to your workflow if any are relevant.

### New requirements and hints keys

- The following keys were added to the task `requirements` section: `fpga`
<sup>[1](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#hardware-accelerators-gpu-and--fpga)</sup>.
- The following keys were added to the task `hints` section: `disks`
<sup>[1](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-disks)</sup>,
`gpu`
<sup>[2](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-gpu-and--fpga)</sup>,
and `fpga`
<sup>[2](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#-gpu-and--fpga)</sup>.

You are encouraged to examine these keys are determine if any of these should be
specified for your tasks.

### Metadata sections for structs

Similarly to tasks and workflows, structs now have `meta` and `parameter_meta`
sections. You are encouraged to use these fields according to the definition in
the
specification<sup>[1](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#struct-definition)</sup>.

### Exponentiation operator

The exponentiation operator (`**`) was added in this release. You are encouraged
to go and update any manual exponentiation to use this operator instead.

You are encouraged to go read the specification section on this concept
([link](https://github.com/openwdl/wdl/blob/wdl-1.2/SPEC.md#multi-line-strings))
and use them where appropriate.

# Prior WDL versions

Versions of WDL prior to the ones outlined in this guide did not exist at the
time the upgrade guide was created. As such, they are not included in the guide.

[wdl-v1.3-release-notes]: https://github.com/openwdl/wdl/releases/tag/v1.3.0
[wdl-v1.2-release-notes]: https://github.com/openwdl/wdl/releases/tag/v1.2.0
[Versioning section]: ../language-guide/versions.md
