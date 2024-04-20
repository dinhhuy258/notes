# Find bugs with the git bisect command

Git Bisect requires just two pieces information before it can start the bug hunt with you:

- A revision where things were definitely good.
- A revision where the bug is present.

![](https://www.git-tower.com/learn/media/pages/git/faq/git-bisect/0e1cea9cd5-1712597625/bisect-overview.png)

```bash
git bisect start
git bisect bad HEAD
git bisect good fcd61994
...
git bisect bad
git bisect good
git bisect reset # Exit the git bisect state and return to the original branch
```

We can automate the process of running git bisect by using script

```bash
#!/bin/bash
# Run the test
command_to_run_your_code

# Save the exit code
exit_code=$?

# Exit with a non-zero exit code if any of the tests fail
if [ $exit_code -ne 0 ]; 
then
  exit 1
fi

# Exit with a zero exit code if the tests succeed
exit 0
```

```bash
git bisect run test.sh
git bisect run yarn lint
```
