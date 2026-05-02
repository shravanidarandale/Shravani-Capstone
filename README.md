# Shravani-Capstone
An inventory that recognizes and stores information ids
# Hybrid Inventory Manager

A console-based inventory manager demonstrating a **C data layer** (binary file storage via `fread`/`fwrite`/`fseek`) combined with a **C++ UI layer** (classes + STL).

---

## File Structure

```
hybrid_inventory/
├── include/
│   └── inventory.h          # Item struct + C API declarations (extern "C")
├── src/
│   ├── inventory.c          # C backend: binary file CRUD
│   ├── InventoryManager.h   # C++ class (header-only, wraps C API)
│   ├── InventoryManager.cpp # C++ translation unit (satisfies multi-file rule)
│   └── main.cpp             # Console menu loop
├── Makefile
├── CMakeLists.txt
└── README.md
```

---

## Build & Run

### Option A — Make (recommended)

```bash
make          # compiles C and C++ separately, links into bin/inventory
make run      # build + run in one step
make clean    # remove all build artifacts
```

### Option B — CMake

```bash
mkdir build && cd build
cmake ..
cmake --build .
./inventory
```

> **Note:** The binary file `inventory.dat` is created in the **working directory** where you launch the executable.  
> Both `make run` and running `./bin/inventory` from the project root use the same file.

---

## Menu Options

```
1) Add item      – enter ID, name, quantity, price
2) View item     – look up one item by ID
3) Update item   – overwrite an existing record
4) Delete item   – soft-delete (marked internally, hidden from all views)
5) List all      – show every active item sorted by ID
6) Exit
```

---

## Input Validation Rules

| Field    | Rule                     | Behaviour on invalid input  |
|----------|--------------------------|-----------------------------|
| ID       | Positive integer (> 0)   | Re-prompt with warning      |
| Name     | Non-empty string         | Re-prompt with warning      |
| Quantity | Integer ≥ 0              | Re-prompt with warning      |
| Price    | Float ≥ 0.0              | Re-prompt with warning      |
| Menu     | 1 – 6                    | Re-prompt with warning      |

The program **never crashes** on bad input; it always recovers and asks again.

---

## Persistence

All data is stored in **`inventory.dat`** as a flat array of fixed-size `Item` structs (binary format).  
`fseek` is used for in-place updates and soft-deletes, so the file never grows on update/delete.

---

## Test Cases

The following scenarios were manually verified:

1. **Persistence across restarts**  
   Added items with IDs 1, 2, 3 → chose *Exit* → restarted → chose *List all* → all three items appeared correctly.

2. **Duplicate ID rejection**  
   Added item ID 5 successfully → tried to add another item with ID 5 → received `✘ Failed: ID may already exist` message; only one record for ID 5 exists in the file.

3. **Update persists after restart**  
   Added item ID 10, name "Widget", quantity 50 → chose *Update*, changed quantity to 99 → *Exit* → restarted → *View item* 10 showed quantity 99.

4. **Soft-delete hides item**  
   Added items 20, 21, 22 → deleted item 21 → *List all* showed only 20 and 22; *View item* 21 returned `✘ Item not found or has been deleted`; `inventory.dat` still contains the record but with `is_deleted = 1`.

5. **Invalid input recovery**  
   At the ID prompt, entered `-5`, then `abc`, then `0` → the program re-prompted each time with a warning message → entered `7` → proceeded normally without crashing.

---

## Design Notes

* **Language separation** — `inventory.c` is compiled with `gcc -std=c11`; C++ files with `g++ -std=c++17`. They link into a single executable.
* **`extern "C"` guard** in `inventory.h` allows the header to be included from both C and C++ translation units without name-mangling issues.
* **STL usage** — `std::vector<Item>` holds the listing buffer; `std::sort` orders items by ID before printing.
* **No heap allocation in C layer** — the C backend uses only stack variables and file I/O; memory management is trivially safe.
