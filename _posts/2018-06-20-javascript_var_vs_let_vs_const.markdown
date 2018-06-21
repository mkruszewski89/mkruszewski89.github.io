---
layout: post
title:      "Javascript Var vs Let vs Const"
date:       2018-06-21 00:45:17 +0000
permalink:  javascript_var_vs_let_vs_const
---


Javascript allows you to declare variables in these three different ways, and they all behave differently.

Javascript treats variables declared with 'var' as if they were declared first thing in scope, regardless of where the variable is actually declared in scope. You can think of this as the 'var' being "hoisted" to the top of scope. This can result in code behaving in unpredictable ways, as the 'var' is available before it is declared:
```
function varDemo(condition) {
    console.log(variable) // OUTPUT : undefined
    if (condition) {
		    console.log(variable) //OUTPUT: undefined
        var variable = "hello!"
				console.log(variable) //OUTPUT: "hello!"
    } else {
        console.log(variable) // OUTPUT : undefined
    }
}
```

'let' on the other hand provides more control through "block-level" scoping, whereby the declared variable is only available in the block {} it is declared:
```
function letDemo(condition) {
    console.log(variable) // OUTPUT : Uncaught ReferenceError: variable is not defined
    if (condition) {
		    console.log(variable) // OUTPUT : Uncaught ReferenceError: variable is not defined
        let variable = "hello!"
				console.log(variable) //OUTPUT: "hello!"
    } else {
        console.log(variable) // OUTPUT : Uncaught ReferenceError: variable is not defined
    }
}
```

'const' behaves the same as 'let', but the main difference is let can be reassigned while const cannot:
```
function reassigningDemo() {
let assignableVar = 1
assignableVar = 2
const unassignableVar = 1
unassignableVar = 2 // OUTPUT: Uncaught TypeError: Assignment to constant variable.
```

Generally 'const' is the most predictable and difficult to make careless mistakes with, but 'let' has a place where reassignment is absolutely necessary.
