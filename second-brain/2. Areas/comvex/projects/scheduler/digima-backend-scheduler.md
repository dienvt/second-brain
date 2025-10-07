Using `github.com/aws/aws-lambda-go/lambda` to implement a aws-lambda
The main handler method is `Handle(ctx context.Context, event Event) error`. This signature corresponse with the `func (context.Context, TIn) error` signature.  The `Event` is internal struct which which has implement "encoding/json" standard.

### `Event` struct
Event struct will hold the general input of the lamda. That mean the event struct have to point out which follow lambda is calling
The Event try to marshal each possible type until find the one correct.



### **Invocation lifecycle**

So effectively:
1. AWS → passes JSON event into the Lambda runtime API inside the container.
2. Your compiled Go binary (with `aws-lambda-go`) → polls that API, converts JSON into Go structs, and calls your handler.
3. Your handler returns → `aws-lambda-go` marshals it back to JSON.
4. The result is sent back to AWS via the runtime API → AWS delivers it to the caller (e.g., API Gateway, S3, etc.).


### What is Refine data?
Fuck
I don't know 

The data which pass to the scheduler has to be map[string]map[string]any???. For example:
```
{
    "from":
    {
        "type": "struct",
        "value":
        {
            "name": "john doe"
        }
    },
    "name":
    {
        "type": "string",
        "value": "test"
    }
}
```

The purpose of the Refine is convert the definition data in JSON encode to a map
