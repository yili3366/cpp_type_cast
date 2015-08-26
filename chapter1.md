# First Chapter

## 1 Dynamic cast


- The result of the expression dynamic_cast&lt;T&gt;(v) is the result of converting the expression v to type T. T
shall be a pointer or reference to a complete class type, or “pointer to cv void.” The dynamic_cast operator
shall not cast away constness (5.2.11).

- If T is a pointer type, v shall be a prvalue of a pointer to complete class type, and the result is a prvalue
of type T. If T is an lvalue reference type, v shall be an lvalue of a complete class type, and the result is
an lvalue of the type referred to by T. If T is an rvalue reference type, v shall be an expression having a
complete class type, and the result is an xvalue of the type referred to by T.

- If the type of v is the same as T, or it is the same as T except that the class object type in T is more
cv-qualified than the class object type in v, the result is v (converted if necessary).

- If the value of v is a null pointer value in the pointer case, the result is the null pointer value of type T.

- If T is “pointer to *cv1* B” and v has type “pointer to *cv2* D” such that B is a base class of D, the result is a
pointer to the unique B subobject of the D object pointed to by v. Similarly, if T is “reference to cv1 B” and
v has type cv2 D such that B is a base class of D, the result is the unique B subobject of the D object referred
to by v. 68 The result is an lvalue if T is an lvalue reference, or an xvalue if T is an rvalue reference. In
both the pointer and reference cases, the program is ill-formed if cv2 has greater cv-qualification than cv1
or if B is an inaccessible or ambiguous base class of D. [ Example:

<pre><code>    struct B { };
    struct D : B { };
    void foo(D* dp) {
        B* bp = dynamic_cast&lt;B*&gt;(dp); // equivalent to B* bp = dp;
    }
</code></pre> 

-- end example ]


- Otherwise, v shall be a pointer to or a glvalue of a polymorphic type (10.3).

- If T is “pointer to cv void,” then the result is a pointer to the most derived object pointed to by v. Otherwise, a run-time check is applied to see if the object pointed or referred to by v can be converted to the type pointed or referred to by T.

- If C is the class type to which T points or refers, the run-time check logically executes as follows:
-- If, in the most derived object pointed (referred) to by v, v points (refers) to a public base class subobject of a C object, and if only one object of type C is derived from the subobject pointed (referred) to by v the result points (refers) to that C object.
-- Otherwise, if v points (refers) to a public base class subobject of the most derived object, and the type of the most derived object has a base class, of type C, that is unambiguous and public, the result points (refers) to the C subobject of the most derived object.
-- Otherwise, the run-time check fails.

- The value of a failed cast to pointer type is the null pointer value of the required result type. A failed cast to reference type throws an exception (15.1) of a type that would match a handler (15.3) of type std::bad_cast (18.7.2).
[ Example:

<pre><code>
    class A { virtual void f(); };
    class B { virtual void g(); };
    class D : public virtual A, private B { };
    void g() {
        D d;
        B* bp = (B*)&d; // cast needed to break protection
        A* ap = &d; // public derivation, no cast needed
        D& dr = dynamic_cast&lt;D&&gt;(*bp); // fails
        ap = dynamic_cast&lt;A*&gt;(bp); // fails
        bp = dynamic_cast&lt;B*&gt;(ap); // fails
        ap = dynamic_cast&lt;A*&gt;(&d); // succeeds
        bp = dynamic_cast&lt;B*&gt;(&d); // ill-formed (not a run-time check)
    }

    class E : public D, public B { };
    class F : public E, public D { };
    void h() {
        F f;
        A* ap = &f; // succeeds: finds unique A
        D* dp = dynamic_cast&lt;D*&gt;(ap); // fails: yields 0
                                      // f has two D subobjects
        E* ep = (E*)ap;// ill-formed: cast from virtual base
        E* ep1 = dynamic_cast&lt;E*&gt;(ap); // succeeds
    }
</code></pre>

-- end example ] [ Note: 12.7 describes the behavior of a dynamic_cast applied to an object under construction or destruction. -- end note ]

## 2 Static cast
- The result of the expression static_cast&lt;T&gt;(v) is the result of converting the expression v to type T. If T is an lvalue reference type or an rvalue reference to function type, the result is an lvalue; if T is an rvalue reference to object type, the result is an xvalue; otherwise, the result is a prvalue. The static_cast operator shall not cast away constness (5.2.11).

- An lvalue of type "*cv1* **B**," where **B** is a class type, can be cast to type “reference to *cv2* **D**,” where **D** is a class derived (Clause 10) from B, if a valid standard conversion from "pointer to D" to "pointer to B" exists (4.10), *cv2* is the same cv-qualification as, or greater cv-qualification than, *cv1*, and B is neither a virtual base class of D nor a base class of a virtual base class of D. The result has type “cv2 D.” An xvalue of type "*cv1* **B**" may be cast to type "rvalue reference to *cv2* **D**" with the same constraints as for an lvalue of type “cv1 B.”
If the object of type “cv1 B” is actually a subobject of an object of type D, the result refers to the enclosing
object of type D. Otherwise, the behavior is undefined. [ Example:

<pre><code>
    struct B { };
    struct D : public B { };
    D d;
    B &br = d;

    static_cast&lt;D&&gt;(br); // produces lvalue to the original d object
</code></pre>

-- end example ]
- A glvalue of type “cv1 T1” can be cast to type “rvalue reference to cv2 T2” if “cv2 T2” is reference-compatible with “cv1 T1” (8.5.3). If the glvalue is not a bit-field, the result refers to the object or the specified base class subobject thereof; otherwise, the lvalue-to-rvalue conversion (4.1) is applied to the bit-field and the resulting prvalue is used as the expression of the static_cast for the remainder of this section. If T2 is an inaccessible (Clause 11) or ambiguous (10.2) base class of T1, a program that necessitates such a cast is ill-formed.

- An expression e can be explicitly converted to a type T using a static_cast of the form static_cast&lt;T&gt;(e) if the declaration T t(e); is well-formed, for some invented temporary variable t (8.5). The effect of such an explicit conversion is the same as performing the declaration and initialization and then using the temporary variable as the result of the conversion. The expression e is used as a glvalue if and only if the initialization uses it as a glvalue.

- Otherwise, the static_cast shall perform one of the conversions listed below. No other conversion shall be performed explicitly using a static_cast.

- Any expression can be explicitly converted to type cv void, in which case it becomes a discarded-value expression (Clause 5). [ Note: however, if the value is in a temporary object (12.2), the destructor for that object is not executed until the usual time, and the value of the object is preserved for the purpose of executing the destructor. — end note ]

- The inverse of any standard conversion sequence (Clause 4) not containing an lvalue-to-rvalue (4.1), array-
to-pointer (4.2), function-to-pointer (4.3), null pointer (4.10), null member pointer (4.11), or boolean (4.12)
conversion, can be performed explicitly using static_cast. A program is ill-formed if it uses static_cast
to perform the inverse of an ill-formed standard conversion sequence. [ Example:

<pre><code>
    struct B { };
    struct D : private B { };
    void f() {
    static_cast&lt;D*&gt;((B*)0); // Error: B is a private base of D.
    static_cast&lt;int B::*&gt;((int D::*)0); // Error: B is a private base of D.
}
</pre></code>

-- end example ]

- The lvalue-to-rvalue (4.1), array-to-pointer (4.2), and function-to-pointer (4.3) conversions are applied to the operand. Such a static_cast is subject to the restriction that the explicit conversion does not cast away constness (5.2.11), and the following additional rules for specific cases:

- A value of a scoped enumeration type (7.2) can be explicitly converted to an integral type. When that type
is cv bool, the resulting value is false if the original value is zero and true for all other values. For the remaining integral types, the value is unchanged if the original value can be represented by the specified type Otherwise, the resulting value is unspecified. A value of a scoped enumeration type can also be explicitly
converted to a floating-point type; the result is the same as that of converting from the original value to the floating-point type.

- A value of integral or enumeration type can be explicitly converted to an enumeration type. The value is unchanged if the original value is within the range of the enumeration values (7.2). Otherwise, the resulting value is unspecified (and might not be in that range). A value of floating-point type can also be converted to an enumeration type. The resulting value is the same as converting the original value to the underlying type of the enumeration (4.9), and subsequently to the enumeration type.
A prvalue of type “pointer to cv1 B,” where B is a class type, can be converted to a prvalue of type “pointer to cv2 D,” where D is a class derived (Clause 10) from B, if a valid standard conversion from “pointer to D” to “pointer to B” exists (4.10), cv2 is the same cv-qualification as, or greater cv-qualification than, cv1, and B is neither a virtual base class of D nor a base class of a virtual base class of D. The null pointer value (4.10) is converted to the null pointer value of the destination type. If the prvalue of type “pointer to cv1 B” points to a B that is actually a subobject of an object of type D, the resulting pointer points to the enclosing object of type D. Otherwise, the behavior is undefined.

- A prvalue of type “pointer to member of D of type cv1 T” can be converted to a prvalue of type “pointer to member of B” of type cv2 T, where B is a base class (Clause 10) of D, if a valid standard conversion from "pointer to member of B of type T" to “pointer to member of D of type T” exists (4.11), and cv2 is the same cv-qualification as, or greater cv-qualification than, cv1. The null member pointer value (4.11) is converted to the null member pointer value of the destination type. If class B contains the original member, or is a base or derived class of the class containing the original member, the resulting pointer to member points to the original member. Otherwise, the behavior is undefined. [ Note: although class B need not contain the
original member, the dynamic type of the object with which indirection through the pointer to member is performed must contain the original member; see 5.5. — end note ]

- A prvalue of type “pointer to cv1 void” can be converted to a prvalue of type “pointer to cv2 T,” where T is an object type and cv2 is the same cv-qualification as, or greater cv qualification than, cv1. The null pointer value is converted to the null pointer value of the destination type. If the original pointer value represents the address A of a byte in memory and A satisfies the alignment requirement of T, then the resulting pointer value represents the same address as the original pointer value, that is, A. The result of any other such pointer conversion is unspecified. A value of type pointer to object converted to “pointer to cv void” and back, possibly with different cv-qualification, shall have its original value. [ Example:
<pre><code>
T* p1 = new T;
const T* p2 = static_cast&lt;const T*&gt;(static_cast$lt;void*&gt;(p1));
bool b = p1 == p2; // b will have the value true.
</pre></code>
-- end example ]

## 3 reinterpret cast

- The result of the expression reinterpret_cast&lt;T&gt;(v) is the result of converting the expression v to type T. If T is an lvalue reference type or an rvalue reference to function type, the result is an lvalue; if T is an rvalue reference to object type, the result is an xvalue; otherwise, the result is a prvalue and the lvalue-to-rvalue (4.1), array-to-pointer (4.2), and function-to-pointer (4.3) standard conversions are performed on the expression v. Conversions that can be performed explicitly using reinterpret_cast are listed below. No other conversion can be performed explicitly using reinterpret_cast.
- The reinterpret_cast operator shall not cast away constness (5.2.11). An expression of integral, enumeration, pointer, or pointer-to-member type can be explicitly converted to its own type; such a cast yields the value of its operand.
- [ Note: The mapping performed by reinterpret_cast might, or might not, produce a representation different from the original value. — end note ]

- A pointer can be explicitly converted to any integral type large enough to hold it. The mapping function is implementation-defined. [ Note: It is intended to be unsurprising to those who know the addressing structure of the underlying machine. — end note ] A value of type std::nullptr_t can be converted to an integral type; the conversion has the same meaning and validity as a conversion of (void*)0 to the integral type. [ Note: A reinterpret_cast cannot be used to convert a value of any type to the type std::nullptr_t. -- end note ]

- A value of integral type or enumeration type can be explicitly converted to a pointer. A pointer converted to an integer of sufficient size (if any such exists on the implementation) and back to the same pointer type will have its original value; mappings between pointers and integers are otherwise implementation-defined.
[ Note: Except as described in 3.7.4.3, the result of such a conversion will not be a safely-derived pointer
value. — end note ]

- A function pointer can be explicitly converted to a function pointer of a different type. The effect of calling a function through a pointer to a function type (8.3.5) that is not the same as the type used in the definition of the function is undefined. Except that converting a prvalue of type “pointer to T1” to the type “pointer to T2” (where T1 and T2 are function types) and back to its original type yields the original pointer value, the result of such a pointer conversion is unspecified. [ Note: see also 4.10 for more details of pointer conversions.
-- end note ]

- An object pointer can be explicitly converted to an object pointer of a different type.72 When a prvalue v of object pointer type is converted to the object pointer type “pointer to cv T”, the result is static_cast&lt;cv T*&gt;(static_cast&lt;cv void*&gt;(v)). Converting a prvalue of type “pointer to T1” to the type “pointer to T2” (where T1 and T2 are object types and where the alignment requirements of T2 are no stricter than
those of T1) and back to its original type yields the original pointer value.
- Converting a function pointer to an object pointer type or vice versa is conditionally-supported. The meaning of such a conversion is implementation-defined, except that if an implementation supports conversions in both directions, converting a prvalue of one type to the other type and back, possibly with different cv-qualification, shall yield the original pointer value.

- The null pointer value (4.10) is converted to the null pointer value of the destination type. [ Note: A null pointer constant of type std::nullptr_t cannot be converted to a pointer type, and a null pointer constant of integral type is not necessarily converted to a null pointer value. — end note ]

- A prvalue of type “pointer to member of X of type T1” can be explicitly converted to a prvalue of a different type “pointer to member of Y of type T2” if T1 and T2 are both function types or both object types. The null member pointer value (4.11) is converted to the null member pointer value of the destination type. The result of this conversion is unspecified, except in the following cases:
-- converting a prvalue of type “pointer to member function” to a different pointer to member function type and back to its original type yields the original pointer to member value.
-- converting a prvalue of type “pointer to data member of X of type T1” to the type “pointer to data member of Y of type T2” (where the alignment requirements of T2 are no stricter than those of T1) and back to its original type yields the original pointer to member value.

- A glvalue expression of type T1 can be cast to the type “reference to T2” if an expression of type “pointer to T1” can be explicitly converted to the type “pointer to T2” using a reinterpret_cast. The result refers to the same object as the source glvalue, but with the specified type. [ Note: That is, for lvalues, a reference cast reinterpret_cast&lt;T&&gt;(x) has the same effect as the conversion *reinterpret_cast&lt;T*&gt;(&x) with
the built-in & and * operators (and similarly for reinterpret_cast&lt;T&&&gt;(x)). — end note ] No temporary is created, no copy is made, and constructors (12.1) or conversion functions (12.3) are not called.

## 4 Const cast
- The result of the expression const_cast&lt;T&gt;(v) is of type T. If T is an lvalue reference to object type, the result is an lvalue; if T is an rvalue reference to object type, the result is an xvalue; otherwise, the result is a prvalue and the lvalue-to-rvalue (4.1), array-to-pointer (4.2), and function-to-pointer (4.3) standard conversions are performed on the expression v. Conversions that can be performed explicitly using const_cast are listed below. No other conversion shall be performed explicitly using const_cast.

- [ Note: Subject to the restrictions in this section, an expression may be cast to its own type using a const_cast operator. — end note ]
For two pointer types T1 and T2 where
&emsp;&emsp;T1 is cv 1,0 pointer to cv 1,1 pointer to ... cv1,n−1 pointer to cv 1,n T
&emsp;&emsp;and
&emsp;&emsp;T2 is cv 2,0 pointer to cv 2,1 pointer to ... cv2,n−1 pointer to cv 2,n T
where T is any object type or the void type and where cv 1,k and cv 2,k may be different cv-qualifications, a prvalue of type T1 may be explicitly converted to the type T2 using a const_cast. The result of a pointer const_cast refers to the original object.

- For two object types T1 and T2, if a pointer to T1 can be explicitly converted to the type “pointer to T2”
using a const_cast, then the following conversions can also be made:
 - an lvalue of type T1 can be explicitly converted to an lvalue of type T2 using the cast const_cast&lt;T2&&gt;;
 - a glvalue of type T1 can be explicitly converted to an xvalue of type T2 using the cast const_cast&lt;T2&&&gt;; and
 - if T1 is a class type, a prvalue of type T1 can be explicitly converted to an xvalue of type T2 using the
cast const_cast&lt;T2&&&gt;.
The result of a reference const_cast refers to the original object.

- For a const_cast involving pointers to data members, multi-level pointers to data members and multi-level mixed pointers and pointers to data members (4.4), the rules for const_cast are the same as those used for pointers; the “member” aspect of a pointer to member is ignored when determining where the cv-qualifiers are added or removed by the const_cast. The result of a pointer to data member const_cast refers to the same member as the original (uncast) pointer to data member.
- A null pointer value (4.10) is converted to the null pointer value of the destination type. The null member pointer value (4.11) is converted to the null member pointer value of the destination type.
- [ Note: Depending on the type of the object, a write operation through the pointer, lvalue or pointer to data member resulting from a const_cast that casts away a const-qualifier75 may produce undefined behavior (7.1.6.1). -- end note ]

- The following rules define the process known as casting away constness. In these rules Tn and Xn represent
types. For two pointer types:
&emsp;&emsp; X1 is T1cv 1,1 &ast; ... cv1,N &ast; where T1 is not a pointer type
&emsp;&emsp; X2 is T2cv 2,1 &ast; ... cv2,M &ast; where T2 is not a pointer type
&emsp;&emsp; K is min(N, M )
casting from X1 to X2 casts away constness if, for a non-pointer type T there does not exist an implicit
conversion (Clause 4) from:
&emsp;&emsp;Tcv 1,(N −K+1) * cv 1,(N −K+2) &ast; ... cv 1,N &ast;
&emsp;to
&emsp;&emsp;Tcv 2,(M −K+1) * cv 2,(M −K+2) &ast; ... cv 2,M &ast;

- Casting from an lvalue of type T1 to an lvalue of type T2 using an lvalue reference cast or casting from an expression of type T1 to an xvalue of type T2 using an rvalue reference cast casts away constness if a cast from a prvalue of type “pointer to T1” to the type “pointer to T2” casts away constness.

- Casting from a prvalue of type “pointer to data member of X of type T1” to the type “pointer to data member of Y of type T2” casts away constness if a cast from a prvalue of type “pointer to T1” to the type “pointer to T2” casts away constness.

- For multi-level pointer to members and multi-level mixed pointers and pointer to members (4.4), the “member” aspect of a pointer to member level is ignored when determining if a const cv-qualifier has been cast away.

- [ Note: some conversions which involve only changes in cv-qualification cannot be done using const_cast. For instance, conversions between pointers to functions are not covered because such conversions lead to values whose use causes undefined behavior. For the same reasons, conversions between pointers to member functions, and in particular, the conversion from a pointer to a const member function to a pointer to a non-const member function, are not covered. -- end note ]
