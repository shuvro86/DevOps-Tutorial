## Understanding Git workflow
- **Objective**: Explore different `git pull` strategies: merge, rebase.
- **Setup**: Make a change on GitHub in `README.md` and ensure your local branch has a new commit:
  ```bash
  echo "Local change" >> README.md
  git add README.md
  git commit -m "Local change to README"
  ```
- **Merge Strategy** (default):
  - Run:
    ```bash
    git pull origin main
    ```
  - This creates a merge commit if there are divergent changes, combining histories from both branches.
  - Result: A new commit with two parents, preserving the history of both branches.
    ```
    A -- B -- C (main on origin)
          \
            D -- E (local main)

      git pull

      A -- B -- C -- M (main on origin & local main)
          \       /
            D -- E
    ```
  - **Explanation**: A, B, C are commits on the remote main branch. D, E are local commits made on your main branch after you last pulled. git pull fetches C and then merges C with E, creating a new merge commit M. This M commit has two parents: C and E.
- **Rebase Strategy**:
  - Run:
    ```bash
    git pull --rebase origin main
    ```
  - This replays your local commits on top of the fetched commits, creating a linear history.
  - Result: No merge commit; history looks like all changes were made sequentially.
    ```
    A -- B -- C (main on origin)
        \
          D -- E (local main)

    git pull --rebase

    A -- B -- C -- D' -- E' (main on origin & local main)
    ```
  - **Explanation**: A, B, C are commits on the remote main branch. D, E are local commits made on your main branch. git pull --rebase fetches C and then takes your local commits D and E, "rewriting" them as D' and E' on top of C. This creates a cleaner, linear history.

  ## Creating and Resolving a Merge Conflict
- **Objective**: Simulate a merge conflict by editing the same line in two branches.
- Create a file `code.py` in the `main` branch:
  ```bash
  echo "print('Hello, World!')" > code.py
  git add code.py
  git commit -m "Add initial code.py"
  git push origin main
  ```
- Create a new branch `feature-a`:
  ```bash
  git checkout -b feature-a
  ```
- Edit `code.py` in `feature-a` (change line 1):
  ```bash
  echo "print('Hello from Feature A!')" > code.py
  git add code.py
  git commit -m "Update code.py in feature-a"
  git push origin feature-a
  ```
- Switch back to `main` and create another branch `feature-b`:
  ```bash
  git checkout main
  git checkout -b feature-b
  ```
- Edit `code.py` in `feature-b` (same line):
  ```bash
  echo "print('Hello from Feature B!')" > code.py
  git add code.py
  git commit -m "Update code.py in feature-b"
  git push origin feature-b
  ```
- Merge `feature-a` into `main`:
  ```bash
  git checkout main
  git merge feature-a
  git push origin main
  ```
- Attempt to merge `feature-b` into `main`:
  ```bash
  git checkout main
  git merge feature-b
  ```
- Git will report a conflict in `code.py`. Open `code.py`, which will look like:
  ```
  <<<<<<< HEAD
  print('Hello from Feature A!')
  =======
  print('Hello from Feature B!')
  >>>>>>> feature-b
  ```
- Resolve the conflict by editing `code.py` to keep the desired code, e.g.:
  ```bash
  echo "print('Hello from both features!')" > code.py
  ```
- Complete the merge:
  ```bash
  git add code.py
  git commit -m "Resolve merge conflict in code.py"
  git push origin main
  ```