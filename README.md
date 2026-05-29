# Xtext Assignment

This project is an assignment for assessing the learning of Xtext, a framework for the development of programming languages and domain-specific languages (DSLs). Make sure to initially access this repository via the GitHub Classroom link provided by your instructor (this creates a copy of the repository for you) and follow the instructions below to complete the assignment.

## Learning Xtext Tutorial

Before you start working on the assignment, make sure that you have completed the [Xtext tutorial of the lecture](https://se-buw.de/teaching/gse/tutorials/xtext/). It covers the definition of Xtext grammars, including meta-model interfaces, enumerations, cross-references (`=[RefType]`), lists (`+=`), and the implementation of code generators using Xtend's dispatch methods.

Your implementation must follow the structure and style of the tutorial.

## Language Syntax by Example

The grammar should be developed by looking at how the concrete language elements are structured in the text file. 

### Basic Declarations
Before you can use actors (keyword `{{keyword_actor}}`), assets (keyword `{{keyword_asset}}`), and operations (keyword `{{keyword_operation}}`) in access rules, they must be declared. Declarations can appear in any order at the top level of your document:

```text
// Declaring standalone actors
{{keyword_actor}} {{example_actor1}}

// Declaring resources being protected
{{keyword_asset}} {{example_asset1}}

// Declaring operations that can be performed
{{keyword_operation}} {{example_operation1}}
```

### Actor Inheritance
Actors can inherit permissions from other previously declared actors. This is represented by declaring a child actor followed by the keyword `inherits` and the parent actor's identifier:

```text
{{keyword_actor}} {{example_actor1}}

// {{example_actor2}} inherits all permissions defined for {{example_actor1}}
{{keyword_actor}} {{example_actor2}} inherits {{example_actor1}}
```

### Policies, Scopes, and Rules
A policy (keyword `{{keyword_policy}}`) groups security configurations inside a named block using curly braces. Within a policy, rules are grouped (keyword `{{keyword_scope}}`) by the actor they apply to (the actor's scope block). 

Each rule specifies whether a single operation is allowed (keyword `{{keyword_allow}}`) or denied (keyword `{{keyword_deny}}`) on a single asset using the  keyword `on`. 

```text
{{keyword_policy}} {{example_policy}} {
    
    // Define a scope block for {{example_actor1}}
    {{keyword_scope}} {{example_actor1}} {
        // An allow rule: 1 operation on 1 asset
        {{keyword_allow}} {{example_operation1}} on {{example_asset1}}

        // A deny rule: 1 operation on 1 asset
        {{keyword_deny}} {{example_operation2}} on {{example_asset1}}
    }
}
```

### Complete Example Policy Document

A complete, valid document (`example.rbac`) in your DSL combining all of these structural features looks like this:

```text
// Actor hierarchy
{{keyword_actor}} {{example_actor1}}
{{keyword_actor}} {{example_actor2}} inherits {{example_actor1}}
{{keyword_actor}} {{example_actor3}} inherits {{example_actor1}}

// Assets
{{keyword_asset}} {{example_asset1}}
{{keyword_asset}} {{example_asset2}}

// Operations
{{keyword_operation}} {{example_operation1}}
{{keyword_operation}} {{example_operation2}}
{{keyword_operation}} {{example_operation3}}

// Security Policy
{{keyword_policy}} {{example_policy}} {
    {{keyword_scope}} {{example_actor1}} {
        {{keyword_allow}} {{example_operation1}} on {{example_asset1}}
        {{keyword_deny}} {{example_operation2}} on {{example_asset1}}
    }
}

// Another Security Policy
{{keyword_policy}} {{example_policy}}2 {
    // Rules for the first inheriting actor
    {{keyword_scope}} {{example_actor2}} {
        {{keyword_allow}} {{example_operation1}} on {{example_asset2}}
        {{keyword_deny}} {{example_operation3}} on {{example_asset2}}
    }

    // Rules for the second inheriting actor
    {{keyword_scope}} {{example_actor3}} {
        {{keyword_allow}} {{example_operation2}} on {{example_asset1}}
        {{keyword_allow}} {{example_operation3}} on {{example_asset2}}
    }
}
```

## Implementing the Language & Code Generator

To complete this language, you must define the grammar rules and write the code generator. Follow the style of the tutorials and implement the grammar and generator in the respective files inside the provided Eclipse project structure.

### Task 1 – Grammar Definition

Define the grammar rules in `gse.xtext.assignment/src/gse/xtext/assignment/AccessPolicies.xtext`.

*   **Meta-Model Interfaces:** Use rule alternatives to define abstract super-types (e.g., `Elements = A | B | C`).
*   **Keywords:** Use the exact keywords provided above for your DSL.
*   **Cross-References:** Ensure correct use of cross-references where appropriate. This provides the IDE with autocomplete and automatic validation.

### Task 2 – Code Generation

Implement the code generator in `gse.xtext.assignment/src/gse/xtext/assignment/generator/AccessPoliciesGenerator.xtend`. The generator must traverse the parsed AST and produce a `.java` file that implements the policy logic.

*   **File Creation:** Always generate a single Java file named `SecurityEvaluator.java`.
*   **Java Class & Method:** Generate a `public class SecurityEvaluator` with a method `public static boolean isAllowed(String actor, String operation, String asset)`. 
* The method should evaluate the policies defined in the DSL and return `true` if the action is allowed, and `false` if it is denied.
* An actor is allowed to perform an operation if there is a matching `allow` rule for that actor or an actor it inherits from. 
* Ignore contradicting `deny` rules for this exercise. If there is an `allow` rule, it takes precedence over any `deny` rules. 

For the code generation, follow these guidelines:
*   **Use `dispatch`:** Use Xtend's `dispatch def` methods to cleanly separate the generation logic for AST elements.
*   **Default Fallback:** Ensure that the very end of the generated Java method returns `false` as the safe default fallback.

