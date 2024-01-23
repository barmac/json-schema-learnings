# JSON Schema Learnings

## Case Study: Property depending on itself

#### Maciej Barelkowski

---

## Goals:

* Learn specific features that allowed to overcome the problem
* Learn about the pitfalls to avoid

---

## What's JSON Schema and why do we need it?

* JSON Schema is a tool that allows you to annotate and validate JSON documents (thanks copilot!).
* We use it in element templates for validation and user guidance.
* Two ways to use it:
  * via code editor
  * via a library like `ajv`

---

## The Problem ([camunda/element-templates-json-schema#125](https://github.com/camunda/element-templates-json-schema/issues/125))

Condition was allowed to depend on the containing property which led to infinite loop.

---

## Solution

* Make sure that the condition is not allowed to depend on the containing property.
* Bonus points: Problem is detected and highlighted in code editor.

---

## [`$data`](https://ajv.js.org/guide/combining-schemas.html#data-reference) - naive solution

```json
{
  "condition": {
    "properties": {
      "property": {
        "not": {
          "const": {
            "$data": "2/id" // JSON Pointer
          }
        }
      }
    }
  }
}
```

---

## Problem with `$data`

* `$data` validates if property doesn't exist
* This means `not` is always true if `id` is not set

---

## [End solution](https://github.com/camunda/element-templates-json-schema/pull/128/files#diff-ee6be2266742098acb7feb7e3d946c44fc93a48369bc849d010a4dbd6d04def2)

* Uses conditional subschemas
* Logic: if condition makes property depend on itself, the condition is invalid.

---

## Problems that stay with us

* ajv !== JSON schema
* `$data` is only a proposal
* `$data` is not supported by all validators, so the solution doesn't work in VSCode
* increased complexity of the schema (but encapsulated!)

---

## Alternative: implement in code

Instead of using JSON Schema, we could implement the validation in code.

However, this solution has the same problems:

* Still not validating correctly in VSCode **and will not be even support for `$data` is added**
* Still increases the complexity

---

## Used features

* `$ref` for composability
* `not` for negation
* `if-then-else` for conditional subschemas
* `const` for constant values
* `$data` for self-reference
* JSON pointer for referencing properties

---

## Thoughts?
