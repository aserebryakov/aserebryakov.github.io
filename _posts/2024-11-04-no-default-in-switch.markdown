---
layout: post
title:  "How Removing Default Branch from Switch Statement Can Save Your Time"
date:   2024-11-02 00:00:00 +0300
categories: c++
---

## Intro

I can't remember the source, but there is a general principle:

> Wrong code should not compile.

In this post, I’ll show a somewhat counterintuitive approach to the `switch` statement.

## Example

Let's take a look at the following example:

```cpp
#include <iostream>
#include "data_from_string.hpp"

void printDataType(const std::string& input) {
    const DATA_TYPE type{parseDataType(input)};

    switch (type) {
        case DATA_TYPE::INTEGER: {
            std::cout << "Integer!";
            break;
        }
        case DATA_TYPE::FLOAT: {
            std::cout << "Float!";
            break;
        }
        case DATA_TYPE::STRING: {
            std::cout << "String!";
            break;
        }
        default: {
            std::cout << "Unknown!";
        }
    }

    std::cout << std::endl;
}
```

For this example, we will assume that the `parseDataType()` function is defined in a header file 
provided by the third-party library `data_from_string`, version `v1.0.0`:

```cpp
// data_from_string.hpp

enum class DATA_TYPE {
    INTEGER,
    FLOAT,
    STRING
};

DATA_TYPE parseDataType(const std::string& input);
```

Everything looks and works as expected.

## Library Upgrade Catch

Now, let's assume that the maintainer released a new version of the library, `v1.1.0`, where a new data type was added:

```cpp
// data_from_string.hpp

enum class DATA_TYPE {
    INTEGER,
    FLOAT,
    STRING,
    DATE_TIME
};

DATA_TYPE parseDataType(const std::string& input);
```

So, instead of `STRING` as we expected before, `DATE_TIME` can be returned. In our code, we’ll start seeing `Unknown!`
appear in some cases.

This issue can be caught by unit tests (if we considered more than just the initial three cases), but it could also potentially
reach QA or even customers.

## Making Code to Fail Early

With an assumption that we use `-Wall -Werror` flags, we can do the following trick: [remove `default:` branch][1].

```cpp
...
    switch (type) {
        case DATA_TYPE::INTEGER: {
            std::cout << "Integer!";
            break;
        }
        case DATA_TYPE::FLOAT: {
            std::cout << "Float!";
            break;
        }
        case DATA_TYPE::STRING: {
            std::cout << "String!";
            break;
        }
        // No default branch
...
```

So, when a new value is added to `DATA_TYPE`, we’ll get a compilation error like this:

```
main.cpp: In function ‘void printDataType(const std::string&)’:
main.cpp:15:12: error: enumeration value ‘DATE_TIME’ not handled in switch [-Werror=switch]
   15 |     switch (type) {
      |            ^
```

We immediately know that this upgrade will require some additional work.

## Pros

A bug in the code causes a compilation error, so it most likely doesn't even reach CI, reducing the time to bug discovery.

## Cons

 * It can be overwhelming to implement if the `enum class` has a lot of values, and most of the values aren't used.
 * It requires your code to be compiled with `-Wall -Werror`, which may be problematic for legacy projects.

## Conclusion

We are always taught to write a `default:` branch in all our `switch` statements. This is good general practice, but as
we saw in this example, there are cases when removing it can help save time and avoid bug handling.

## Happy Coding!

[1]: https://godbolt.org/#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,selection:(endColumn:1,endLineNumber:36,positionColumn:1,positionLineNumber:36,selectionStartColumn:1,selectionStartLineNumber:36,startColumn:1,startLineNumber:36),source:'%23include+%3Ciostream%3E%0A%0Aenum+class+DATA_TYPE+%7B%0A++++INTEGER,%0A++++FLOAT,%0A++++STRING,%0A++++DATE_TIME%0A%7D%3B%0A%0ADATA_TYPE+parseDataType(const+std::string%26+input)+%7B%0A++++//+Just+to+return+something%0A++++return+DATA_TYPE::STRING%3B+%0A%7D%0A%0Avoid+printDataType(const+std::string%26+input)+%7B%0A++++const+DATA_TYPE+type%7BparseDataType(input)%7D%3B%0A%0A++++switch+(type)+%7B%0A++++++++case+DATA_TYPE::INTEGER:+%7B%0A++++++++++++std::cout+%3C%3C+%22Integer!!%22%3B%0A++++++++++++break%3B%0A++++++++%7D%0A++++++++case+DATA_TYPE::FLOAT:+%7B%0A++++++++++++std::cout+%3C%3C+%22Float!!%22%3B%0A++++++++++++break%3B%0A++++++++%7D%0A++++++++case+DATA_TYPE::STRING:+%7B%0A++++++++++++std::cout+%3C%3C+%22String!!%22%3B%0A++++++++++++break%3B%0A++++++++%7D%0A++++%7D%0A%0A++++std::cout+%3C%3C+std::endl%3B%0A%7D%0A%0A%0Aint+main()+%7B%0A%0A%7D'),l:'5',n:'1',o:'C%2B%2B+source+%231',t:'0')),k:40.203336580561015,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:compiler,i:(compiler:g142,filters:(b:'0',binary:'1',binaryObject:'1',commentOnly:'0',debugCalls:'1',demangle:'0',directives:'0',execute:'1',intel:'0',libraryCode:'0',trim:'1',verboseDemangling:'0'),flagsViewOpen:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,libs:!(),options:'-std%3Dc%2B%2B17+-O0+-ggdb++-Wall+-Werror',overrides:!(),selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:1),l:'5',n:'0',o:'+x86-64+gcc+14.2+(Editor+%231)',t:'0')),header:(),k:35.40591912993099,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:executor,i:(argsPanelShown:'1',compilationPanelShown:'0',compiler:g93,compilerName:'',compilerOutShown:'0',execArgs:'',execStdin:'',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,libs:!(),options:'-std%3Dc%2B%2B17+-O0+-ggdb',overrides:!(),runtimeTools:!(),source:1,stdinPanelShown:'1',tree:0,wrap:'1'),l:'5',n:'0',o:'Executor+x86-64+gcc+9.3+(C%2B%2B,+Editor+%231)',t:'0')),k:24.39074428950802,l:'4',n:'0',o:'',s:0,t:'0')),l:'2',n:'0',o:'',t:'0')),version:4
