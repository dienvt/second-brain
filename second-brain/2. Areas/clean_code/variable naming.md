**To name a function, use a description of the return value** A function returns a value, and the function should be named for the value it returns. For example, `cos()`, `customerId.Next()`, `printer.IsReady()`, and `pen.CurrentColor()` are all good function
names that indicate precisely what the functions return.

**To name a procedure, use a strong verb followed by an object** A procedure with functional cohesion usually performs an operation on an object. The name should reflect what the procedure does, and an operation on an object implies a verb-plus-object name. PrintDocument(), CalcMonthlyRevenues(), CheckOrderlnfo(), and Repagi-nateDocument() are samples of good procedure names.

**Use opposites precisely** Using naming conventions for opposites helps consistency, which helps readability. Opposite-pairs like first/last are commonly understood. Opposite-pairs like FileOpen() and FileClose() are not symmetrical and are confusing. Here are some common opposites:
* add/remove 
* increment/decrement 
* open/close
* begin/end
* insert/delete
* show/hide
* create/destroy 
* lock/unlock
* source/target
* first/last
* min/max
* start/stop
* get/put
* next/previous
* up/down
* get/set
* old/new

**Boolean variable prefix** 
- "is": Indicates a state or condition (e.g., `isLoggedIn`, `isAvailable`). 
- "has": Indicates possession or presence (e.g., `hasPermission`, `hasError`). 
- "can": Indicates capability or ability (e.g., `canExecute`, `canModify`). 
- "should": Indicates a conditional requirement (e.g., `shouldUpdate`, `shouldBeDeleted`). 
- "are": Used when dealing with multiple objects (e.g., `areConditionsMet`, `arePointersNull`).
Prefer positive names that reflect the true state (e.g., `isAvailable` instead of `isNotAvailable`).