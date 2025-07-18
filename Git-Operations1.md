# Part 1: Setting Up Git and GitHub

## Step 1: Install Git

### On Windows
Download Git from [git-scm.com](https://git-scm.com/downloads/win) and run the installer.

### On macOS
Run the following command in the terminal: `brew install git`

### On Linux
Run the following command in the terminal: `sudo apt-get install git`

Once installed open the terminal and run the following command to verify the installation: `git --version`

## Step 2: Configure Git

   - Set your name and email, which will be associated with your commits:
     ```bash
     git config --global user.name "Your Name"
     git config --global user.email "your.email@example.com"
     ```
   - Verify the configuration:
     ```bash
     git config --global --list
     ```

## Step 3: Generate ssh key for git

- Generate an SSH key:
  ```bash
  ssh-keygen -t ed25519 -C "your.email@example.com"
  ```
  Press Enter to accept defaults.
  
- Copy the public key for git:
  ```bash
  cat ~/.ssh/id_ed25519.pub
  ```

## Step 4: Create a GitHub Account
- Go to the [GitHub Signup](https://github.com/signup) page and create an account if you don't have one with the necessary information.
- For easier Git authentication, add the ssh key to GitHub. Copy previously copied ssh public key, then add it to GitHub under **Settings > SSH and GPG keys > New SSH key**.
## Step 5: Create a GitHub Repository
- On the top right corner of the GitHub page, click on the `+` button and select `New repository`.  
- Enter the repository name and description.
- Select the repository visibility.
- Click on the `Create repository` button.
- Once the repository is created, you will be redirected to the repository page.
- run `git init` to initialize the repository.
- run `git remote add origin <repository-ssh-url>` to add the remote repository. 

## Step 6: Make Changes and Commit

- Create a new file or edit an existing one (e.g., `README.md`):
  ```bash
  echo "# My First Git hub" > README.md
  ```
- Check the status of your repository:
  ```bash
  git status
  ```
- Stage the changes:
  ```bash
  git add README.md
  ```
- Commit the changes with a message:
  ```bash
  git commit -m "Update README with hub title"
  ```
- Push your committed changes to the remote repository:
  ```bash
  git push origin main
  ```