Hey there!

Last week I faced the need to have different steps to be run when Github Actions was executed. Today I brought you a simple solution to how I managed to achieve this. Be warned, the important content of this post will be the Github Actions Workflow.

```yaml
on: push

jobs:
    project-a-or-project-b:
        runs-on: ubuntu-latest
        name: Project A or Project B

        steps:

        - name: Checkout Main repository
          uses: actions/checkout@v2
          with:
            path: ./mainRepository
    
        - uses: actions/setup-go@v1
          with:
            go-version: '1.9.3' # The Go version to download (if necessary) and use.

        - name: Run Main Repository 
          id: main
          run: |
            cd ./mainRepository &&
            OUTPUT=$(go run main.go | tail -1) &&
            echo "::set-output name=OUTPUT::$OUTPUT"
    
        - name: Run Project A
          if: steps.main.outputs.OUTPUT == 0
          run: |
            cd ./mainRepository/projectA &&
            go run main.go
        
        - name: Run Project B
          if: steps.main.outputs.OUTPUT == 1
          run: |
            cd ./mainRepository/projectB &&
            go run main.go
```

Step by step:

We define a workflow that will be run on a 'push' event. We name this job 'project-a-or-project-b' and we assign it to a ubuntu container with the latest version possible.

Our main program only calculates a random number between 0 and 1. We could say it's mostly a flip-coin program. If we get 0 we will run our 'projectA' program and if we get 1, we run 'projectB'

```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	s1 := rand.NewSource(time.Now().UnixNano())
	r1 := rand.New(s1)
	result := r1.Intn(2)
	fmt.Println(result)
}
```

Inside 'projectA' and 'projectB' there's just a Print indicating which routine is being run

```golang
package main

import "fmt"

func main() {
	fmt.Println("I'm running Project A")
}
```

Our first step on the workflow is going to check-out the main repository in the ./mainRepository folder and then it will install go 1.9.3 in order to run the main program.

Once we get to the 'Run Main Repository' step, we assign it the id 'main' which will allow us later to get it's outputs. So we define an output of the program:

```bash
OUTPUT=$(go run main.go | tail -1)
```

and then we indicate Github Actions that the OUPUT variable is going to be an output on that step:

```bash
echo "::set-output name=OUTPUT::$OUTPUT"
```

Now it's time to know which side our coin landed on:

```yaml
if: steps.main.outputs.OUTPUT == 0
```

We can define an if statement on the steps that will depend on our OUTPUT from the last step. You can get your outputs concatenating the string 'steps.{step_name}.outputs.{step_output}

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/5wqszoyjfzxekhxk3zni.png)

That's it for today. I'll leave you the repo so you can check if you are missing something while trying it

