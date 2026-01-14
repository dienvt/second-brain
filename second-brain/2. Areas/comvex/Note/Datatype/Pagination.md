```go


// Parameters holds the pagination parameters to be used in the query and should be transported as a cross-cutting// object.  
// Request
type Parameters struct {  
    OrderCol string  
    // ASC or DESC
    OrderDir string  
    // 1-index
    Page     int
    PerPage  int  
}

  
// Pagination holds the pagination data to be transported as a cross-cutting object.
// Result
type Pagination struct {  
	// Total record match the filter
    Total       int  
    // Count the current page quantity
    Count       int
    CurrentPage int  
    PerPage     int
    // total / perpage + total%perPage ? 1 : 0
    TotalPages  int  
}
```