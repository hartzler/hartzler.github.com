+++
date = "2014-12-12T08:50:11-06:00"
draft = false
title = "Retry with Timeout"
tags = ["code", "go"]
+++

Retry with timeout can be a tricky little function to get right.  Here I want to highlight go's select statement which makes short work of what can be a tangled code path in many other languages.


While checking out hashicorp's excellent [terraform](https://github.com/hashicorp/terraform) I noticed this gem:


```go
// retryFunc is used to retry a function for a given duration
func retryFunc(timeout time.Duration, f func() error) error {
	finish := time.After(timeout)
	for {
		err := f()
		if err == nil {
			return nil
		}
		log.Printf("Retryable error: %v", err)

		select {
		case <-finish:
			return err
		case <-time.After(3 * time.Second):
		}
	}
}
```

The stdlib time package has an After method

```
time.After(timeout)
```

which creates a channel that will be closed after a given duration.  Go's select lets you wait on several channels at the same time which is used here to return after the passed duration or continue the loop after 3 seconds.
