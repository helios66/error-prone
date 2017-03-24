---
title: StaticQualifiedUsingExpression
summary: A static variable or method should be qualified with a class name, not expression
layout: bugpattern
category: JDK
severity: WARNING
---

<!--
*** AUTO-GENERATED, DO NOT MODIFY ***
To make changes, edit the @BugPattern annotation or the explanation in docs/bugpattern.
-->

_Alternate names: static, static-access, StaticAccessedFromInstance_

## The problem
To refer to a static member of another class, we typically *qualify* that member
name by prepending the name of the class it's in, and a dot:
`TheClass.theMethod()`.

But the Java language also permits you to qualify this call using any
*expression* (typically, a variable) whose static type is the class that
contains the method: `instanceOfTheClass.theMethod()`.

Doing this creates the appearance of an ordinary polymorphic method call, but it
behaves very differently. For example:

```
public class Main {
  static class TheClass {
    public static int theMethod() {
      return 1;
    }
  }

  static class TheSubclass extends TheClass {
    public static int theMethod() {
      return 2;
    }
  }

  public static void main(String[] args) {
    TheClass instanceOfTheClass = new TheSubclass();
    System.out.println(instanceOfTheClass.theMethod());
  }
}
```

`TheSubclass` appears to "override" `theMethod`, so we might expect this code to
print the number `2`.

The code, however, prints the number `1`. The runtime type of
`instanceOfTheClass`, `TheSubclass`, is ignored; only the static type of the
reference, as seen by `javac`, matters.

In fact, the instance that `instanceOfTheClass` points to is *entirely*
irrelevant. To prove this, set the variable to `null` and run again. The program
will *still* print `1`, not throw a `NullPointerException`!

Qualifying a static reference in this way creates an unnecessarily confusing
situation. To prevent it, always qualify static method calls using a class name,
never an expression.

## Suppression
Suppress false positives by adding an `@SuppressWarnings("StaticQualifiedUsingExpression")` annotation to the enclosing element.

----------

### Positive examples
__StaticQualifiedUsingExpressionPositiveCase1.java__

{% highlight java %}
/*
 * Copyright 2012 Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.google.errorprone.bugpatterns.testdata;

import java.math.BigDecimal;

/** @author eaftan@google.com (Eddie Aftandilian) */
class MyClass {

  static int STATIC_FIELD = 42;

  static int staticMethod() {
    return 42;
  }

  int FIELD = 42;

  int method() {
    return 42;
  }

  static class StaticInnerClass {
    static final MyClass myClass = new MyClass();
  }
}

class MyStaticClass {
  static MyClass myClass = new MyClass();
}

public class StaticQualifiedUsingExpressionPositiveCase1 {

  public static int staticVar1 = 1;
  private StaticQualifiedUsingExpressionPositiveCase1 next;

  public static int staticTestMethod() {
    return 1;
  }

  public static Object staticTestMethod2() {
    return new Object();
  }

  public static Object staticTestMethod3(Object x) {
    return null;
  }

  public void test1() {
    StaticQualifiedUsingExpressionPositiveCase1 testObj =
        new StaticQualifiedUsingExpressionPositiveCase1();
    int i;

    // BUG: Diagnostic contains: variable staticVar1
    // i = StaticQualifiedUsingExpressionPositiveCase1.staticVar1
    i = this.staticVar1;
    // BUG: Diagnostic contains: variable staticVar1
    // i = StaticQualifiedUsingExpressionPositiveCase1.staticVar1
    i = testObj.staticVar1;
    // BUG: Diagnostic contains: variable staticVar1
    // i = StaticQualifiedUsingExpressionPositiveCase1.staticVar1
    i = testObj.next.next.next.staticVar1;
  }

  public void test2() {
    int i;
    Integer integer = new Integer(1);
    // BUG: Diagnostic contains: variable MAX_VALUE
    // i = Integer.MAX_VALUE
    i = integer.MAX_VALUE;
  }

  public void test3() {
    String s1 = new String();
    // BUG: Diagnostic contains: method valueOf
    // String s2 = String.valueOf(10)
    String s2 = s1.valueOf(10);
    // BUG: Diagnostic contains: method valueOf
    // s2 = String.valueOf(10)
    s2 = new String().valueOf(10);
    // BUG: Diagnostic contains: method staticTestMethod
    // int i = staticTestMethod()
    int i = this.staticTestMethod();
    // BUG: Diagnostic contains: method staticTestMethod2
    // String s3 = staticTestMethod2().toString
    String s3 = this.staticTestMethod2().toString();
    // BUG: Diagnostic contains: method staticTestMethod
    // i = staticTestMethod()
    i = this.next.next.next.staticTestMethod();
  }

  public void test4() {
    BigDecimal decimal = new BigDecimal(1);
    // BUG: Diagnostic contains: method valueOf
    // BigDecimal decimal2 = BigDecimal.valueOf(1)
    BigDecimal decimal2 = decimal.valueOf(1);
  }

  public static MyClass hiding;

  public void test5(MyClass hiding) {
    // BUG: Diagnostic contains: method staticTestMethod3
    // Object o = staticTestMethod3(this.toString())
    Object o = this.staticTestMethod3(this.toString());
    // BUG: Diagnostic contains: variable myClass
    // x = StaticInnerClass.myClass.FIELD;
    int x = new MyClass.StaticInnerClass().myClass.FIELD;
    // BUG: Diagnostic contains: variable STATIC_FIELD
    // x = MyClass.STATIC_FIELD;
    x = new MyClass.StaticInnerClass().myClass.STATIC_FIELD;
    // BUG: Diagnostic contains: variable hiding
    // StaticQualifiedUsingExpressionPositiveCase1.hiding = hiding;
    this.hiding = hiding;
    // BUG: Diagnostic contains: variable STATIC_FIELD
    // x = MyClass.STATIC_FIELD;
    x = MyStaticClass.myClass.STATIC_FIELD;
    // BUG: Diagnostic contains: method staticMethod
    // x = MyClass.staticMethod();
    x = MyStaticClass.myClass.staticMethod();

    x = MyStaticClass.myClass.FIELD;
    x = MyStaticClass.myClass.method();
  }

  static class Bar {
    static int baz = 0;

    static int baz() {
      return 42;
    }
  }

  static class Foo {
    static Bar bar;
  }

  static void test6() {
    Foo foo = new Foo();
    // BUG: Diagnostic contains: method baz
    // x = Bar.baz();
    int x = Foo.bar.baz();
    Bar bar = Foo.bar;
    // BUG: Diagnostic contains: variable bar
    // bar = Foo.bar;
    bar = foo.bar;
    // BUG: Diagnostic contains: variable baz
    // x = Bar.baz;
    x = Foo.bar.baz;
  }

  static class C<T extends String> {
    static int foo() {
      return 42;
    }
  }

  public void test7() {
    // BUG: Diagnostic contains: method foo
    // x = C.foo();
    int x = new C<String>().foo();
  }
}
{% endhighlight %}

__StaticQualifiedUsingExpressionPositiveCase2.java__

{% highlight java %}
/*
 * Copyright 2012 Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.google.errorprone.bugpatterns.testdata;


/** @author eaftan@google.com (Eddie Aftandilian) */
public class StaticQualifiedUsingExpressionPositiveCase2 {

  private static class TestClass {
    public static int staticTestMethod() {
      return 1;
    }
  }

  public int test1() {
    // BUG: Diagnostic contains: method staticTestMethod
    // return TestClass.staticTestMethod()
    return new TestClass().staticTestMethod();
  }
}
{% endhighlight %}

### Negative examples
__StaticQualifiedUsingExpressionNegativeCases.java__

{% highlight java %}
/*
 * Copyright 2012 Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.google.errorprone.bugpatterns.testdata;


/** @author eaftan@google.com (Eddie Aftandilian) */
public class StaticQualifiedUsingExpressionNegativeCases {

  public static int staticVar1 = 1;

  public static void staticTestMethod() {}

  public void test1() {
    Integer i = Integer.MAX_VALUE;
    i = Integer.valueOf(10);
  }

  public void test2() {
    int i = staticVar1;
    i = StaticQualifiedUsingExpressionNegativeCases.staticVar1;
  }

  public void test3() {
    test1();
    this.test1();
    new StaticQualifiedUsingExpressionNegativeCases().test1();
    staticTestMethod();
  }

  public void test4() {
    Class<?> klass = String[].class;
  }

  @SuppressWarnings("static")
  public void testJavacAltname() {
    this.staticTestMethod();
  }

  @SuppressWarnings("static-access")
  public void testEclipseAltname() {
    this.staticTestMethod();
  }
}
{% endhighlight %}
