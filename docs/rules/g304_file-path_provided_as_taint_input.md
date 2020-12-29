---
id: g304
title: G304: File path provided as taint input
---

Trying to open a file provided as an input in a variable. The content of this variable might be controlled by an attacker who could change it to hold unauthorised file paths from the system.  In this way, it is possible to exfiltrate confidential information or such.

## Example problematic code:
This code lets an attacker read a `/private/path`
```
package main

import (
	"fmt"
	"io/ioutil"
	"strings"
)

func main() {
	repoFile := "/safe/path/../../private/path"
	if !strings.HasPrefix(repoFile, "/safe/path/") {
		panic(fmt.Errorf("Unsafe input"))
	}
	byContext, err := ioutil.ReadFile(repoFile)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s", string(byContext))
}
```

## Gosec command line output

```
[examples/main.go:11] - G304 (CWE-22): Potential file inclusion via variable (Confidence: HIGH, Severity: MEDIUM)
  > ioutil.ReadFile(repoFile)
```

## The right way
This code panics if `/safe/path` was removed by an attacker
```
package main

import (
	"fmt"
	"io/ioutil"
	"path/filepath"
	"strings"
)

func main() {
	repoFile := "/safe/path/../../private/path"
	repoFile = filepath.Clean(repoFile)
	if !strings.HasPrefix(repoFile, "/safe/path/") {
		panic(fmt.Errorf("Unsafe input"))
	}
	byContext, err := ioutil.ReadFile(repoFile)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s", string(byContext))}
```

## See also

* https://pkg.go.dev/path/filepath?tab=doc#Clean
