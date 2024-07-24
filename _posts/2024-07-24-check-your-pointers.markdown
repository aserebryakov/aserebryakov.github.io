---
layout: post
title:  "How to Dereference a nullptr without Crashing or Why You Should Check Pointers for nullptr"
date:   2024-07-24 12:00:47 +0300
categories: jekyll update
---

## It "works".

Let's say that there is a factory in your code that produces objects basing on the requested type:

{% highlight cpp %}
enum FOO_TYPE {
    TYPE_A = 0,
    TYPE_B
};


class FooFactory {
    public:
        std::unique_ptr<Foo> createFoo(const FOO_TYPE type) { 
            switch (type) {
                case TYPE_A:
                    return std::make_unique<FooA>();
                case TYPE_B:
                    return std::make_unique<FooB>();
                default:
                    return nullptr;
            }
            return nullptr; 
        }
};
{% endhighlight %}

This looks pretty legitimate and does the job. If incorrect type was provided
factory returns nullptr.

The type of the produced object may be e.g. read from e.g. network buffer that
is assumed to convert integer to `FOO_TYPE` by some legacy code in your code base
so for some reason it doesn't validate that integer value is actually
corresponding to `FOO_TYPE`.

Let's take a look on our `Foo`, `FooA`, `FooB`, implementations

{% highlight cpp %}
class Foo {
    public:
        Foo(const int f) : foo{f} {}

        int getSomeValue() const { return 42; }

    private:
        int foo;
};


class FooA : public Foo {
    public:
        FooA() : Foo(314) {}

    private:
        int bar{7};
};


class FooB : public Foo {
    public:
        FooB() : Foo(27) {}

    private:
        std::string baz{"some data"};
};
{% endhighlight %}

Using inheritance to inherit data isn't the best pattern, but we still can face it in a wild.

Factory client code can look like this:

{% highlight cpp %}
FooFactory f{};
std::cout << "TYPE_A: " << f.createFoo(TYPE_A)->getSomeValue() << std::endl;
std::cout << "TYPE_B: "<< f.createFoo(TYPE_B)->getSomeValue() << std::endl;
std::cout << "TYPE_?: "<< f.createFoo(static_cast<FOO_TYPE>(TYPE_B + 1))->getSomeValue() << std::endl;
{% endhighlight %}

In the case when input is correct, everything works as expected:

```
TYPE_A: 42
TYPE_B: 42
TYPE_?: 42
```

But wait... Something is wrong. Shouldn't we get a crash here because of
`nullptr` dereference?

{% highlight cpp %}
f.createFoo(static_cast<FOO_TYPE>(TYPE_B + 1))->getSomeValue()
{% endhighlight %}

## No, it doesn't

But the `Foo` code contains one thing that can be easily missed: `Foo::getSomeValue()`
method always returns a constant value and doesn't use `this` to access data members.

{% highlight cpp %}
int getSomeValue() const { return 42; }
{% endhighlight %}

Assembly generated for this function:

{% highlight assembly %}
Foo::getSomeValue() const:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     eax, 42 <-- Putting constant to a return value
        pop     rbp
        ret
{% endhighlight %}

A single little change makes it crash - accessing the data member:

{% highlight cpp %}
int getSomeValue() const { return foo; }
{% endhighlight %}

the expected crash appears:

```
Program returned: 139
Program stdout

TYPE_A: 314
TYPE_B: 27

Program stderr
Program terminated with signal: SIGSEGV
```

Let's take a look on assembly this time:

{% highlight assembly %}
Foo::getSomeValue() const:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     eax, DWORD PTR [rax] <---- member access
        pop     rbp
        ret
{% endhighlight %}

## Conclusion

This kind of issue may be really confusing, especially because on the first
sight code "works".

This is the reason why in such cases `std::optional` or `std::expected` should be
used instead of returning `nullptr` from a function in case of unexpected input.

You may find a complete code example by the following [link](
https://godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGISQAcpK4AMngMmAByPgBGmMQSGqQADqgKhE4MHt6%2B/kGp6Y4CYRHRLHEJXEl2mA6ZQgRMxATZPn6Btpj2RQwNTQQlUbHxibaNza25HQrjA%2BFD5SNVAJS2qF7EyOwc5gDM4cjeWADUJrtuTjPEmKxn2CYaAIJ7B0eYp%2BdsLCQAnncPzzM%2BwYhy8JzObiu4WA/yeAIBhyYCgUxwAYqhUKcAOxWJ7HfHHZJeGK0PDIEAAglUtEYiBoBgzY7hAjHKjLY4gVkYkw4qg8gAi2KsWP58Lx1PxzOOwEwwlQbAAamIvJgIOz6YyeRZjtcCBsGMdJGYztqBZSJccAPSWpmCaWyoTyzBK7yq9UCTU4nWy/Vc1Am7GiuHi/HJYh4ABuTAI7HNEql/H9u1xzxFJrFzyeiORNNQjw5hOJpOQuaFccLJLJFJD1PRebVBbrEF2UnZWrNwfzBLDkejsZrVKlMSaWqxZuT8LTE87CIMObr2s5RMrJbrZZry%2BL1a7EoXDc5TbMWLbOI7mZ3hPDUZj24txxm6BAIChRmOw4AXu2zGYFE7juhoyYcxjSnFNxzAztGB8NEAHkYIAfQAFQATWUbACyldsaxQtD4PzM5BSScscOweCwNAjNZyRFE61RJgHF%2BdcL03KtyypB8ny8Bg8AARxVeDkgIYgITrO5jmQa4%2BybDUWVROCkNQ9CCB%2BZJMBPU0BwtBQAHdCGQBBjggZTVPUti7wJUQlGOEi8Nvcz7N1X0OJAFgmAAa0weCuN4lURIxZ5dmwNV000%2BzLPeGyLDs%2By70c4gDWc1yPK87i%2BMwPzUCsQLgunC8Yv/TAqCYLxaAIaL8upOKDQYEraEE4Tcvys98qq44atoOqhIDMz8TPcDKKeOSEJI%2B0CDrRCVLdJj2MaRwSylYz0pxGzU1FRqCVaoaFLQozJusaxlhC1aMylVzwgbLCL1o%2BiiGIH5WXbCia2ctAvBZCEIVOb8Vs5YCPjcT6qAAOgkm4YybFblgAWjuGU5UVZUpo%2B857wIR8QFcWgjpm9HXve85PuAyLfu/ZG3FZEHJPB2lIuh2GHSdF0VQugmUeczHsYJF71nxgGUaJxT4LOVESeNVnyeB0GpNpGZozJeDLIIET5JIu4IEir7tS4ZY6cCuHHQR10Wb58n2YYdAsdy8s6wAKj9ASupxdrOuEp6L25t7/sBjEHeIGG9YZw3mbbcXUfRjmrZrVqNCOgUOFWWhOAAVl4PxuF4VBOAByxrHvdZNnePYeFIMqOC0HXSDckBdgANiBjRdjMGuk40SQuF2FsuDMfROEkVPNAzzheAUEAklL8vSDgWAkDQFhkjoeJyEoWf5/oBJgAATl2Pg6BjYgR4gGIB9IGJwiaH5OGL0/mDumCYm0WpS%2BL2e2EEGCGFoC%2By94LAYi8YA3BiFoCPdOpAsCuSMOIb%2BYC8DXDqBGTAICtDBFULUN62xi7Mi6MfUkMRiDnw8FgY%2BQk8AsEvrwBBxAYhpEwPyTAEDgCkiMAPVYVADDAAUAqPAmBtIwVUmnYu/BBAiDEOwKQMhBCKBUOoaBuguD6EMMYfalh9B4BiCPSAqxUCCUyCAqGD4CKmBzpYLgWJjhQxghocxwBgDoBiOYqgqliAsDwMiSMmAM6UPDFgDRapOjdEyC4c2kw/DyNCPMMoFQ9AFAyAIEJ0S0ixIYIMSJSx/GP3qLMeJ8iah1AEH0ZoKThiVDGP0bJpTCkROKRIVYv4NhbBqT3DgKdSBp2QZnDgxxVABBrlDGukhpTIBLBvIGuxDK4EICQL6LZli8HHvHVYCAbhYASH4quuwAhAy4FwHpLcsRYhrlib8kgN5NL7q04%2BHTh6jxLiwyeMBEAY1QcgN6JAl4QCaBw5QhguhCAQKgbSAjeArzoHLAQ3yIi0D%2BQCtpwL5SrxGJvbeIK16RFYNsOFc8F7EBgm9aFgLj6YGeY8YgHCh4oNqA0fAadeBCOEKIcQ4i6VSLUMfORCjmFGKsCo3BvitE6I9JwfRaNDHKIsKY8xljrG2JiJ4%2BI3jEHwFWMQLijg2CIQxMApVax6liLGNSiFvz/kEtAUJTA7AkjaXwckchCdk792gR07AzzXnEC6T0vpAzgBDOOCMsZEBs7cpsMcCZt1pnazmSw1YVczBJyBgEQINcN4BACBoDeG8a410bmch17TyUjzHlGppZhc2Dw4JG7%2BFdKHpGcJIIAA%3D%3D
)
