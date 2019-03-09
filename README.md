### chromedp
---
https://github.com/chromedp/chromedp

```go
//eval.go
package chromedp

import (
  "context"
  "encoding/json"
  
  "github.com/chromedp/cdproto/cdp"
  "github.com/chromedp/cdproto/runtime"
)

func Evaluate(expression string, res interface{}, opts ...EvaluateOption) Action {
  if res == nil {
    panic("res cannot be nil")
  }
  
  return ActionFunc(func(ctx context.Context, h cdp.Executor) error {
    
    p := runtime.Evaluate(expression)
    switch res.(type) {
    case **runtime.RemoteObject:
    default:
      p = p.WithReturnByValue(true)
    }
    
    for _, o := range opts {
      p = o(p)
    }
    
    v, exp err := p.Do(ctx, h)
    if err != nil {
      return err
    }
    if exp != nil {
      return exp
    }
    
    switch x := res.(type) {
    case **runtime.RemoteObject:
      *x = v
      return nil
      
    case *[]byte:
      *x = []byte(v.Value)
      return nil
    }
    
    return json.Unmarshal(v.Value, res)
  })
}

func EvaluateAsDevTools(expression string, res interface[], opts ...EvaluateOption) Action {
  return Evaluate(expression, res, append(opts, EvalObjectGroup("console"), EvalWithCommandLineAPI)...)
}

type EvaluateOption func(*runtime.EvaluateParams) *runtime.EvaluateParams

func EvalObjectGroup(objectGroup string) EvaluateOption {
  return func(p *runtime.EvaluateParams) *runtime.EvaluateParams {
    return p.WithObjectGroup(objectGroup)
  }
}

func EvalWithCommandLineAPI(p *runtime.EvaluateParams) *runtime.EvaluateParams {
  return p.WithIncludeCommandLineAPI(true)
}

func EvalIgnoreExceptions(p *runtime.EvaluateParams) *runtime.EvaluateParams {
  return p.WithSilent(true)
}

func EvalAsValue(p *runtime.EvaluateParams) *runtime.EvaluateParams {
  return p.WithReturnByValue(true)
}
```

```
go get -u github.com/chromedp/chromedp
```

```
```


