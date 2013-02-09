# CF_CleanUp

This document is intended to be a collection of recommendations to Adobe from the ColdFusion Community (meaning all ColdFusion developers), on how to improve the CFML and CFScript languages.

## WAT?

ColdFusion has been around for 18 years (which actually makes it younger than Ruby and Python) and while the overall features of the language have kept up with the times such as adding object-oriented programming and ORM the core language has not progressed as fast. 

ColdFusion 9 made giant strides forward with increased cfscript options but two things stand out like sore thumbs; the functions implemented as cfc's (query being a prime example) and documentation which barely exists for script.

Overall the language should embrace a JavaScript syntax and the simplicity of JavaScript libraries such as jQuery.

_Fork, add your two cents, and submit a pull request. No change is too small!_

## Opposing Viewpoints

There are bound to be disagreements on proposals, and we welcome differing opinions, and would be more than happy to have multiple proposals to fix the same problem. _Bring it on._

## Organization

Add proposals of specific changes to a tag or function's implementation to the alphabetized tag/function **Implementation Changes** section.

Add proposals for function/tag renaming, moving, etc to the **Organizational Changes** section.

Add syntax chagnes to the **Syntax Changes** section.

## Backwards Compatibility

ColdFusion should maintain backwards compatibility while improving the lanugage. The documentation and examples provided by Adobe should primarily feature the suggestions in this document.

Features, including language changes, that have been marked as deprecated need to be removed in the next version permantly.

# Proposals

## Organizational Changes

### Headless Functions

We have too many headless functions. Instead of 24 headless `ArrayX()` functions, we should have ~24 `arrayObj.x()` methods. The same goes for structures, strings, etc.

 * ArrayAppend
 * ArrayAvg
 * ArrayClear
 * ArrayDeleteAt
 * ArrayInsertAt
 * ArrayContains
 * ArrayIsDefined
 * ArrayIsEmpty
 * ArrayLen
 * ArrayMax
 * ArrayMin
 * ArrayDelete
 * ArrayNew
 * ArrayPrepend
 * ArrayResize
 * ArraySet
 * ArraySort
 * ArrayFind
 * ArrayFindNoCase
 * ArraySum
 * ArraySwap
 * ArrayToList
 * IsArray
 * ListToArray

Perhaps `myArray.length` would be better implemented as a property than a method. Likewise, some may not be good fits for instance methods, like `IsArray()`. In this case, we should either leave the functions as headless, or create a sort of class object from which they can be used without an object instance, such as: `Array.isArray( foo )`, or `Array.new(2)`.

Headless functions are not bad by definition, but making **all** functions headless is a bad choice.

## Implementation Changes: CFML

### CFQuery

Inserts that result in an auto-generated Identity column value should all return the created value in the same result key name. Currently every database platform has its own key (e.g. IDENTITYCOL for MSSQL, and GENERATED_KEY for MySQL), which is problematic for portable software (think blogs, frameworks, etc) as they have to have switches to deal with all possible cases. If the key was always the same name regardless of the DB platform, it would be easier for all developers to work with, and projects would be more portable. See also, [Bug #3490074](https://bugbase.adobe.com/index.cfm?event=bug&id=3490074).

## Implementation Changes: CFScript

### Queries

The current problem with CFScript queries is that the syntax is awkward and they don't behave the same as their tag counterparts. For example, to do a Query of Queries, you've got to pass the existing Query resultset into the new Query object so that it can be referenced. These are symptoms of a larger problem: While implementing some tags as CFC's is a fine stop-gap, Queries are simply not a good fit.

Our proposal is to remove Query.cfc and properly add it to the language. Our proposed syntax would be:

    [result =] query( SQL, parameters, options );

    parameters: {id: 1, lastName: 'foo'}
    options: {cachedWithin: createTimespan(...), ...}

Example:

```cfs
result = query(
    "insert into myTable (name) values (:name)",
    { name: "Bob the Builder" },
    { datasource: "myDb", username: "db_user", password: "db_pass" }
);
inserted_id = result.IdentityCol;
```

Being properly implemented into the CFScript language, instead of deferring to CFCs, should also mean that this syntax would properly have access to available-scoped query objects:

```cfs
variables.people = query("select name, age from person");

variables.octogenarians = query("
    select name, age from variables.people
    where age >= 80 and age <= 89
", {}, { dbType: "query" });
```

We believe that this syntax is easy to remember, terse, and just as intuitive as the `<cfquery>` tag. There is no need to call `execute()`, and the result is always returned.

## Syntax Changes

### Line breaks without a comma

A comma to mark the end of a line should be made optional. A line break would be designated by either a comma or a carraige return.

### Declare multiple variables with one var keyword. 

Just like in JavaScript add the ability to define multiple variables with one var keyword.
