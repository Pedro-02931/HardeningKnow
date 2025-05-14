---
icon: globe
---

# Globals Constants and Preamble

{% code overflow="wrap" %}
```python
import sqlite3
from typing import Optional, List, Any, Tuple, Dict, Union

def jit(signature_or_function=True, nopython=True, nogil=True, cache=True, parallel=True, error_model='python', fastmath=True, locals={}, **options):
  if callable(signature_or_function):
    return signature_or_function
  else:
    def decorator(func):
      return func
    return decorator
```
{% endcode %}

* `import sqlite3`:  Provides an interface for working with SQLite databases, that is a lightweight disk-based database that doesn't require a separate server process used for embedded applications or small to medium-sized datasets where high concurrency isn't the primary concern.
* **`from typing import Optional, List, Any, Tuple, Dict, Union`**: This line imports several type hints from the `typing` module, ant these hints are used to indicate the expected data types of variables, function arguments, and return values. This improves code readability and helps with static analysis, but doesn't affect the runtime behavior of the Python code.
*   **`def jit(...)`**: This defines a function named `jit` which is intended to act as a decorator, that are a way to modify or enhance functions or methods in a reusable manner.&#x20;

    * **`signature_or_function=None`**: This argument is designed to optionally accept either a type signature (specifying the expected input and output types of the function being decorated) or the function itself.
    * **`nopython`**: This flag, commonly seen in JIT compilers like Numba, suggests whether the compiled code should strictly avoid using the Python interpreter.&#x20;
    * **`nogil`**: The Global Interpreter Lock (GIL) in Python restricts true multi-threading parallelism in CPU-bound tasks.&#x20;
    * **`cache`**: This flag indicates whether the compiled version of the function should be cached (e.g., on disk) so that it doesn't need to be recompiled on subsequent calls with the same signature.&#x20;
    * **`forceobj`**: In a real JIT context, this flag might force the compiler to fall back to using Python objects even if more efficient native code could be generated.&#x20;
    * **`parallel`**: This flag enabling automatic parallelization of the code within the function, if possible.&#x20;
    * **`error_model`**: This parameter likely dictates how errors during compilation or execution should be handled.
    * **`fastmath`**: This flag often relates to enabling aggressive mathematical optimizations that might sacrifice some numerical precision for speed.&#x20;
    * **`locals_={}`**: This argument allows passing a dictionary of local variables that might be needed during compilation.
    * **`**options`**: This allows passing arbitrary keyword arguments to the decorator, providing flexibility for future extensions or specific compiler options.

    > "Just-In-Time" compilation is a technique used in HPC to improve performance by compiling code to machine code during runtime, often specializing it for the specific data being processed.

The @jit decorator serves as a placeholder to illustrate the concept of using JIT compilation to potentially enhance the performance of the tree operations. In HPC environment, the libries like Numpa or Cython with configured flags could significantly speed up computationally intensive parts of the tree manipulation logic.

In real HPC scenarios, you need might consider:

* If the tree data fits in memory, using an in-memory database system could provide much faster access.
* For very large trees that don't fit on a single machine, a distributed database system designed for high performance and scalability would be necessary.
* Depending on the specific operations performed on the trees, custom in-memory data structures optimized for those operations might be used instead of a general-purpose database.
* For read-heavy workloads, storing the tree data in files with appropriate indexing and using libraries that allow parallel reading could be an option.
* Implement caching mechanisms at the application level to reduce the number of direct database accesses.
* If the tree structure allows, consider partitioning the data into multiple SQL databases that can be accessed by different processes, reducing contention.
* For production, carefully design the application to minimize write contention or consider using write-ahead logging (WAL) mode in SQLite for better concurrency.
* Ensure the SQL database file is stored on a high-performance parallel file system accessible to all compute nodes.

<pre class="language-python" data-overflow="wrap"><code class="lang-python"><strong>class cython
</strong>  int = int
  double = float
  bint = bool
  char = str
</code></pre>

* **`class cython:`**: Is a programming language that allows writing C extensions for Python, thats often used to achieve significant performance gains by compiling parts of the Python code (especially numerical or computationally intensive sections) into optimized C code.
* **`int = int`**, **`double = float`**, **`bint = bool`**, **`char = str`**: These lines simply assign Python's built-in types (`int`, `float`, `bool`, `str`) to the corresponding names that are commonly used as type declarations in Cython syntax.

In a real scenario, parts of the `BaseTree` class or its subclasses, especially methods involving complex data manipulation or traversal, could potentially be rewritten in Cython to leverage the speed of compiled C code. The type aliases provided here (`cython.int`, `cython.double`, etc.) are used in Cython to declare the types of variables, which allows the Cython compiler to generate more efficient C code.

In a production HPC environment, if performance of the tree operations becomes a bottleneck, one could consider using Cython to rewrite critical parts of the code. This would involve:

* Profile the code to pinpoint the functions or methods that consume the most time.
* Create `.pyx` files containing the Python code with Cython type declarations.
* Use the Cython compiler to translate the `.pyx` files into C source code, which is then compiled into Python extension modules.
* Import and use these extension modules in the main Python application.

> Using Cython can be particularly effective for operations involving loops, numerical calculations, or interactions with C libraries, which are common in HPC workloads.

{% code overflow="wrap" %}
```python
NULL_PTR: cython.int = -1
_DEFAULT_DB_PATH = "ultimate_trees_final.db
```
{% endcode %}

* **`NULL_PTR: cython.int = -1`**: The type hint `: cython.int` indicates that this constant is intended to represent an integer value, aligning with the use of the `cython` class for type declarations.&#x20;
  * In the context of tree data structures, `NULL_PTR` is likely used to represent a null or non-existent pointer or ID, for example, indicating that a node has no parent or no children in certain positions.
  * The specific value (`-1` here) is less important than the consistent interpretation of that value throughout the code.
* **`_DEFAULT_DB_PATH = "ultimate_trees_final_v2.db"`**: The leading underscore in the name conventionally that this is an internal constant within the module and specifies the default file path for the SQLite database that will be used to store the tree data if no path is explicitly provided when creating an instance of the `BaseTree` class or its subclasses.
  * In real scenarios, we must set an environment variable before running the script or atributte var name using theses env var.
  * We can use a separate configuration file (e.g., in JSON or YAML format) can store the database path and other settings.
  * The choice of storage location (local disk vs. shared file system) would depend on the scale and concurrency requirements of the HPC application.

In the context of distributed memory HPC systems, representing null pointers or the absence of data in a consistent way across processes is important for data management and communication&#x20;

> While `-1` is a simple representation, but more complex systems might use special sentinel values or dedicated data structures to handle null or missing data, but this is a just didatic content.

Using default paths can be convenient for testing and it's often necessary to configure these paths through environment variables, configuration files, or command-line arguments to manage data across different jobs and file systems in the HPC cluster.

{% code overflow="wrap" %}
```python
class BaseTree:
    TABLE_NODES: str
    TABLE_META: str
    ROOT_ID_KEY = "root_id"

  def __init__(self, db_path: Optional[str] = None, tree_name: str = "base"):
    self.conn = sqlite3.connect(db_path if db_path is not None else _DEFAULT_DB_PATH)
    self.conn.execute("PRAGMA foreign_keys = ON;")

    self.TABLE_NODES = f"{tree_name}_nodes"
    self.TABLE_META = f"{tree_name}_metadata"

    self._create_tables_virtual()
    self.root_id = self._get_metadata(self.ROOT_ID_KEY, NULL_PTR)

  [...]
```
{% endcode %}

* **`class BaseTree:`**: Base classes in object-oriented programming provide a common interface and implementation that can be inherited by more specialized subclasses.&#x20;

> It contains an abstract method (`_create_tables_virtual`).

* **`TABLE_NODES: str`**, **`TABLE_META: str`**: These are class attributes intended to store the names of the SQLite tables that will hold the tree nodes and metadata, respectively.&#x20;
  * The actual table names will be constructed based on the `tree_name` provided during object initialization.&#x20;
  * The type hints `: str` indicate that these attributes are expected to hold string values.
* **`ROOT_ID_KEY = "root_id"`**: This string will be used as the key to store and retrieve the ID of the root node of the tree from the metadata table.
* **`def __init__(self, db_path: Optional[str] = None, tree_name: str = "base"):`**: This is the constructor (initializer) method of the `BaseTree` class, and it's called automatically when a new `BaseTree` object (or an object of a subclass) is created.
  *   **`db_path: Optional[str] = None`**: This parameter allows specifying the path to the SQLite database file

      > If no path is provided, the `_DEFAULT_DB_PATH` constant will be used.
  * **`tree_name: str = "base"`**: This name will be used to construct the names of the database tables, making it possible to store multiple trees in the same database.
  * **`self.conn = sqlite3.connect(db_path if db_path is not None else _DEFAULT_DB_PATH)`**:  It uses the provided `db_path` if it's not `None`, otherwise it uses the `_DEFAULT_DB_PATH`.
  * **`self.conn.execute("PRAGMA foreign_keys = ON;")`**: This line executes an SQL pragma command to enable foreign key constraints in the SQLite database for this connection and this is used to help maintain data integrity by ensuring relationships between tables are valid.
  * **`self.TABLE_NODES = f"{tree_name}_nodes"`**: This line constructs the name for the table that will store the tree nodes by combining the `tree_name` with the suffix `_nodes` using an f-string.
  * **`self.TABLE_META = f"{tree_name}_metadata"`**: This line constructs the name for the metadata table thats defines if is a son, father, black,eg.
  *   **`self._create_tables_virtual()`**: It's an internal method intended for use within the class or its subclasses.&#x20;

      > This one is not implemented in the `BaseTree` class, making it an abstract method that subclasses must implement to define the specific structure of their tree node tables.
  * **`self.root_id = self._get_metadata(self.ROOT_ID_KEY, NULL_PTR)`**: This line retrieves the ID of the root node of the tree from the metadata table using the `_get_metadata` method. It uses the `ROOT_ID_KEY` to identify the root ID and provides `NULL_PTR` as the default value if the root ID hasn't been set yet.

{% code overflow="wrap" %}
```python
[...]
  def _create_tables_virtual(self):
    raise NotImplementedError()

  def _create_metadata_table_if_not_exists(self):
    with self.conn:
      self.conn.execute(f'''
        CREATE TABLE IF NOT EXISTS {self.TABLE_META} (
          meta_key TEXT PRIMARY KEY,
          meta_value ANY
        )''')
      cursor = self.conn.execute(f"SELECT meta_value FROM {self.TABLE_META} WHERE meta_key = ?", (self.ROOT_ID_KEY,))
      if cursor.fetchone() is None:
        self.conn.execute(f"INSERT INTO {self.TABLE_META} (meta_key, meta_value) VALUES (?, ?)",
        (self.ROOT_ID_KEY, NULL_PTR))
[...]
```
{% endcode %}

*   **`def _create_tables_virtual(self):`**: This method is intended to be overridden by subclasses to define the schema of the table that will store the nodes of the specific type of tree being implemented (e.g., binary tree, N-ary tree).&#x20;

    > The term "virtual" here (though not a formal Python keyword in this context) emphasizes that this method is meant to be implemented in derived classes.&#x20;
    >
    > The `raise NotImplementedError()` line indicates that the `BaseTree` class itself doesn't provide an implementation for creating the node table, forcing subclasses to provide their own.
* **`def _create_metadata_table_if_not_exists(self):`**: This internal method is responsible for creating the metadata table in the SQLite database if it doesn't already exist. This table will be used to store general information about the tree, such as the ID of the root node.
  *   **`with self.conn:`**: This statement ensures that the database transaction is properly managed.&#x20;

      > If an error occurs within the block, the transaction will be rolled back.&#x20;
      >
      > If the block completes successfully, the changes will be committed to the database.
  * **`self.conn.execute(f'''...''')`**: This line executes an SQL `CREATE TABLE IF NOT EXISTS` statement to create the metadata table.
    * `CREATE TABLE IF NOT EXISTS {self.TABLE_META}`: This command creates a table with the name stored in `self.TABLE_META` only if a table with that name doesn't already exist.
    * `(meta_key TEXT PRIMARY KEY, meta_value ANY)`: This defines the columns of the metadata table:
      * `meta_key TEXT PRIMARY KEY`: A column named `meta_key` that stores text values and is the primary key of the table to ensures that each metadata key is unique.
      * `meta_value ANY`: A column named `meta_value` that can store values of any data type (text, numbers, etc).
  * **`cursor = self.conn.execute(...)`**: This line executes an SQL `SELECT` statement to check if the `ROOT_ID_KEY` already exists in the metadata table.
  * **`if cursor.fetchone() is None:`**: If the query returns no rows (meaning the `ROOT_ID_KEY` doesn't exist), the code inside the `if` block is executed.
  *   **`self.conn.execute(...)`**: This line executes an SQL `INSERT INTO` statement to add the `ROOT_ID_KEY` with the default value `NULL_PTR` into the metadata table.&#x20;

      > This ensures that the root ID is always present in the metadata, even if a root node hasn't been created yet.

{% code overflow="wrap" %}
```python
[...]
  def _get_metadata(self, key: str, default_value: Any = None) -> Any:
    cursor = self.conn.execute(f"SELECT meta_value FROM {self.TABLE_META} WHERE meta_key = ?", (key,))
    row = cursor.fetchone()
    return row[0] if row else default_value
[...]
```
{% endcode %}

* **`def _get_metadata(self, key: str, default_value: Any = None) -> Any:`**: This internal method retrieves a metadata value from the `TABLE_META` based on a given key.
  * **`cursor = self.conn.execute(...)`**: It executes an SQL `SELECT` statement to retrieve the `meta_value` associated with the provided `key`.
  * **`row = cursor.fetchone()`**: This fetches the first (and should be only) row returned by the query.
  * **`return row[0] if row else default_value`**:&#x20;
    * If a row is found (meaning the key exists), it returns the value in the first column (`meta_value`).&#x20;
    * If no row is found, it returns the `default_value` that was provided (which is `NULL_PTR` in the case of retrieving the root ID for the first time).

{% code overflow="wrap" %}
```python
def _set_metadata(self, key: str, value: Any):
    with self.conn:
      self.conn.execute(f"INSERT OR REPLACE INTO {self.TABLE_META} (meta_key, meta_value) VALUES (?, ?)", (key, value))
    if key == self.ROOT_ID_KEY:
      self.root_id = value

  # close function is obvious and unnecessary explain!
```
{% endcode %}

*   **`def _set_metadata(self, key: str, value: Any):`**: This internal method sets or updates a metadata value in the `TABLE_META` for a given key.

    * **`with self.conn:`**: Ensures proper transaction management.
    * **`self.conn.execute(...)`**: Executes an SQL `INSERT OR REPLACE INTO` statement. This command will insert a new row if the `key` doesn't exist, or update the existing row if the `key` already exists (since `meta_key` is the primary key).
    *   **`if key == self.ROOT_ID_KEY:`**:&#x20;

        * If the key being set is the `ROOT_ID_KEY`, it updates the `self.root_id` instance attribute to reflect the new root ID.





Using a base class like `BaseTree` promotes code reuse and provides a common interface for interacting with these structures, even if their underlying implementations (e.g., how nodes are stored and linked) differ. Subclasses can then implement specific tree types (like binary trees or k-d trees) by inheriting from `BaseTree` and providing their own implementations for methods like `_create_tables_virtual` and methods for inserting, deleting, and searching nodes.

> For this annotation book, I will create a 12 tree algorithms, and this its just a OOP base for all!

Storing metadata (like the root node ID) separately can be useful for managing the overall state and properties of the tree structure in a database, and this is analogous to how HPC simulations or data processing pipelines might store metadata about the simulation parameters, input data versions, or output file locations.

In production environment, you must consider:

* The structure of the `_nodes` table would need to be carefully designed in subclasses based on the specific requirements of the tree (e.g., columns for parent ID, child IDs, data payload).
  * Using appropriate data types and indexes in the SQL schema is crucial for performance, especially for search operations.
*   Enabling foreign keys (`PRAGMA foreign_keys = ON;`) is good practice and must be used in any database environment, including a production setting, to ensure data integrity when managing relationships between nodes (e.g., between parents and children).

    > I know, its obvioius, but dont doubt the stupid human behavior ¯\\\_(ツ)\_/¯
* In a production HPC application, more robust error handling would be needed, especially around database operations.&#x20;
  * This might involve using try-except blocks to catch potential `sqlite3` exceptions and logging errors appropriately.
* For applications that frequently access the database, especially in a parallel environment (even with SQLite), using a connection pool can help improve performance by reusing database connections instead of creating new ones for each operation.
* In more complex scenarios, explicit transaction management (using `conn.begin()`, `conn.commit()`, `conn.rollback()`) might be necessary.
* If the `meta_value` column in the metadata table needs to store complex Python objects, those objects would need to be serialized (e.g., using `pickle` or `json`) before being stored in the database and deserialized when retrieved.&#x20;
  * This can introduce performance overhead and potential security considerations. For production, it's often better to store metadata in a more native database format if possible.
