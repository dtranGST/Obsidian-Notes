# ⚖️ Table Variable vs. Temp Table

### ✅ **Table Variable (`@MyTable`)**

Best for **small, lightweight sets**.

- **Pros**
    
    - Lives only in memory (though spills to tempdb if too large).
        
    - Simple to declare and use inline.
        
    - Auto-cleans at end of batch/scope (no need to drop).
        
    - Good for < **1,000 rows** (rule of thumb).
        
- **Cons**
    
    - No **statistics** → SQL Server always estimates **1 row**.
        
    - Can cause **bad execution plans** on joins/filters with large row counts.
        
    - Limited indexing (must be declared inline via PK, UNIQUE, or `INDEX`).
        

👉 Use when:

- Small row counts (parameter passing, small staging, splitting values).
    
- Inside stored procedures when you just need a scratchpad.
    
- You want scope-limited data with minimal overhead.
    

---

### ✅ **Temporary Table (`#TempTable`)**

Best for **large or intermediate result sets**.

- **Pros**
    
    - Supports **statistics** → SQL Server knows how many rows you really have.
        
    - Can create **clustered & nonclustered indexes** after creation.
        
    - Better for **complex joins, aggregations, large inserts**.
        
    - Can persist across multiple stored procedure calls (if passed by session).
        
- **Cons**
    
    - Created in `tempdb` → slightly more overhead (especially for small sets).
        
    - Must be explicitly dropped (though auto-drops when connection closes).
        
    - Slightly more verbose than table variables.
        

👉 Use when:

- Dealing with **10k–10M rows**.
    
- Query performance matters and you need optimizer to pick smart plans.
    
- You need **advanced indexing** (filtered, multiple nonclustered, etc.).
    
- Intermediate staging in **ETL pipelines** or reporting.
    

---

### ✅ Quick Comparison

| Feature                  | Table Variable (`@`)              | Temp Table (`#`)                |
| ------------------------ | --------------------------------- | ------------------------------- |
| Statistics               | ❌ No                              | ✅ Yes                           |
| Indexing                 | Limited (PK/Unique, inline INDEX) | Full support                    |
| Scope                    | Batch / procedure only            | Connection (or explicit drop)   |
| Performance (small sets) | ✅ Faster                          | Slight overhead                 |
| Performance (large sets) | ❌ Can degrade                     | ✅ Scales better                 |
| Best For                 | < 1k rows, lightweight ops        | 10k+ rows, joins, heavy queries |

---

### ⚡ **Rule of Thumb**

- If you expect only a **handful of rows** → use a **table variable**.
    
- If you expect **thousands to millions of rows** → use a **#temp table**.
    
- If you’re not sure → start with a **#temp table** for safety, then optimize later.