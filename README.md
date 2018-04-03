
## How to Install 
```bash
git clone https://github.com/HsiehShuJeng/esmre.git ${desired folder name}
cd ${desired folder name}
python3 ./setup.py install
```

## Notes  
For installing `esmre` with success in Python3, some part in `esm.c` and `esmre.c` needs to be modified for warding off confronting the following exception:  
```bash
src/esm.c:38:11: error: no member named 'ob_type' in 'esm_IndexObject'
    self->ob_type->tp_free((PyObject*) self);
    ~~~~  ^
src/esm.c:185:5: warning: suggest braces around initialization of subobject
      [-Wmissing-braces]
    PyObject_HEAD_INIT(NULL)
    ^~~~~~~~~~~~~~~~~~~~~~~~
/Library/Frameworks/Python.framework/Versions/3.6/include/python3.6m/object.h:87:5: note: 
      expanded from macro 'PyObject_HEAD_INIT'
    1, type },
    ^~~~~~~
src/esm.c:187:5: warning: incompatible pointer to integer conversion
      initializing 'Py_ssize_t' (aka 'long') with an expression of type
      'char [10]' [-Wint-conversion]
    "esm.Index",                        /*tp_name*/
    ^~~~~~~~~~~
src/esm.c:190:5: warning: incompatible function pointer types initializing
      'printfunc' (aka 'int (*)(struct _object *, struct __sFILE *, int)') with
      an expression of type 'destructor' (aka 'void (*)(struct _object *)')
      [-Wincompatible-function-pointer-types]
    (destructor) esm_Index_dealloc,     /*tp_dealloc*/
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
src/esm.c:205:5: warning: incompatible integer to pointer conversion
      initializing 'const char *' with an expression of type 'unsigned long'
      [-Wint-conversion]
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE, /*tp_flags*/
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/Library/Frameworks/Python.framework/Versions/3.6/include/python3.6m/object.h:658:29: note: 
      expanded from macro 'Py_TPFLAGS_DEFAULT'
#define Py_TPFLAGS_DEFAULT  ( \
                            ^
src/esm.c:206:5: warning: incompatible pointer types initializing 'traverseproc'
      (aka 'int (*)(struct _object *, int (*)(struct _object *, void *), void
      *)') with an expression of type 'char [47]' [-Wincompatible-pointer-types]
    "Index() -> new efficient string matching index",  /* tp_doc */
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
src/esm.c:213:5: warning: incompatible pointer types initializing
      'struct PyMemberDef *' with an expression of type 'PyMethodDef [4]'
      [-Wincompatible-pointer-types]
    esm_Index_methods,             /* tp_methods */
    ^~~~~~~~~~~~~~~~~
src/esm.c:214:5: warning: incompatible pointer types initializing
      'struct PyGetSetDef *' with an expression of type 'PyMemberDef [1]'
      [-Wincompatible-pointer-types]
    esm_Index_members,             /* tp_members */
    ^~~~~~~~~~~~~~~~~
src/esm.c:221:5: warning: incompatible function pointer types initializing
      'allocfunc' (aka 'struct _object *(*)(struct _typeobject *, long)') with
      an expression of type 'initproc' (aka 'int (*)(struct _object *, struct
      _object *, struct _object *)') [-Wincompatible-function-pointer-types]
    (initproc) esm_Index_init,      /* tp_init */
    ^~~~~~~~~~~~~~~~~~~~~~~~~
src/esm.c:223:5: warning: incompatible pointer types initializing 'freefunc'
      (aka 'void (*)(void *)') with an expression of type 'PyObject
      *(PyTypeObject *, PyObject *, PyObject *)' (aka 'struct _object *(struct
      _typeobject *, struct _object *, struct _object *)')
      [-Wincompatible-pointer-types]
    esm_Index_new,                 /* tp_new */
    ^~~~~~~~~~~~~
src/esm.c:239:9: error: non-void function 'initesm' should return a value
      [-Wreturn-type]
        return;
        ^
src/esm.c:241:9: warning: implicit declaration of function 'Py_InitModule3' is
      invalid in C99 [-Wimplicit-function-declaration]
    m = Py_InitModule3("esm", esm_methods,
        ^
src/esm.c:241:9: warning: this function declaration is not a prototype
      [-Wstrict-prototypes]
src/esm.c:241:7: warning: incompatible integer to pointer conversion assigning
      to 'PyObject *' (aka 'struct _object *') from 'int' [-Wint-conversion]
    m = Py_InitModule3("esm", esm_methods,
      ^ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
src/esm.c:245:9: error: non-void function 'initesm' should return a value
      [-Wreturn-type]
        return;
        ^
12 warnings and 3 errors generated.
error: command '/usr/bin/clang' failed with exit status 1
```

## Description  
**esmre - Efficient String Matching Regular Expressions**  
esmre is a Python module that can be used to speed up the execution of a large
collection of regular expressions. It works by building a index of compulsory
substrings from a collection of regular expressions, which it uses to quickly
exclude those expressions which trivially do not match each input.  

Here is some example code that uses esmre:  
```python
>>> import esmre
>>> index = esmre.Index()
>>> index.enter(r"Major-General\W*$", "savoy opera")
>>> index.enter(r"\bway\W+haye?\b", "sea shanty")
>>> index.query("I am the very model of a modern Major-General.")
['savoy opera']
>>> index.query("Way, hay up she rises,")
['sea shanty']
>>> 
```

The esmre module builds on the simpler string matching facilities of the esm
module, which wraps a C implementation some of the algorithms described in
Aho's and Corasick's paper on efficient string matching [Aho, A.V, and
Corasick, M. J. Efficient String Matching: An Aid to Bibliographic Search.
Comm. ACM 18:6 (June 1975), 333-340]. Some minor modifications have been made
to the algorithms in the paper and one algorithm is missing (for now), but
there is enough to implement a quick string matching index.

Here is some example code that uses esm directly:  
```python
>>> import esm
>>> index = esm.Index()
>>> index.enter("he")
>>> index.enter("she")
>>> index.enter("his")
>>> index.enter("hers")
>>> index.fix()
>>> index.query("this here is history")
[((1, 4), 'his'), ((5, 7), 'he'), ((13, 16), 'his')]
>>> index.query("Those are his sheep!")
[((10, 13), 'his'), ((14, 17), 'she'), ((15, 17), 'he')]
>>> 
```

You can see more usage examples in the tests.
