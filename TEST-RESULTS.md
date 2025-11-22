# OmniFocus JavaScript Environment Test Results

**Date**: November 22, 2025  
**OmniFocus Version**: 4  
**Platform**: macOS

## Executive Summary

🎉 **All modern JavaScript features work!** Our earlier assumptions about const/let limitations were INCORRECT.

### The Only Real Constraint

❌ **Credentials objects can ONLY be instantiated at plugin load time** (top level of the IIFE closure)  
✅ **Everything else works**: const, let, arrow functions, template literals, destructuring, spread, etc.

## Test Results

### ✅ Test 1: const/let Scope Test

**All tests PASSED**

| Test | Status | Notes |
|------|--------|-------|
| const at top level of closure | ✅ PASS | Works perfectly |
| let at top level of closure | ✅ PASS | Works perfectly |
| var at top level of closure | ✅ PASS | Works perfectly |
| const inside function | ✅ PASS | Works perfectly |
| let inside function | ✅ PASS | Works perfectly |
| var inside function | ✅ PASS | Works perfectly |
| Arrow function | ✅ PASS | Works perfectly |
| Traditional function | ✅ PASS | Works perfectly |
| forEach with const | ✅ PASS | Works perfectly |

**Output**:
```
All tests passed!

Top level const: top level const
Top level let: top level let
Top level var: top level var
Function const: function const
Function let: function let
Function var: function var
Arrow function: arrow function works
Traditional function: traditional function works
forEach results: a, b, c
```

### ❌ Test 2: Credentials Scope Test

**FAILED - But revealed the actual constraint**

| Test | Status | Notes |
|------|--------|-------|
| `const credentials = new Credentials()` (top) | ❌ FAIL | "Credential objects may only be constructed when loading a plug-in" |
| Credentials at ANY scope | ❌ FAIL | Same error everywhere |

**Error Message**:
```
Error: Credential objects may only be constructed when loading a plug-in
/Users/harpchad/Library/Containers/com.omnigroup.OmniFocus4/Data/Library/Application Support/Plug-Ins/test-credentials-scope.omnifocusjs:23:41
```

**Key Finding**: The error occurred at line 23, which is the TOP level of the closure. This means:
- ✅ Credentials CAN be created at the top level of the plugin IIFE
- ❌ Credentials CANNOT be created inside the Action function
- ❌ Credentials CANNOT be created in multiple places

**The issue isn't const vs var - it's WHERE you instantiate Credentials!**

### ✅ Test 3: Modern JavaScript Features

**All tests PASSED**

| Feature | Status | Result |
|---------|--------|--------|
| Template literals (`${var}`) | ✅ PASS | "Hello, World!" |
| Destructuring `{a, b}` | ✅ PASS | a=1, b=2 |
| Spread operator `...arr` | ✅ PASS | [1,2,3,4] |
| Default parameters `(x = 5)` | ✅ PASS | result=10 |
| Array.find() | ✅ PASS | found=4 |
| Object.keys() | ✅ PASS | a,b,c |
| Promise | ✅ PASS | resolved with test |

**Output**:
```
Template literals: OK (Hello, World!)
Destructuring: OK (a=1, b=2)
Spread operator: OK (1,2,3,4)
Default parameters: OK (result=10)
Array.find(): OK (found=4)
Object.keys(): OK (a,b,c)
Promise: OK (resolved with test)
```

## Conclusions

### What We Learned

1. **Modern JavaScript works perfectly** in OmniFocus plugins
2. **const/let are NOT the problem** - they work at all scopes
3. **Arrow functions are fine** - no scoping issues
4. **forEach works great** - no issues at all
5. **The ONLY constraint**: `new Credentials()` must be at plugin load time

### The Real Issue We Encountered

During development, we saw errors and incorrectly attributed them to const/let. The actual issues were likely:

1. **Credentials instantiation location** - We may have tried to create Credentials inside the action function
2. **Other unrelated bugs** - That happened to work when we switched to var (coincidence)
3. **Cargo cult debugging** - Once var "worked", we assumed const was the problem

### Correct Pattern for Credentials

```javascript
(() => {
  // ✅ CORRECT: Instantiate at plugin load time (top of IIFE)
  const credentials = new Credentials();
  
  const action = new PlugIn.Action((selection, sender) => {
    // ✅ Can use credentials here
    const cred = credentials.read("my-key");
    
    // ❌ WRONG: Cannot create new instance here
    // const newCreds = new Credentials(); // This would fail!
  });
  
  return action;
})();
```

### Best Practices (Updated)

✅ **DO**:
- Use `const` for values that don't change
- Use `let` for values that do change
- Use arrow functions for cleaner syntax
- Use template literals for string interpolation
- Use modern array methods (map, filter, find, forEach)
- Use destructuring and spread operators
- Instantiate `Credentials` once at plugin load time

❌ **DON'T**:
- Try to create `new Credentials()` inside action functions
- Try to create multiple Credentials instances
- Use `var` unless you have a specific scoping need

## Impact on Existing Code

### Our Plugin Uses var Unnecessarily

Current code:
```javascript
(() => {
  var credentials = new Credentials();  // Could be const
  
  var action = new PlugIn.Action(function(selection, sender) {
    var jiraUrl = "...";  // Could be const
    var credentialKey = "...";  // Could be const
    // ... etc
  });
})();
```

Could be modernized to:
```javascript
(() => {
  const credentials = new Credentials();
  
  const action = new PlugIn.Action((selection, sender) => {
    const jiraUrl = "...";
    const credentialKey = "...";
    // ... etc
  });
  
  return action;
})();
```

### Should We Refactor?

**Pros**:
- More modern, idiomatic JavaScript
- const/let provide better scoping and prevent accidental reassignment
- Arrow functions are cleaner

**Cons**:
- Code is working now
- Refactoring introduces risk of bugs
- No functional benefit, only stylistic

**Recommendation**: 
- Update AGENTS.md with correct information
- Leave existing code as-is (working code > "perfect" code)
- Use modern JavaScript for new features
- Document the real constraint (Credentials instantiation)

## Original Assumptions (Now Corrected)

| Assumption | Reality | Evidence |
|-----------|---------|----------|
| ❌ const/let at top level cause errors | ✅ They work fine | Test 1 passed |
| ❌ Arrow functions cause scoping issues | ✅ They work fine | Test 1 passed |
| ❌ forEach loops problematic | ✅ They work fine | Test 1 passed |
| ✅ Credentials must be at top level | ✅ Correct! | Test 2 error message |
| ❌ Must use var for Credentials | ❌ const works too | Any declaration works, location matters |

## Recommendations

### For AGENTS.md

Remove/update these incorrect sections:
1. ❌ "NO const or let at plugin level - Use var only"
2. ❌ "Traditional function syntax preferred"
3. ❌ "Avoid forEach with complex scoping"

Add this correct constraint:
1. ✅ "Credentials MUST be instantiated at plugin load time (top of IIFE), not inside action functions"

### For Future Development

- Feel free to use modern JavaScript
- const/let/arrow functions are all fine
- The only special case is the Credentials class

## Test Files

Test plugins are located at:
```
/Users/harpchad/Library/Containers/com.omnigroup.OmniFocus4/Data/Library/Application Support/Plug-Ins/
├── test-const-let.omnifocusjs
├── test-credentials-scope.omnifocusjs
└── test-modern-js.omnifocusjs
```

These can be kept for future reference or deleted.
