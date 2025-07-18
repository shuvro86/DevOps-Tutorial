# Part 2: Git Operations

## Step 1: Clone Git Repository

- To Clone the repository you created on GitHub, visit the repository page and click the `< > Code` Button.
- Under **Local** select **SSH** and copy the `git@github.com:<your-github-account>/<repository-name>`
- Run the following command to clone using SSH KEY. If you have set a passphrase when generating the SSH KEY, you might be prompted to provide the passphrase:

  ```bash
  git clone <repository-url>
  ```

## Step 2: Pull Changes
- Simulate a change made directly on GitHub:
  - Go to your `git-hub-demo` repository on GitHub.
  - Edit `README.md` online (e.g., add a new line) and commit the change.
- Pull the changes to your local repository:
  ```bash
  git pull origin main
  ```
- Check the updated `README.md`:
  ```bash
  cat README.md
  ```

## Step 3: Create and Manage Branches
- Create a new branch called `feature-branch`:
  ```bash
  git branch feature-branch
  # git branch -h
  ```
- Switch to the new branch:
  ```bash
  git checkout feature-branch
  ```
- Alternatively, create and switch in one command:
  ```bash
  git checkout -b feature-branch
  ```
- Make a change (e.g., add a new file):
  ```bash
  echo "This is a test file." > test.txt
  git add test.txt
  git commit -m "Add test.txt in feature branch"
  ```
- Push the branch to GitHub:
  ```bash
  git push origin feature-branch
  ```
- Merge the branch into `main`:
  - Switch back to `main`:
    ```bash
    git checkout main
    ```
  - Merge `feature-branch`:
    ```bash
    git merge feature-branch
    # Same can be acheived using git pull
    ```
  - Push the updated `main` branch:
    ```bash
    git push origin main
    ```
- Delete the branch (optional):
  ```bash
  git branch -d feature-branch
  git push origin --delete feature-branch
  ```

## Step 4: Understanding Git Pull Strategies
- **Objective**: Explore different `git pull` strategies: merge, rebase, and fast-forward.
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
- **Fast-Forward**:
  - Occurs automatically when there are no divergent changes (e.g., only remote has new commits).
  - Run:
    ```bash
    git pull origin main
    ```
  - If your local branch has no unique commits, Git moves the branch pointer to the latest commit.
  - To force fast-forward explicitly:
    ```bash
    git pull --ff-only origin main
    ```
    - Note: Fails if there are divergent changes.
    ```
    A -- B -- C (main on origin)
    A -- B (local main)

    git pull

    A -- B -- C (main on origin & local main)
    ```
  - **Explanation**: Your local main branch is a direct ancestor of the remote main branch. git pull fetches C and simply moves your local main pointer forward to C. This is a "fast-forward" because no actual merging of divergent histories is required.

## Step 5: Creating and Resolving a Merge Conflict
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
Rolling Back Commits
- **Objective**: Learn how to undo commits using `git reset` and `git revert`.
- **Soft Reset** (undo commit, keep changes in working directory):
  - View commit history:
    ```bash
    git log --oneline
    ```
  - Reset to a previous commit (e.g., undo the last commit):
    ```bash
    git reset --soft HEAD~1
    ```
  - Changes are unstaged but preserved. Re-commit if needed:
    ```bash
    git commit -m "Re-commit changes after soft reset"
    ```
- **Hard Reset** (undo commit and discard changes):
  - Reset to a previous commit and discard changes:
    ```bash
    git reset --hard HEAD~1
    ```
  - Warning: This permanently deletes uncommitted changes and the commit.
- **Revert** (create a new commit to undo changes):
  - Revert the last commit:
    ```bash
    git revert HEAD
    ```
  - This creates a new commit that undoes the changes from the specified commit.
  - Push the revert commit:
    ```bash
    git push origin main
    ```

---
## Part 1: Setting Up Branch Protection in GitHub

### Objective
Learn how to configure branch protection rules to enforce workflows, such as requiring pull request reviews or passing status checks before merging.

### Background
Branch protection rules in GitHub prevent direct pushes to critical branches (e.g., `main`), enforce code reviews, and ensure quality checks. They are available in public repositories with GitHub Free and in public/private repositories with GitHub Pro, Team, Enterprise Cloud, or Enterprise Server. Only users with admin permissions or custom roles with "edit repository rules" permission can manage these rules.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches)

### Steps

1. **Navigate to Repository Settings**
   - Go to your GitHub repository (e.g., `https://github.com/your-username/git-lab-demo`).
   - Click **Settings** > **Branches** (under "Code and automation").

2. **Add a Branch Protection Rule**
   - Under "Branch protection rules," click **Add rule**.
   - In the "Branch name pattern" field, enter `main` (or use a pattern like `*` for all branches or `*release*` for branches containing "release").
   - Configure the following options:
     - **Require a pull request before merging**: Ensures no direct pushes to the branch. All changes must go through a PR.
     - **Require approvals**: Select this and set the number of required approving reviews (e.g., 1 or 2).
     - **Dismiss stale pull request approvals when new commits are pushed**: Ensures approvals are valid only for the current state of the PR. New commits invalidate previous approvals, requiring re-review.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/approving-a-pull-request-with-required-reviews)
     - **Require review from Code Owners**: If a `CODEOWNERS` file exists, ensure designated owners review changes to their code.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)
     - **Require status checks to pass before merging**: Select specific checks (e.g., CI tests) that must pass. Optionally, enable **Require branches to be up to date before merging** to ensure the PR is tested against the latest base branch.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://www.compositional-it.com/news-blog/protecting-your-main-branch-in-github-from-bad-commits/)
     - **Require conversation resolution before merging**: Ensures all review comments are resolved before merging.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://graphite.dev/guides/prevent-merge-without-review-github)
     - **Require linear history**: Prevents merge commits, enforcing a rebase or squash merge for a linear history.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://www.cloudwithchris.com/blog/use-github-branch-protection-rules/)
     - **Include administrators**: Applies rules to admins, preventing them from bypassing requirements.[](https://www.cloudwithchris.com/blog/use-github-branch-protection-rules/)[](https://eliteionic.com/tutorials/requiring-pull-requests-and-reviews-in-your-git-workflow/)
   - Click **Create** or **Save changes**.

3. **Optional: Create a CODEOWNERS File**
   - In the repository root, `.github/`, or `docs/` directory, create a file named `CODEOWNERS`.
   - Example content:
     ```
     # Owners for specific files
     *.py @username1 @team-name
     README.md @username2
     ```
   - Commit the file to the branch you’re protecting (e.g., `main`).
   - This automatically requests reviews from specified users/teams when their code is modified.[](https://www.bornfight.com/blog/how-to-protect-github-projects-from-non-reviewed-code-and-force-code-review-culture/)[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)

4. **Verify Branch Protection**
   - Try pushing a commit directly to `main`:
     ```bash
     echo "Direct push test" >> README.md
     git add README.md
     git commit -m "Test direct push"
     git push origin main
     ```
   - If protection is set, this will fail with an error (e.g., "remote: error: GH006: Protected branch update failed").

---

## Part 2: Creating Pull Requests

### Objective
Learn how to create a pull request to propose changes to a protected branch.

### Steps

1. **Create a Feature Branch**
   - Clone the repository locally:
     ```bash
     git clone https://github.com/your-username/git-lab-demo.git
     cd git-lab-demo
     ```
   - Create and switch to a new branch:
     ```bash
     git checkout -b feature-update-readme
     ```

2. **Make Changes**
   - Edit a file (e.g., `README.md`):
     ```bash
     echo "Adding a new section to README" >> README.md
     git add README.md
     git commit -m "Update README with new section"
     ```

3. **Push the Branch**
   - Push the branch to GitHub:
     ```bash
     git push origin feature-update-readme
     ```

4. **Create a Pull Request**
   - Go to your repository on GitHub.
   - You’ll see a prompt to create a PR for the `feature-update-readme` branch. Click **Compare & pull request**.
   - Set the base branch to `main` and the compare branch to `feature-update-readme`.
   - Add a title (e.g., "Update README with new section") and description.
   - Click **Create pull request**.[](https://graphite.dev/guides/how-to-require-pull-request-reviews-before-merging)[](https://eliteionic.com/tutorials/requiring-pull-requests-and-reviews-in-your-git-workflow/)

---

## Part 3: Performing Code Review

### Objective
Learn how to review code in a pull request, provide feedback, and request changes.

### Steps

1. **Access the Pull Request**
   - In the repository, go to the **Pull requests** tab and open the PR you created.
   - Click the **Files changed** tab to view the changes.

2. **Add Comments**
   - Hover over a line of code in the diff and click the `+` icon to add a comment.
   - Example: Suggest improving the wording in `README.md`:
     ```
     Consider rephrasing this sentence for clarity.
     ```
   - Click **Add single comment** or **Start a review** to group multiple comments.

3. **Submit a Review**
   - Click **Review changes** (top-right of the PR page).
   - Choose one of the following:
     - **Comment**: Provide general feedback without approving or blocking.
     - **Approve**: Approve the changes, allowing the PR to be merged (if all requirements are met).
     - **Request changes**: Request modifications before the PR can be merged.
   - Add a summary comment (e.g., "Looks good, but please clarify the README text").
   - Click **Submit review**.[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/approving-a-pull-request-with-required-reviews)[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)

4. **Respond to Feedback (PR Author)**
   - If changes are requested, make updates in the same branch:
     ```bash
     echo "Updated README with clearer text" >> README.md
     git add README.md
     git commit -m "Address review feedback"
     git push origin feature-update-readme
     ```
   - The PR updates automatically with the new commits.
   - Resolve conversation threads by clicking **Resolve conversation** if the feedback is addressed.[](https://www.softwaretestinghelp.com/github-tutorial/)

---

## Part 4: Approving Pull Requests

### Objective
Learn how to approve a pull request to allow merging.

### Steps

1. **Review Requirements**
   - Ensure all branch protection requirements are met (e.g., required approvals, passing status checks, resolved conversations).[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://graphite.dev/guides/prevent-merge-without-review-github)
   - If "Dismiss stale pull request approvals" is enabled, new commits will invalidate previous approvals, requiring re-review.[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/approving-a-pull-request-with-required-reviews)

2. **Approve the PR**
   - As a reviewer with write or admin permissions (or a code owner, if specified), go to the PR.
   - Click **Review changes**, select **Approve**, and add a comment (e.g., "Changes look good, ready to merge").
   - Click **Submit review**.
   - Note: The PR author cannot approve their own PR if "Require approval of the most recent reviewable push" is enabled.[](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)[](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops)

3. **Handle Blocking Reviews**
   - If a reviewer selects **Request changes**, the PR cannot be merged until the same reviewer approves or the review is dismissed by someone with write/admin permissions.[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/approving-a-pull-request-with-required-reviews)[](https://nira.com/how-to-set-up-github-branch-protection-rules/)
   - To dismiss a review (if allowed), click **Dismiss review** and provide a reason.

---

## Part 5: Merging Pull Requests

### Objective
Learn how to merge a pull request into the protected branch.

### Steps

1. **Verify Mergeability**
   - Ensure all branch protection rules are satisfied (e.g., required approvals, passing status checks, resolved conversations).
   - If the PR is not mergeable, GitHub will display the reasons (e.g., "1 approval required" or "Status checks failing").

2. **Merge the Pull Request**
   - In the PR page, click **Merge pull request** (available if all requirements are met).
   - Choose a merge strategy:
     - **Create a merge commit**: Combines histories with a merge commit (default, used with `--no-ff`).[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request)
     - **Squash and merge**: Combines all commits into a single commit, creating a cleaner history.
     - **Rebase and merge**: Applies commits from the PR branch onto the base branch, maintaining a linear history.
   - Add a commit message (e.g., "Merge PR #1: Update README") and click **Confirm merge**.
   - The changes are now in the `main` branch.[](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request)

3. **Delete the Branch (Optional)**
   - After merging, click **Delete branch** to remove the `feature-update-readme` branch from GitHub.
   - Locally, delete the branch:
     ```bash
     git fetch origin
     git branch -d feature-update-readme
     ```

4. **Test Auto-Merge (Optional)**
   - Enable **Allow auto-merge** in the PR settings to automatically merge the PR once all checks pass and approvals are received.[](https://x.com/github/status/1336360682221133827)[](https://medium.com/%40lauravuo/managing-github-branch-protections-4fa37b36ee4f)
   - To enable:
     - In the PR, click **Enable auto-merge** and select a merge strategy.
     - GitHub will merge the PR when all requirements are met.

---