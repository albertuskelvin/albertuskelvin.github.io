---
title: 'Obfuscating Python Scripts with PyArmor'
date: 2019-12-08
permalink: /posts/2019/12/code-obfuscation-with-pyarmor/
tags:
  - obfuscation
  - python
  - pyarmor
  - security
---

Basically, code obfuscation is a technique used to modify the source code so that it becomes difficult to understand but remains fully functional. The main objective is to protect intellectual properties and prevent hackers from reverse engineering a proprietary source code.

There are a lot of obfuscation tools for python. However, most of them are not maintained. In case of pyspark, not all of them can be executed with `spark-submit`. One of them is a tool called <a href="https://github.com/Falldog/pyconcrete">pyconcrete</a>.

<b>pyconcrete</b> is an obfuscator tool that encrypts python code and decrypts when imported. The extension of the obfuscated scripts is `.pye` (encrypted python). Unlike <a href="https://github.com/ga0/pyprotect">pyprotect</a> which has auto generated wrappers for the encrypted main scripts, <b>pyprotect</b> doesn't have something like it. To be precise, the generated wrapper is basically a `.py` file that calls the `.pye` file of the corresponding script.

Such a thing like this wrapper is quite critical in case of pyspark. Since pyspark needs to know the driver program (which has `.py` extension), the obfuscated codes should be able to accomodate this needs. Precisely, although the obfuscation modifies the original code, there would still be a need to have generated `.py` files. I think this is important as we also need to consider the capability of running the obfuscated code which is independent of the platform.

To resolve such an issue, I tried to package all the pyspark modules (along with the drivers) into an `.egg` file. Submitting the `.egg` file via the `--py-file` on `spark-submit` didn't work. Here's what's written in the <a href="https://spark.apache.org/docs/latest/submitting-applications.html#launching-applications-with-spark-submit">documentation</a>.

```
For Python applications, simply pass a .py file in the place of <application-jar> instead of a JAR, 
and add Python .zip, .egg or .py files to the search path with --py-files.
```

In my opinion, the above statement clearly states that the driver should only be a `.py` file, while the `.egg` file just denotes the required dependencies.

In this post I'm going to write about my experience in leveraging one of the python obfuscators, that is <a href="https://github.com/dashingsoft/pyarmor">pyarmor</a>.

I opt for pyarmor since besides of its most updated maintenance aspect, it also has several pros compared with other tools, such as <b>pyprotect</b> and <b>pyconcrete</b>. Here's what I found.

<b>Pros</b>
<ul>
<li>Capable of creating license for code. It enables us to set the expiration date</li>
<li>Provides more specific options for obfuscating certain scripts (exactly single module, single package, and many packages)</li>
<li>Provides private Global Capsule (different from each user)</li>
<li>Provides web UI for obfuscation, license, and bundling</li>
<li>No need to install any extra dependency</li>
<li>The server machines don’t need to have PyArmor installed</li>
<li>Detail documentation</li>
</ul>

<b>Cons</b>
<ul>
<li>We need to know about the OS used by the server first before deploying code. This is because the Runtime Package is platform dependent</li>
<li>All the trial version of PyArmor provides shared Global Capsule. This might be risky if the attackers are able to grasp the techniques used for obfuscation or even generate a new license</li>
<li>Doesn’t support if we want to specify certain scripts that should not be obfuscated. We actually can do this by leveraging single module obfuscation yet it might be too cumbersome for a large number of modules</li>
</ul>

According to the <a href="https://pyarmor.readthedocs.io/en/latest/index.html">official documentation</a>, pyarmor is a command line tool used to obfuscate Python scripts, bind obfuscated scripts to fixed machine, or expire obfuscated scripts. It supports Python 2.6, 2.7, and 3.

It protects Python scripts by the following ways:
<ul>
<li>Obfuscate code object to protect constants and literal strings</li>
<li>Obfuscate <i>co_code</i> of each function (code object) in runtime</li>
<li>Clear <i>f_locals</i> of frame as soon as code object completed execution</li>
<li>Verify the license file of obfuscated scripts while running it</li>
</ul>

Basically, pyarmor has two primary properties used to obfuscate and execute the obfuscated scripts. The first one is called <b>Global Capsule</b>, while the second one is called <b>Runtime Package</b>.

<b>Global Capsule</b> is created automatically after the pyarmor obfuscate (used to obfuscate scripts) command is executed. It is basically a file called `.pyarmor_capsule.zip` located in the HOME path. It is used as the core property to obfuscate the scripts. This capsule is only stored in the build machine, not used by the obfuscated scripts and should not be distributed to the end users. Please visit this <a href="https://pyarmor.readthedocs.io/en/latest/understand-obfuscated-scripts.html#global-capsule">link</a> for further information.

<b>Runtime Package</b> is required to run the obfuscated scripts. It is the only dependency of obfuscated scripts. All the scripts obfuscated by the same <b>Global Capsule</b> could be run using this package. Please visit this <a href="https://pyarmor.readthedocs.io/en/latest/understand-obfuscated-scripts.html#runtime-package">link</a> for further information.

Besides obfuscating and running the obfuscated scripts, pyarmor is also able to expire obfuscated scripts. This is done by leveraging a runtime file called `license.lic`. It is required to run the obfuscated scripts.

When executing pyarmor obfuscate, a default `license.lic` will be generated, which allows obfuscated scripts to run in any machine and never expired. In order to bind obfuscated scripts to fix machine or expire the obfuscated scripts, use command `pyarmor licenses` to generate a new `license.lic` and overwrite the default one.

<h3>How to Use</h3>

In pyarmor, entry script is the first obfuscated script to be run or to be imported in a Python interpreter process. For example, `__init__.py` is the entry script if only one single Python package is obfuscated.

The first two lines in the entry script is called bootstrap code. This code only exists in the entry script. Below is the example of bootstrap code.

```python
from pytransform import pyarmor_runtime
pyarmor_runtime()
```

If the <b>Runtime Package</b> is not located in the same directory as the obfuscated scripts, we can just pass it as a parameter like the following.

```python
from pytransform import pyarmor_runtime
pyarmor_runtime('/path/to/runtime')
```

Let's take a look at a simple example.

Suppose that there's only a Python script (`myscript.py`) in our project directory. This simply will be the entry script.

To obfuscate `myscript.py`, we use command `pyarmor obfuscate myscript.py`. Doing so, pyarmor obfuscates the script and does the followings:

<ul>
<li>Obfuscates <i>myscript.py</i> and all the <i>*.py</i> in the same folder</li>
<li>Creates <i>.pyarmor_capsule.zip</i> in the HOME folder if it doesn't exist</li>
<li>Creates a folder <i>dist</i> in the same folder as the script if it doesn't exist</li>
<li>Writes the obfuscated <i>myscript.py</i> and all the obfuscated <i>*.py</i> (in the same folder as <i>myscript.py</i>) to the <i>dist</i> folder</li>
<li>Copies <b>Runtime Package</b> used to run the obfuscated scripts to the <i>dist</i> folder</li>
</ul>

By now the content of the <i>dist</i> folder would look like the following.

```
dist/
    myscript.py

    pytransform/
        __init__.py
        _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
        pytransform.key
        license.lic
```

To run the obfuscated scripts, we use the following commands.

```
cd dist
python myscript.py
```

By default, only the `*.py` in the same path as the entry script are obfuscated. To obfuscate all the `*.py` in the sub-folder recursively, execute the following command:

```
pyarmor obfuscate --recursive myscript.py
```

The above approach works by obfuscating the entry script and all the `*.py` files in the same folder as the entry script. We can actually use more advanced approach, such as obfuscating <a href="https://pyarmor.readthedocs.io/en/latest/usage.html#obfuscating-single-module">single module</a>, <a href="https://pyarmor.readthedocs.io/en/latest/usage.html#obfuscating-whole-package">single package</a>, and <a href="https://pyarmor.readthedocs.io/en/latest/advanced.html#obfuscating-many-packages">many packages</a>.

Last but not least, here's how it works in general.

<h3>How It Works Generally</h3>

Suppose we have the following simple script to obfuscate.

```python
# myscript.py

from pyspark.sql import SparkSession


def func_a(df):
	...

def func_b(df):
	...

spark = SparkSession().builder.appName('myscript').getOrCreate()

df = spark.createDataFrame(...)

df = func_a(df)
df = func_b(df)

df.show()
```

From the above code snippet, we can see that there are three code objects, namely `myscript.py`, `func_a`, and `func_b`.

According to the <a href="https://pyarmor.readthedocs.io/en/latest/how-to-do.html">documentation</a>, here's how pyarmor does the obfuscation.

Before being obfuscated, each code object is transformed into the following (wrapped within a `try...finally` block).

```
wrap header:

        LOAD_GLOBALS    N (__armor_enter__)     N = length of co_consts
        CALL_FUNCTION   0
        POP_TOP
        SETUP_FINALLY   X (jump to wrap footer) X = size of original byte code

changed original byte code:

        Increase oparg of each absolute jump instruction by the size of wrap header

        Obfuscate original byte code

        ...

wrap footer:

        LOAD_GLOBALS    N + 1 (__armor_exit__)
        CALL_FUNCTION   0
        POP_TOP
        END_FINALLY
```

Afterwards, the followings are also acted upon each code object.

<ul>
<li>Append function names <b>__armor_enter, __armor_exit__</b> to <b>co_consts</b></li>
<li>Increase <b>co_stacksize</b> by 2</li>
<li>Set <b>CO_OBFUSCAED (0x80000000)</b> flag in <b>co_flags</b></li>
</ul>

After the code object has been modified, next it's serialized with marshal into byte code. This serialized code object (byte code) is then obfuscated to protect constants and literal strings.

```python
# co is the modified code object

char *string_code = marshal.dumps( co );
char *obfuscated_code = obfuscate_algorithm( string_code  );
```

Last but not least, the obfuscated script is generated.

```python
sprintf( buf, "__pyarmor__(__name__, __file__, b'%s')", obfuscated_code );
save_file( "dist/myscript.py", buf );
```

Now we have got the original script obfuscated. How to run them?

It turns out that <b>__pyarmor__, __armor_enter__,</b> and <b>__armor_exit__</b> has their own functionality in deobfuscating the script. Here's how it goes.

`__pyarmor__` is used to import original byte code (serialized code object) from the obfuscated code. The original byte code is then deserialized with marshal to obtain the code object.

```
static PyObject *
__pyarmor__(char *name, char *pathname, unsigned char *obfuscated_code)
{
    char *string_code = restore_obfuscated_code( obfuscated_code );
    PyCodeObject *co = marshal.loads( string_code );
    return PyImport_ExecCodeModuleEx( name, co, pathname );
}
```

`__armor_enter__` is then called when the code object is executed. It will restore the byte code of this code object.

This is performed since the obfuscation was done twice, one for the byte code of the original code object (before `try...finally` was added) and second for the byte code of the modified code object (after `try...finally` was added).

```
static PyObject *
__armor_enter__(PyObject *self, PyObject *args)
{
    // Got code object
    PyFrameObject *frame = PyEval_GetFrame();
    PyCodeObject *f_code = frame->f_code;

    // Increase refcalls of this code object
    // Borrow co_names->ob_refcnt as call counter
    // Generally it will not increased  by Python Interpreter
    PyObject *refcalls = f_code->co_names;
    refcalls->ob_refcnt ++;

    // Restore byte code if it's obfuscated
    if (IS_OBFUSCATED(f_code->co_flags)) {
        restore_byte_code(f_code->co_code);
        clear_obfuscated_flag(f_code);
    }

    Py_RETURN_NONE;
}
```

`__armor_exit__` is called after the code object execution completes. It will obfuscate the byte code again. I think in this case, <b>the byte code refers to the one from the original code object</b>. The reason would be since the obfuscation flag in `co_flags` denotes whether the byte code of the code object is obfuscated. I think so because the flag was set after the byte code is obfuscated and before the modified code object is serialized and obfuscated.

Another reason is that when the `__armor_enter__` is executed, it checks whether the byte code is obfuscated (via the flag) before restoring the byte code. If the obfuscation flag is used for the byte code of the modified code object, it's not logical to check the flag in this step again since the byte code of the modified code object has been restored (by `__pyarmor__`).

```
static PyObject *
__armor_exit__(PyObject *self, PyObject *args)
{
    // Got code object
    PyFrameObject *frame = PyEval_GetFrame();
    PyCodeObject *f_code = frame->f_code;

    // Decrease refcalls of this code object
    PyObject *refcalls = f_code->co_names;
    refcalls->ob_refcnt --;

    // Obfuscate byte code only if this code object isn't used by any function
    // In multi-threads or recursive call, one code object may be referenced
    // by many functions at the same time
    if (refcalls->ob_refcnt == 1) {
        obfuscate_byte_code(f_code->co_code);
        set_obfuscated_flag(f_code);
    }

    // Clear f_locals in this frame
    clear_frame_locals(frame);

    Py_RETURN_NONE;
}
```

Thank you for reading.

I really appreciate any feedback.
