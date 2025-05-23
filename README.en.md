# ex4nicegui

<div align="center">

English| [简体中文](./README.md)

</div>

An extension library for [nicegui](https://github.com/zauberzeug/nicegui). It has built-in responsive components and fully implements data-responsive interface programming.

![todo-app](https://github.com/CrystalWindSnake/ex4nicegui-examples/blob/main/asset/todo-app.02.gif)

[more examples](https://github.com/CrystalWindSnake/ex4nicegui-examples)

## 📦 Install

```
pip install ex4nicegui -U
```

---

## Guide

We start with a simple counter application where users can increase or decrease the count by clicking buttons.

![counter](https://github.com/CrystalWindSnake/ex4nicegui-examples/blob/main/asset/counter.gif)

Here is the complete code:

```python
from nicegui import ui
from ex4nicegui import rxui

# Data state code
class Counter(rxui.ViewModel):
    count: int = 0

    def increment(self):
        self.count += 1

    def decrement(self):
        self.count -= 1

# UI code
counter = Counter()

with ui.row(align_items="center"):
    ui.button(icon="remove", on_click=counter.decrement)
    rxui.label(counter.count).bind_color(counter.text_color)
    ui.button(icon="add", on_click=counter.increment)

ui.run()
```

---

Now let's dive into more details. `ex4nicegui` follows a data-driven approach to define the user interface. State data defines all the mutable data in the application.

Here is the state data definition for `Counter`:

```python
class Counter(rxui.ViewModel):
    count: int = 0
```

- Custom classes need to inherit from `rxui.ViewModel`.
- A variable `count` is defined here, representing the current value of the counter, with an initial value of 0.

Next, define a series of methods to manipulate the data within the class:
```python
def increment(self):
    self.count += 1

def decrement(self):
    self.count -= 1
```

- These are instance methods that can modify the value of the `count` variable.


Then, in the UI code, instantiate an object of `Counter`.
```python
counter = Counter()
```


We bind the `count` variable to the `rxui.label` component and bind the data manipulation methods to button click events.

```python
ui.button(icon="remove", on_click=counter.decrement)
rxui.label(counter.count)
ui.button(icon="add", on_click=counter.increment)
```

- We need to use the `label` component from the `rxui` namespace, not the `label` component from the `nicegui` namespace.
- The `rxui.label` component binds to the `counter.count` variable, and it automatically updates when `counter.count` changes.
- The `ui.button` component binds to the `counter.decrement` and `counter.increment` methods, which are called when the buttons are clicked.


> In complex projects, the `Counter` definition can be placed in a separate module and imported into the UI code.

Note that when a class variable name is prefixed with an underscore, the data state will not be automatically updated.

```python
class Counter(rxui.ViewModel):
    count: int = 0  # Reactive data, automatically synchronized with the UI
    _count: int = 0  # The underscore here indicates a private variable, which will not be automatically synchronized with the UI
```

---

### Computation

Continuing from the previous example, let's add another feature. When the counter value is less than 0, the text color should be red; when greater than 0, it should be green; otherwise, it should be black.

```python
# Data state code
class Counter(rxui.ViewModel):
    count: int = 0

    def text_color(self):
        if self.count > 0:
            return "green"
        elif self.count < 0:
            return "red"
        else:
            return "black"

    def increment(self):
        self.count += 1

    def decrement(self):
        self.count -= 1

# UI code
counter = Counter()

with ui.row(align_items="center"):
    ui.button(icon="remove", on_click=counter.decrement)
    rxui.label(counter.count).bind_color(counter.text_color)
    ui.button(icon="add", on_click=counter.increment)
```

The color value is computed based on the current value of the counter. This is a derived computation, which can be achieved by defining an ordinary instance method.

```python
def text_color(self):
    if self.count > 0:
        return "green"
    elif self.count < 0:
        return "red"
    else:
        return "black"
```


Then, bind the `text_color` method to the `rxui.label` component using the `bind_color` method to ensure the color value updates automatically.

```python
rxui.label(counter.count).bind_color(counter.text_color)
```

### Caching
Now, we will display the current counter value and its color text below the counter.

```python
...
#  Data state code
class Counter(rxui.ViewModel):
    ...

#  UI code
counter = Counter()

with ui.row(align_items="center"):
    ui.button(icon="remove", on_click=counter.decrement)
    rxui.label(counter.count).bind_color(counter.text_color)
    ui.button(icon="add", on_click=counter.increment)

rxui.label(lambda: f"Current counter value is {counter.count}, color value is {counter.text_color()}")
```

- When the derived computation is very simple, you can directly use a lambda expression.

In the code above, the `counter.text_color` method is used in two places. When `counter.count` changes, `counter.text_color` is computed twice, and the second computation is redundant.

To avoid the redundant computation, we can cache the result of `counter.text_color`.

```python
# Data state code
class Counter(rxui.ViewModel):
    count: int = 0

    @rxui.cached_var
    def text_color(self):
        if self.count > 0:
            return "green"
        elif self.count < 0:
            return "red"
        else:
            return "black"

```

- The `rxui.cached_var` decorator can cache the result of a function to avoid redundant computations.

### list data

The following example demonstrates how to use lists.

```python

class AppState(rxui.ViewModel):
    nums = []
    # nums = [1,2,3] ❌ If initialization is needed, it must be set in the `__init__` method.

    def __init__(self):
        super().__init__()
        self.nums = [1, 2, 3]

    def append(self):
        new_num = max(self.nums) + 1
        self.nums.append(new_num)

    def pop(self):
        self.nums.pop()

    def reverse(self):
        self.nums.reverse()

    def display_nums(self):
        return ", ".join(map(str, self.nums))


# UI code
state = AppState()

with ui.row(align_items="center"):
    ui.button("append", on_click=state.append)
    ui.button("pop", on_click=state.pop)
    ui.button("reverse", on_click=state.reverse)

rxui.label(state.display_nums)

```

If you need to initialize a list when defining it, it is recommended to set it in the `__init__` method.

```python
class AppState(rxui.ViewModel):
    nums = []
    # nums = [1,2,3] ❌ If initialization is needed, it must be set in the `__init__` method.

    def __init__(self):
        super().__init__()
        self.nums = [1, 2, 3]

    ...
```

Another way is to use `rxui.list_var`.



```python
class AppState(rxui.ViewModel):
    # nums = []
    # nums = [1,2,3] ❌
    nums = rxui.list_var(lambda: [1, 2, 3])

    ...
```

- The parameter for `rxui.list_var` is a function that returns a list.


### List Looping

After defining a list, we can use the `effect_refreshable.on` decorator to display the list data in the UI.

In the following example, the UI will dynamically display the icon selected from a dropdown.
```python
from ex4nicegui import rxui, effect_refreshable


class AppState(rxui.ViewModel):
    icons = []
    _option_icons = ["font_download", "warning", "format_size", "print"]


state = AppState()

# UI code
with ui.row(align_items="center"):

    @effect_refreshable.on(state.icons)
    def _():
        for icon in state.icons:
            ui.icon(icon, size="2rem")


rxui.select(state._option_icons, value=state.icons, multiple=True)
```

Here, `@effect_refreshable.on(state.icons)` explicitly specifies the dependency. When `state.icons` changes, the `_` function will be re-executed.

```python
@effect_refreshable.on(state.icons)
def _():
    # This code will re-execute when `state.icons` changes.
    ...
```

> Note that each execution will clear the content inside. This is the data-driven version of `ui.refreshable`.

In principle, you don't need to specify the data to monitor using `.on`. Any "reactive data" used within the function will be automatically monitored.
```python
@effect_refreshable # Not using .on(state.icons)
def _():
    # This reads `state.icons`, so it will be automatically monitored
    for icon in state.icons:
        ui.icon(icon, size="2rem")

```

> It is recommended to always specify dependencies using `.on` to avoid unexpected refreshes.

---

### Data Persistence

`ViewModel` uses proxy objects to create reactive data. When you need to save the data, you can use `rxui.ViewModel.to_value` to convert it into plain data.

In the following example, clicking the button will display the state data dictionary of `my_app`.

```python
from nicegui import ui
from ex4nicegui import rxui


class MyApp(rxui.ViewModel):
    a = 0
    sign = "+"
    b = 0

    def show_data(self):
        # >> {"a": 0, "sign": '+', "b": 0}
        return rxui.ViewModel.to_value(self)

    def show_a(self):
        # >> 0
        return rxui.ViewModel.to_value(self.a)

my_app = MyApp()

rxui.number(value=my_app.a, min=0, max=10)
rxui.radio(["+", "-", "*", "/"], value=my_app.sign)
rxui.number(value=my_app.b, min=0, max=10)

ui.button("show data", on_click=lambda: ui.notify(my_app.show_data()))
```

By combining `rxui.ViewModel.on_refs_changed`, you can automatically save the data to a local file whenever the data changes.


```python
from nicegui import ui
from ex4nicegui import rxui
from pathlib import Path
import json


class MyApp(rxui.ViewModel):
    a = 0
    sign = "+"
    b = 0

    _json_path = Path(__file__).parent / "data.json"

    def __init__(self):
        super().__init__()

        @rxui.ViewModel.on_refs_changed(self)
        def _():
            # Automatically save to local file when any of a, sign, b changes
            self._json_path.write_text(json.dumps(self.show_data()))

    def show_data(self):
        return rxui.ViewModel.to_value(self)
...
```

---

- [ex4nicegui](#ex4nicegui)
  - [📦 Install](#-install)
  - [Guide](#guide)
    - [Computation](#computation)
    - [Caching](#caching)
    - [list data](#list-data)
    - [List Looping](#list-looping)
    - [Data Persistence](#data-persistence)
  - [apis](#apis)
    - [ViewModel](#viewmodel)
      - [for list data](#for-list-data)
    - [reactive](#reactive)
      - [`to_ref`](#to_ref)
      - [`deep_ref`](#deep_ref)
      - [`effect`](#effect)
      - [`ref_computed`](#ref_computed)
      - [`async_computed`](#async_computed)
      - [`on`](#on)
      - [`new_scope`](#new_scope)
    - [functionality](#functionality)
      - [vmodel](#vmodel)
      - [vfor](#vfor)
      - [bind\_classes](#bind_classes)
      - [bind\_style](#bind_style)
      - [bind\_prop](#bind_prop)
      - [rxui.echarts](#rxuiecharts)
        - [echarts mouse events](#echarts-mouse-events)
        - [rxui.echarts.from\_javascript](#rxuiechartsfrom_javascript)
        - [rxui.echarts.register\_map](#rxuiechartsregister_map)
        - [rxui.echarts.register\_theme](#rxuiechartsregister_theme)
    - [tab\_panels](#tab_panels)
    - [lazy\_tab\_panels](#lazy_tab_panels)
    - [scoped\_style](#scoped_style)
    - [BI Module](#bi-module)
      - [`bi.data_source`](#bidata_source)
      - [ui\_select](#ui_select)
      - [ui\_table](#ui_table)
      - [ui\_aggrid](#ui_aggrid)
    - [Toolbox](#toolbox)
      - [use\_dark](#use_dark)
      - [use\_breakpoints](#use_breakpoints)
      - [use\_qr\_code](#use_qr_code)
---

## apis

### ViewModel
In version v0.7.0, the `ViewModel` class has been incorporated to facilitate the management of reactive data sets. Below is an illustrative example utilizing a basic calculator interface:

1. Upon modification of the numerical input fields or selection of arithmetic symbols by the user, the resultant value is instantaneously reflected on the display panel.
2. Should the computed outcome fall below zero, its representation assumes a red hue; conversely, non-negative results are rendered in black.

```python
from ex4nicegui import rxui

class Calculator(rxui.ViewModel):
    num1 = 0
    sign = "+"
    num2 = 0

    @rxui.cached_var
    def result(self):
        # When any of the values of num1, sign, or num2 changes, the result is recalculated accordingly.
        return eval(f"{self.num1}{self.sign}{self.num2}")

# Each object possesses its own independent data.
calc = Calculator()

with ui.row(align_items="center"):
    rxui.number(value=calc.num1, label="Number 1")
    rxui.select(value=calc.sign, options=["+", "-", "*", "/"], label="Sign")
    rxui.number(value=calc.num2, label="Number 2")
    ui.label("=")
    rxui.label(calc.result).bind_color(
        lambda: "red" if calc.result() < 0 else "black"
    )

```

---

#### for list data

In the following example, each person is displayed using a card. The average age of all individuals is shown at the top. When an individual's age exceeds the average age, the card's border turns red.

Age modifications are made through the `number` component, triggering automatic updates across the display.

```python
from typing import List
from ex4nicegui import rxui, Ref
from itertools import count
from nicegui import ui

id_generator = count()


class Person(rxui.ViewModel):
    name = ""
    age = 0

    def __init__(self, name: str, age: int):
        super().__init__()
        self.name = name
        self.age = age
        self.id = next(id_generator)



class Home(rxui.ViewModel):
    persons: List[Person] = []
    deleted_person_index = 0

    @rxui.cached_var
    def avg_age(self) -> float:
        if len(self.persons) == 0:
            return 0

        return round(sum(p.age for p in self.persons) / len(self.persons), 2)

    def avg_name_length(self):
        if len(self.persons) == 0:
            return 0

        return round(sum(len(p.name) for p in self.persons) / len(self.persons), 2)

    def delete_person(self):
        if self.deleted_person_index < len(self.persons):
            del self.persons[int(self.deleted_person_index)]

    def sample_data(self):
        self.persons = [
            Person("alice", 25),
            Person("bob", 30),
            Person("charlie", 31),
            Person("dave", 22),
            Person("eve", 26),
            Person("frank", 29),
        ]

home = Home()
home.sample_data()

rxui.label(lambda: f"avg age: {home.avg_age()}")
rxui.label(lambda: f"avg name length: {home.avg_name_length()}")

rxui.number(
    value=home.deleted_person_index, min=0, max=lambda: len(home.persons) - 1, step=1
)
ui.button("delete", on_click=home.delete_person)

with ui.row():

    @rxui.vfor(home.persons, key="id")
    def _(store: rxui.VforStore[Person]):
        person = store.get_item()
        with rxui.card().classes("outline").bind_classes(
            {
                "outline-red-500": lambda: person.age.value > home.avg_age(),
            }
        ):
            rxui.input(value=person.name, placeholder="name")
            rxui.number(value=person.age, min=1, max=100, step=1, placeholder="age")

ui.run()
```

If you find the `rxui.vfor` code too complex, you can use the `effect_refreshable` decorator instead.

```python
from ex4nicegui import rxui, Ref, effect_refreshable
...

# Explicitly specifying to monitor changes in `home.persons` can prevent unintended refreshes.
@effect_refreshable.on(home.persons)
def _():
    
    for person in home.persons.value:
        ...
        rxui.number(value=person.age, min=1, max=100, step=1, placeholder="Age")
...
```

Note that whenever the `home.persons` list changes (such as when elements are added or removed), the function decorated with `effect_refreshable` will be re-executed. This means that all elements will be recreated.


For more complex applications, please refer to the [examples](./examples)

---

### reactive

```python
from ex4nicegui import (
    to_ref,
    ref_computed,
    on,
    effect,
    effect_refreshable,
    batch,
    event_batch,
    deep_ref,
)
```
Commonly used `to_ref`,`deep_ref`,`effect`,`ref_computed`,`on`

#### `to_ref`
Defines responsive objects, read and written by `.value`.
```python
a = to_ref(1)
b = to_ref("text")

a.value =2
b.value = 'new text'

print(a.value)
```

When the value is a complex object, responsiveness of nested objects is not maintained by default.

```python
a = to_ref([1,2])

@effect
def _().
    print(len(a.value))

# doesn't trigger the effect
a.value.append(10)

# the whole substitution will be triggered
a.value = [1,2,10]
```

Depth responsiveness is obtained when `is_deep` is set to `True`.

```python
a = to_ref([1,2],is_deep=True)

@effect
def _():
    print('len:',len(a.value))

# print 3
a.value.append(10)

```

> `deep_ref` is equivalent to `to_ref` if `is_deep` is set to `True`.
---

#### `deep_ref`
Equivalent to `to_ref` when `is_deep` is set to `True`.

Especially useful when the data source is a list, dictionary or custom class. Objects obtained via `.value` are proxies

```python
data = [1,2,3]
data_ref = deep_ref(data)

assert data_ref.value is not data
```

You can get the raw object with `to_raw`.
```python
from ex4nicegui import to_raw, deep_ref

data = [1, 2, 3]
data_ref = deep_ref(data)

assert data_ref.value is not data
assert to_raw(data_ref.value) is data
```

---

#### `effect`
Accepts a function and automatically monitors changes to the responsive objects used in the function to automatically execute the function.

```python
a = to_ref(1)
b = to_ref("text")


@effect
def auto_run_when_ref_value():
    print(f"a:{a.value}")


def change_value():
    a.value = 2
    b.value = "new text"


ui.button("change", on_click=change_value)
```

The first time the effect is executed, the function `auto_run_when_ref_value` will be executed once. After that, clicking on the button changes the value of `a` (via `a.value`) and the function `auto_run_when_ref_value` is executed again.

> Never spread a lot of data processing logic across multiple `on`s or `effects`s, which should be mostly interface manipulation logic rather than responsive data processing logic

---

#### `ref_computed`
As with `effect`, `ref_computed` can also return results from functions. Typically used for secondary computation from `to_ref`.

```python
a = to_ref(1)
a_square = ref_computed(lambda: a.value * 2)


@effect
def effect1():
    print(f"a_square:{a_square.value}")


def change_value():
    a.value = 2


ui.button("change", on_click=change_value)
```

When the button is clicked, the value of `a.value` is modified, triggering a recalculation of `a_square`. As the value of `a_square` is read in `effect1`, it triggers `effect1` to execute the

> `ref_computed` is read-only `to_ref`

Starting from version `v0.7.0`, it is not recommended to use `ref_computed` for instance methods. You can use `rxui.ViewModel` and apply the `rxui.cached_var` decorator instead.

```python
class MyState(rxui.ViewModel):
    def __init__(self) -> None:
        self.r_text = to_ref("")

    @rxui.cached_var
    def post_text(self):
        return self.r_text.value + "post"

state = MyState()

rxui.input(value=state.r_text)
rxui.label(state.post_text)
```

---

#### `async_computed`
Use `async_computed` when asynchronous functions are required for computed.

```python

# Simulate asynchronous functions that take a long time to execute
async def long_time_query(input: str):
    await asyncio.sleep(2)
    num = random.randint(20, 100)
    return f"query result[{input=}]:{num=}"


search = to_ref("")
evaluating = to_ref(False)

@async_computed(search, evaluating=evaluating, init="")
async def search_result():
    return await long_time_query(search.value)

rxui.lazy_input(value=search)

rxui.label(
    lambda: "Query in progress" if evaluating.value else "Enter content in input box above and return to search"
)
rxui.label(search_result)

```

- The first argument to `async_computed` must explicitly specify the responsive data to be monitored. Multiple responses can be specified using a list.
- Parameter `evaluating` is responsive data of type bool, which is `True` while the asynchronous function is executing, and `False` after the computation is finished.
- Parameter `init` specifies the initial result

---

#### `on`
Similar to `effect`, but `on` needs to explicitly specify the responsive object to monitor.

```python

a1 = to_ref(1)
a2 = to_ref(10)
b = to_ref("text")


@on(a1)
def watch_a1_only():
    print(f"watch_a1_only ... a1:{a1.value},a2:{a2.value}")


@on([a1, b], onchanges=True)
def watch_a1_and_b():
    print(f"watch_a1_and_b ... a1:{a1.value},a2:{a2.value},b:{b.value}")


def change_a1():
    a1.value += 1
    ui.notify("change_a1")


ui.button("change a1", on_click=change_a1)


def change_a2():
    a2.value += 1
    ui.notify("change_a2")


ui.button("change a2", on_click=change_a2)


def change_b():
    b.value += "x"
    ui.notify("change_b")


ui.button("change b", on_click=change_b)

```

- If the parameter `onchanges` is True (the default value is False), the specified function will not be executed at binding time.

> Never spread a lot of data processing logic across multiple `on`s or `effects`s, which should be mostly interface manipulation logic rather than responsive data processing logic

---

#### `new_scope`

By default, all watch functions are automatically destroyed when the client connection is disconnected. For finer-grained control, the `new_scope` function can be utilized.

```python
from nicegui import ui
from ex4nicegui import rxui, to_ref, effect, new_scope

a = to_ref(0.0)

scope1 = new_scope()

@scope1.run
def _():
    @effect
    def _():
        print(f"scope 1:{a.value}")


rxui.number(value=a)
rxui.button("dispose scope 1", on_click=scope1.dispose)
```

---

### functionality

#### vmodel
Create two-way bindings on form input elements or components.

Bidirectional bindings are supported by default for `ref` simple value types
```python
from ex4nicegui import rxui, to_ref, deep_ref

data = to_ref("init")

rxui.label(lambda: f"{data.value=}")
# two-way binding 
rxui.input(value=data)
```
- Simple value types are generally immutable values such as `str`, `int`, etc.

When using complex data structures, `deep_ref` is used to keep nested values responsive.
```python
data = deep_ref({"a": 1, "b": [1, 2, 3, 4]})

rxui.label(lambda: f"{data.value=!s}")

# No binding effect
rxui.input(value=data.value["a"])

# readonly binding
rxui.input(value=lambda: data.value["a"])

# two-way binding
rxui.input(value=rxui.vmodel(data,'a'))

# also two-way binding, but not recommended
rxui.input(value=rxui.vmodel(data.value["a"]))
```

- The first input box will be completely unresponsive because the code is equivalent to `rxui.input(value=1)`
- The second input box will get read responsiveness due to the use of a function.
- The third input box, wrapped in `rxui.vmodel`, can be bi-directionally bound.

> If you use `rxui.ViewModel`, you may find that you don't need to use `vmodel` at all.

see [todo list examples](./examples/todomvc/)

---

#### vfor
Render list components based on list responsive data. Each component is updated on demand. Data items support dictionaries or objects of any type

Starting from version `v0.7.0`, it is recommended to use in conjunction with `rxui.ViewModel`. Unlike using the `effect_refreshable` decorator, `vfor` does not recreate all elements but updates existing ones.

Below is an example of card sorting, where cards are always sorted by age. When you modify the age data in a card, the cards adjust their order in real time. However, the cursor focus does not leave the input field.

```python
from typing import List
from nicegui import ui
from ex4nicegui import rxui, deep_ref as ref, Ref


class Person(rxui.ViewModel):
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = ref(age)


class MyApp(rxui.ViewModel):
    persons: Ref[List[Person]] = rxui.var(lambda: [])
    order = rxui.var("asc")

    def sort_by_age(self):
        return sorted(
            self.persons.value,
            key=lambda p: p.age.value,
            reverse=self.order.value == "desc",
        )

    @staticmethod
    def create():
        persons = [
            Person(name="Alice", age=25),
            Person(name="Bob", age=30),
            Person(name="Charlie", age=20),
            Person(name="Dave", age=35),
            Person(name="Eve", age=28),
        ]
        app = MyApp()
        app.persons.value = persons
        return app


# UI
app = MyApp.create()

with rxui.tabs(app.order):
    rxui.tab("asc", "Ascending")
    rxui.tab("desc", "Descending")


@rxui.vfor(app.sort_by_age, key="name")
def each_person(s: rxui.VforStore[Person]):
    person = s.get_item()

    with ui.card(), ui.row(align_items="center"):
        rxui.label(person.name)
        rxui.number(value=person.age, step=1, min=0, max=100)
```

- The `rxui.vfor` decorator is applied to a custom function.
    - The first argument is the reactive list. Note that there is no need to call `app.sort_by_age`.
    - The second argument `key`: To track each node's identifier and thus reuse and reorder existing elements, you can provide a unique key for each block corresponding to an element. By default, the list element index is used. In the example, it is assumed that each person's name is unique.
- The custom function takes one parameter. You can obtain the current row's object via `store.get_item`. Since `Person` itself inherits from `rxui.ViewModel`, its properties can be directly bound to components.


---


#### bind_classes

All component classes provide `bind_classes` for binding `class`, supporting three different data structures.

Bind dictionaries

```python
bg_color = to_ref(False)
has_error = to_ref(False)

rxui.label("test").bind_classes({"bg-blue": bg_color, "text-red": has_error})

rxui.switch("bg_color", value=bg_color)
rxui.switch("has_error", value=has_error)
```

Dictionary key  is the class name, and a dictionary value of  a responsive variable with value `bool`. When the responsive value is `True`, the class name is applied to the component `class`.

---

Bind a responsive variable whose return value is a dictionary.

```python
bg_color = to_ref(False)
has_error = to_ref(False)

class_obj = ref_computed(
    lambda: {"bg-blue": bg_color.value, "text-red": has_error.value}
)

rxui.switch("bg_color", value=bg_color)
rxui.switch("has_error", value=has_error)
rxui.label("bind to ref_computed").bind_classes(class_obj)

# or direct function passing
rxui.label("bind to ref_computed").bind_classes(
    lambda: {"bg-blue": bg_color.value, "text-red": has_error.value}
)
```

---

Bind to list

```python
bg_color = to_ref("red")
bg_color_class = ref_computed(lambda: f"bg-{bg_color.value}")

text_color = to_ref("green")
text_color_class = ref_computed(lambda: f"text-{text_color.value}")

rxui.select(["red", "green", "yellow"], label="bg color", value=bg_color)
rxui.select(["red", "green", "yellow"], label="text color", value=text_color)

rxui.label("binding to arrays").bind_classes([bg_color_class, text_color_class])
rxui.label("binding to single string").bind_classes(bg_color_class)
```

- Each element in the list is a responsive variable that returns the class name

---

#### bind_style

```python
from nicegui import ui
from ex4nicegui.reactive import rxui
from ex4nicegui.utils.signals import to_ref


bg_color = to_ref("blue")
text_color = to_ref("red")

rxui.label("test").bind_style(
    {
        "background-color": bg_color,
        "color": text_color,
    }
)

rxui.select(["blue", "green", "yellow"], label="bg color", value=bg_color)
rxui.select(["red", "green", "yellow"], label="text color", value=text_color)
```

`bind_style` passed into dictionary, `key` is style name, `value` is style value, responsive string

---

#### bind_prop
Binds a property.

```python
label = to_ref("hello")

rxui.button("").bind_prop("label", label)
# Functions are also permissible
rxui.button("").bind_prop(
    "label", lambda: f"{label.value} world"
)

rxui.input(value=label)
```

---

#### rxui.echarts
Charting with echarts


```python
from nicegui import ui
from ex4nicegui import ref_computed, effect, to_ref
from ex4nicegui.reactive import rxui

r_input = to_ref("")


# ref_computed Creates a read-only reactive variable
# Functions can use any other reactive variables automatically when they are associated
@ref_computed
def cp_echarts_opts():
    return {
        "title": {"text": r_input.value}, #In the dictionary, use any reactive variable. Get the value by .value
        "xAxis": {
            "type": "category",
            "data": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
        },
        "yAxis": {"type": "value"},
        "series": [
            {
                "data": [120, 200, 150, 80, 70, 110, 130],
                "type": "bar",
                "showBackground": True,
                "backgroundStyle": {"color": "rgba(180, 180, 180, 0.2)"},
            }
        ],
    }


input = rxui.input("Enter the content, and the chart title will be synchronized", value=r_input)

# Get the native nicegui component object through the element attribute of the responsive component object
input.element.classes("w-full")

rxui.echarts(cp_echarts_opts).classes('h-[20rem]')

ui.run()
```
![](./asset/asyc_echarts_title.gif)


##### echarts mouse events

the `on` function parameters `event_name` and `query` to view the[echarts English Documentation](https://echarts.apache.org/handbook/en/concepts/event/)


The following example binds the mouse click event
```python
from nicegui import ui
from ex4nicegui.reactive import rxui

opts = {
    "xAxis": {"type": "value", "boundaryGap": [0, 0.01]},
    "yAxis": {
        "type": "category",
        "data": ["Brazil", "Indonesia", "USA", "India", "China", "World"],
    },
    "series": [
        {
            "name": "first",
            "type": "bar",
            "data": [18203, 23489, 29034, 104970, 131744, 630230],
        },
        {
            "name": "second",
            "type": "bar",
            "data": [19325, 23438, 31000, 121594, 134141, 681807],
        },
    ],
}

bar = rxui.echarts(opts)

def on_click(e: rxui.echarts.EChartsMouseEventArguments):
    ui.notify(f"on_click:{e.seriesName}:{e.name}:{e.value}")

bar.on("click", on_click)
```


The following example only triggers the mouseover event for a specified series.
```python
from nicegui import ui
from ex4nicegui.reactive import rxui

opts = {
    "xAxis": {"type": "value", "boundaryGap": [0, 0.01]},
    "yAxis": {
        "type": "category",
        "data": ["Brazil", "Indonesia", "USA", "India", "China", "World"],
    },
    "series": [
        {
            "name": "first",
            "type": "bar",
            "data": [18203, 23489, 29034, 104970, 131744, 630230],
        },
        {
            "name": "second",
            "type": "bar",
            "data": [19325, 23438, 31000, 121594, 134141, 681807],
        },
    ],
}

bar = rxui.echarts(opts)

def on_first_series_mouseover(e: rxui.echarts.EChartsMouseEventArguments):
    ui.notify(f"on_first_series_mouseover:{e.seriesName}:{e.name}:{e.value}")


bar.on("mouseover", on_first_series_mouseover, query={"seriesName": "first"})

ui.run()
```
---




##### rxui.echarts.from_javascript
Create echart from javascript code

```python
from pathlib import Path

rxui.echarts.from_javascript(Path("code.js"))
# or
rxui.echarts.from_javascript(
    """
(myChart) => {

    option = {
        xAxis: {
            type: 'category',
            data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
        },
        yAxis: {
            type: 'value'
        },
        series: [
            {
                data: [120, 200, 150, 80, 70, 110, 130],
                type: 'bar'
            }
        ]
    };

    myChart.setOption(option);
}
"""
)
```

- The first parameter of the function is the echart instance object. You need to configure the chart in the function with `setOption`.

The function also has a second parameter, which is the global echarts object, allowing you to register maps via echarts.registerMap.


```python
rxui.echarts.from_javascript(
"""
(chart,echarts) =>{

    fetch('https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json')
    .then(response => response.json())
    .then(data => {
            echarts.registerMap('test_map', data);

            chart.setOption({
                geo: {
                    map: 'test_map',
                    roam: true,
                },
                tooltip: {},
                legend: {},
                series: [],
            });
    });
}
"""
)
```


---

##### rxui.echarts.register_map
Register a map.

```python
rxui.echarts.register_map(
    "china", "https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json"
)

rxui.echarts(
    {
        "geo": {
            "map": "china",
            "roam": True,
        },
        "tooltip": {},
        "legend": {},
        "series": [], 
    }
)
```

- The parameter `map_name` is a customized map name. Note that `map` must correspond to a registered name in the chart configuration.
- The parameter `src` is a valid network link to the map data.


If it's SVG data, you need to set the parameter `type="svg"`
```python
rxui.echarts.register_map("svg-rect", "/test/svg", type="svg")
```



You can also provide the path to the local json file for the map data.
```python
from pathlib import Path

rxui.echarts.register_map(
    "china", Path("map-data.json")
)
```

---

##### rxui.echarts.register_theme
Register a theme.

```python
@ui.page("/")  
def page():  
    # Within the scope of this page, the default theme for all rxui.echarts will be "walden" (the last registered theme)  
    rxui.echarts.register_theme(  
        "my_theme", Path(__file__).parent / "echarts_theme.json"  
    ).register_theme("walden", Path(__file__).parent / "echarts_walden.json")  

    opts = {  
        "xAxis": {"data": ["Shirts", "Cardigans"]},  
        "yAxis": {},  
        "series": [{"type": "bar", "data": [5, 20]}],  
    }  

    # Theme is "walden"  
    rxui.echarts(opts)  

    # Theme is "my_theme"  
    rxui.echarts(  
        opts,  
        theme="my_theme",  
    )  

```

---

### tab_panels

Compared to `nicegui.ui.tab_panels`, `rxui.tab_panels` lacks a `tabs` parameter. Under the reactive data mechanism, the linkage between `tabs` and `tab_panels` can be achieved solely through the `value` parameter.

```python
from nicegui import ui
from ex4nicegui import rxui, to_ref

names = ["Tab 1", "Tab 2", "Tab 3"]
current_tab = to_ref(names[0])

with rxui.tabs(current_tab):
    for name in names:
        rxui.tab(name)

with rxui.tab_panels(current_tab):
    for name in names:
        with rxui.tab_panel(name):
            ui.label(f"Content of {name}")
```

This is because, in a reactive context, component interactions are facilitated by an intermediary data layer (`to_ref`). Consequently, `tab_panels` can be synchronized with other components (ensuring the use of the same `ref` object).

```python
names = ["Tab 1", "Tab 2", "Tab 3"]
current_tab = to_ref(names[0])

with rxui.tab_panels(current_tab):
    for name in names:
        with rxui.tab_panel(name):
            ui.label(f"Content of {name}")

# The tabs definition does not have to precede the panels
with rxui.tabs(current_tab):
    for name in names:
        rxui.tab(name)

rxui.select(names, value=current_tab)
rxui.radio(names, value=current_tab).props("inline")

rxui.label(lambda: f"Current tab is: {current_tab.value}")
```

---

### lazy_tab_panels

In lazy loading mode, only the currently active tab is rendered.
```python
from ex4nicegui import to_ref, rxui, on, deep_ref

current_tab = to_ref("t1")

with rxui.tabs(current_tab):
    ui.tab("t1")
    ui.tab("t2")

with rxui.lazy_tab_panels(current_tab) as panels:

    @panels.add_tab_panel("t1")
    def _():
        # you can use `panels.get_panel` to get the panel component.
        panels.get_panel("t1").classes("bg-green")
        ui.notify("Hello from t1")
        ui.label("This is t1")

    @panels.add_tab_panel("t2")
    def _():
        panels.get_panel("t2").style("background-color : red")
        ui.notify("Hello from t2")
        ui.label("This is t2")

```

Upon page load, "Hello from t1" is displayed immediately. When switching to the "t2" tab, "Hello from t2" is then shown.

---
### scoped_style
The `scoped_style` method allows you to create styles that are scoped within the component.

```python
# All child elements will have a red outline, excluding the component itself
with rxui.row().scoped_style("*", "outline: 1px solid red;") as row:
    ui.label("Hello")
    ui.label("World")


# All child elements will have a red outline, including the component itself
with rxui.row().scoped_style(":self *", "outline: 1px solid red;") as row:
    ui.label("Hello")
    ui.label("World")

# When hovering over the row component, all child elements will have a red outline, excluding the component itself
with rxui.row().scoped_style(":hover *", "outline: 1px solid red;") as row:
    ui.label("Hello")
    ui.label("World")

# When hovering over the row component, all child elements will have a red outline, including the component itself
with rxui.row().scoped_style(":self:hover *", "outline: 1px solid red;") as row:
    ui.label("Hello")
    ui.label("World")
```

---

### BI Module

Create an interactive data visualization report using the minimal API.

![](./asset/bi_examples1.gif)

```python
from nicegui import ui
import pandas as pd
import numpy as np
from ex4nicegui import bi
from ex4nicegui.reactive import rxui
from ex4nicegui import effect, effect_refreshable
from pyecharts.charts import Bar


# data ready
def gen_data():
    np.random.seed(265)
    field1 = ["a1", "a2", "a3", "a4"]
    field2 = [f"name{i}" for i in range(1, 11)]
    df = (
        pd.MultiIndex.from_product([field1, field2], names=["cat", "name"])
        .to_frame()
        .reset_index(drop=True)
    )
    df[["idc1", "idc2"]] = np.random.randint(50, 1000, size=(len(df), 2))
    return df


df = gen_data()

# Create a data source.
ds = bi.data_source(df)

# ui
ui.query(".nicegui-content").classes("items-stretch no-wrap")

with ui.row().classes("justify-evenly"):
    # Create components based on the data source `ds`.
    ds.ui_select("cat").classes("min-w-[10rem]")
    ds.ui_select("name").classes("min-w-[10rem]")


with ui.grid(columns=2):
    # Configure the chart using a dictionary.
    @ds.ui_echarts
    def bar1(data: pd.DataFrame):
        data = data.groupby("name").agg({"idc1": "sum", "idc2": "sum"}).reset_index()

        return {
            "xAxis": {"type": "value"},
            "yAxis": {
                "type": "category",
                "data": data["name"].tolist(),
                "inverse": True,
            },
            "legend": {"textStyle": {"color": "gray"}},
            "series": [
                {"type": "bar", "name": "idc1", "data": data["idc1"].tolist()},
                {"type": "bar", "name": "idc2", "data": data["idc2"].tolist()},
            ],
        }

    bar1.classes("h-[20rem]")

    # Configure the chart using pyecharts.
    @ds.ui_echarts
    def bar2(data: pd.DataFrame):
        data = data.groupby("name").agg({"idc1": "sum", "idc2": "sum"}).reset_index()

        return (
            Bar()
            .add_xaxis(data["name"].tolist())
            .add_yaxis("idc1", data["idc1"].tolist())
            .add_yaxis("idc2", data["idc2"].tolist())
        )

    bar2.classes("h-[20rem]")

    # Bind the click event to achieve navigation.
    @bar2.on_chart_click
    def _(e: rxui.echarts.EChartsMouseEventArguments):
        ui.open(f"/details/{e.name}", new_tab=True)


# with response mechanisms, you can freely combine native nicegui components.
label_a1_total = ui.label("")


# this function will be triggered when ds changed.
@effect
def _():
    # prop `filtered_data` is the filtered DataFrame.
    df = ds.filtered_data
    total = df[df["cat"] == "a1"]["idc1"].sum()
    label_a1_total.text = f"idc1 total(cat==a1):{total}"


# you can also use `effect_refreshable`, but you need to note that the components in the function are rebuilt each time.
@effect_refreshable
def _():
    df = ds.filtered_data
    total = df[df["cat"] == "a2"]["idc1"].sum()
    ui.label(f"idc1 total(cat==a2):{total}")


# the page to be navigated when clicking on the chart series.
@ui.page("/details/{name}")
def details_page(name: str):
    ui.label("This table data will not change")
    ui.aggrid.from_pandas(ds.data.query(f'name=="{name}"'))

    ui.label("This table will change when the homepage data changes. ")

    @bi.data_source
    def new_ds():
        return ds.filtered_data[["name", "idc1", "idc2"]]

    new_ds.ui_aggrid()


ui.run()
```


#### `bi.data_source`
The data source is the core concept of the BI module, and all data linkage is based on this. In the current version (0.4.3), there are two ways to create a data source.

Receive `pandas`'s `DataFrame`:
```python
from nicegui import ui
from ex4nicegui import bi
import pandas as pd

df = pd.DataFrame(
    {
        "name": list("aabcdf"),
        "cls": ["c1", "c2", "c1", "c1", "c3", None],
        "value": range(6),
    }
)

ds =  bi.data_source(df)
```

Sometimes, we want to create a new data source based on another data source, in which case we can use a decorator to create a linked data source:
```python
df = pd.DataFrame(
    {
        "name": list("aabcdf"),
        "cls": ["c1", "c2", "c1", "c1", "c3", None],
        "value": range(6),
    }
)

ds =  bi.data_source(df)

@bi.data_source
def new_ds():
    # df is pd.DataFrame 
    df = ds.filtered_data
    df=df.copy()
    df['value'] = df['value'] * 100
    return df

ds.ui_select('name')
new_ds.ui_aggrid()
```

Note that since `new_ds` uses `ds.filtered_data`, changes to `ds` will trigger the linkage change of `new_ds`, causing the table component created by `new_ds` to change.

---

Remove all filter states through the `ds.remove_filters` method:
```python
ds = bi.data_source(df)

def on_remove_filters():
    ds.remove_filters()

ui.button("remove all filters", on_click=on_remove_filters)

ds.ui_select("name")
ds.ui_aggrid()
```
---

Reset the data source through the `ds.reload` method:
```python

df = pd.DataFrame(
    {
        "name": list("aabcdf"),
        "cls": ["c1", "c2", "c1", "c1", "c3", None],
        "value": range(6),
    }
)

new_df = pd.DataFrame(
    {
        "name": list("xxyyds"),
        "cls": ["cla1", "cla2", "cla3", "cla3", "cla3", None],
        "value": range(100, 106),
    }
)

ds = bi.data_source(df)

def on_remove_filters():
    ds.reload(new_df)

ui.button("reload data", on_click=on_remove_filters)

ds.ui_select("name")
ds.ui_aggrid()
```

---
#### ui_select

Dropdown Select Box

```python
from nicegui import ui
from ex4nicegui import bi
import pandas as pd

df = pd.DataFrame(
    {
        "name": list("aabcdf"),
        "cls": ["c1", "c2", "c1", "c1", "c3", None],
        "value": range(6),
    }
)

ds = bi.data_source(df)

ds.ui_select("name")
```

The first parameter column specifies the column name of the data source.

---
Set the order of options using the parameter `sort_options`:
```python
ds.ui_select("name", sort_options={"value": "desc", "name": "asc"})
```

---
Set whether to exclude null values using the parameter `exclude_null_value`:
```python
df = pd.DataFrame(
    {
        "cls": ["c1", "c2", "c1", "c1", "c3", None],
    }
)

ds = bi.data_source(df)
ds.ui_select("cls", exclude_null_value=True)
```

---
You can set the parameters of the native nicegui select component through keyword arguments.

Set default values through the value attribute:
```python
ds.ui_select("cls",value=['c1','c2'])
ds.ui_select("cls",multiple=False,value='c1')
```
For multiple selections (the parameter `multiple` is defaulted to True), `value` needs to be specified as a list. For single selections, `value` should be set to non-list.

---


#### ui_table

Table

```python
from nicegui import ui
from ex4nicegui import bi
import pandas as pd

data = pd.DataFrame({"name": ["f", "a", "c", "b"], "age": [1, 2, 3, 1]})
ds = bi.data_source(data)

ds.ui_table(
    columns=[
        {"label": "new colA", "field": "colA", "sortable": True},
    ]
)

```

- The parameter `columns` are consistent with nicegui `ui.table`. The key value `field` corresponds to the column name of the data source, and if it does not exist, this configuration will not take effect
- The `rows` parameter will not take effect. Because the data source of the table is always controlled by the data source.

---

#### ui_aggrid


```python
from nicegui import ui
from ex4nicegui import bi
import pandas as pd

data = pd.DataFrame(
    {
        "colA": list("abcde"),
        "colB": [f"n{idx}" for idx in range(5)],
        "colC": list(range(5)),
    }
)
df = pd.DataFrame(data)

source = bi.data_source(df)

source.ui_aggrid(
    options={
        "columnDefs": [
            {"headerName": "xx", "field": "no exists"},
            {"headerName": "new colA", "field": "colA"},
            {
                "field": "colC",
                "cellClassRules": {
                    "bg-red-300": "x < 3",
                    "bg-green-300": "x >= 3",
                },
            },
        ],
        "rowData": [{"colX": [1, 2, 3, 4, 5]}],
    }
)
```

- The parameter `options` is consistent with nicegui `ui.aggrid`. The key value `field` in columnDefs corresponds to the column name of the data source, and if it does not exist, this configuration will not take effect.
- The `rowData` key value will not take effect. Because the data source of the table is always controlled by the data source.

---

### Toolbox

The `toolbox` module provides a set of commonly used utility functions.

```python
from ex4nicegui import toolbox
```

#### use_dark

Toggle dark mode

```python
from ex4nicegui import rxui, toolbox as tb
from nicegui import ui


dark = tb.use_dark(False)

rxui.label(lambda: f"Dark mode: {dark.value}")
rxui.button(
    icon=lambda: "sunny" if dark.value else "dark_mode",
    color=lambda: "red" if dark.value else "blue",
    on_click=dark.toggle,
).props("flat round")
```

#### use_breakpoints

Responsive breakpoints

```python
from ex4nicegui import rxui, toolbox as tb
from nicegui import ui


options = {"Phone": 0, "Tablet": 640, "Laptop": 1024, "Desktop": 1280}
bp = tb.use_breakpoints(options)
active = bp.active
is_between = bp.between("Phone", "Laptop")

with ui.card():
    rxui.label(lambda: f"Current Breakpoint: {active.value}")
    rxui.label(
        lambda: f"Is between Phone and Laptop (exclusive): {is_between.value}"
    ).bind_classes({"text-red-500": is_between})

    rxui.label(lambda: f'Phone(0px - 640px): {active.value == "Phone"}').bind_classes(
        {"bg-red-300": lambda: active.value == "Phone"}
    )
    rxui.label(
        lambda: f'Tablet(640px - 1024px): {active.value == "Tablet"}'
    ).bind_classes({"bg-red-300": lambda: active.value == "Tablet"})
    rxui.label(
        lambda: f'Laptop(1024px - 1280px): {active.value == "Laptop"}'
    ).bind_classes({"bg-red-300": lambda: active.value == "Laptop"})
    rxui.label(lambda: f'Desktop(1280px+): {active.value == "Desktop"}').bind_classes(
        {"bg-red-300": lambda: active.value == "Desktop"}
    )
```

#### use_qr_code

Generate QR code

```python
from ex4nicegui import rxui, to_ref, toolbox as tb
from nicegui import ui


text = to_ref("ex4nicegui")
qr_code = tb.use_qr_code(text)

rxui.input(value=text)
rxui.image(qr_code.code).classes("w-20 h-20").props("no-transition")
```