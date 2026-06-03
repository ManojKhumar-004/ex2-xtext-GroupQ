#### 💯Points: ![Points bar](../../blob/badges/.github/badges/points-bar.svg)

#### 📝 [Report](../../blob/badges/report.md)

---

# Xtext Assignment

This project is an assignment for assessing the learning of Xtext, a framework for the development of programming languages and domain-specific languages (DSLs). Make sure to initially access this repository via the GitHub Classroom link provided by your instructor (this creates a copy of the repository for you) and follow the instructions below to complete the assignment.

## Learning Xtext Tutorial

Before you start working on the assignment, make sure that you have completed the [Xtext tutorial of the lecture](https://se-buw.de/teaching/gse/tutorials/xtext/). It covers the definition of Xtext grammars, including meta-model interfaces, enumerations, cross-references (`=[RefType]`), lists (`+=`), and the implementation of code generators using Xtend's dispatch methods.

Your implementation must follow the structure and style of the tutorial.

## Application Domain: Smart Home

You are tasked with building a textual Domain Specific Language (DSL) for access control in a **Smart Home** system. 

Writing security policies in raw code is dangerous and error-prone. In this exercise, you will build a DSL with Xtext that models these rules using a clean, readable syntax tailored specifically to the **Smart Home** domain. Then, you will write a generator that compiles the textual models with extension `rbac` into Java, allowing us to automatically evaluate the policies for a given scenario.


## Language Syntax by Example

The grammar should be developed by looking at how the concrete language elements are structured in the examples of `*.rbac` files below. 

### Basic Declarations
Before you can use actors (keyword `user`), assets (keyword `device`), and operations (keyword `command`) in access rules, they must be declared. Declarations can appear in any order at the top level of your document:

```text
// Declaring standalone actors
user Resident

// Declaring resources being protected
device FrontDoor

// Declaring operations that can be performed
command lock
```

### Actor Inheritance
Actors can inherit permissions from other previously declared actors. This is represented by declaring a child actor followed by the keyword `inherits` and the parent actor's identifier:

```text
user Resident

// Alice inherits all permissions defined for Resident
user Alice inherits Resident
```

### Policies, Scopes, and Rules
A policy (keyword `home_profile`) groups security configurations inside a named block using curly braces. Within a policy, rules are grouped (keyword `for_user`) by the actor they apply to (the actor's scope block). 

Each rule specifies whether a single operation is allowed (keyword `grant`) or denied (keyword `revoke`) on a single asset using the  keyword `on`. 

```text
home_profile NightMode {
    
    // Define a scope block for Resident
    for_user Resident {
        // An allow rule: 1 operation on 1 asset
        grant lock on FrontDoor

        // A deny rule: 1 operation on 1 asset
        revoke unlock on FrontDoor
    }
}
```

### Complete Example Policy Document

A complete, valid document (`example.rbac`) in your DSL combining all of these structural features looks like this:

```text
// Actor hierarchy
user Resident
user Alice inherits Resident
user Bob inherits Resident

// Assets
device FrontDoor
device Thermostat

// Operations
command lock
command unlock
command set_temperature

// Security Policy
home_profile NightMode {
    for_user Resident {
        grant lock on FrontDoor
        revoke unlock on FrontDoor
    }
}

// Another Security Policy
home_profile NightMode2 {
    // Rules for the first inheriting actor
    for_user Alice {
        grant lock on Thermostat
        revoke set_temperature on Thermostat
    }

    // Rules for the second inheriting actor
    for_user Bob {
        grant unlock on FrontDoor
        grant set_temperature on Thermostat
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

