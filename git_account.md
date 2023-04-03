### Setup Git accout in linux
Sure, here are the steps to set up a Git account on Linux:

Install Git: Open the terminal and enter the following command to install Git:
```bash
sudo apt-get update
sudo apt-get install git
sudo apt install geomview
```
```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@example.com"
```
```bash
git config --list
```

Generate SSH keys: To use Git over SSH, you'll need to generate an SSH key. Enter the following command to generate a new SSH key:
```bash
ssh-keygen -t rsa -b 4096 -C "youremail@example.com"
```
Add the SSH key to your account: After generating the SSH key, you'll need to add it to your Git account. Copy the contents of the ~/.ssh/id_rsa.pub file and paste it into your Git account's SSH key settings.

That's it! You should now be able to use Git on Linux.

SSH key example ?
Sure, here's an example of an SSH key generated using the RSA algorithm:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDf8Zt1uV7umg0e3u
VlFTdiy47lKGmPR1mbFKn3mWzLHhLmZvrRg/w+aYZQc29nJ5d5BafF
DY6S/wU6cNlZdL2Qa4D4wj4/4wWUVvc8J87y4YX+6OIFc6UZd35UZJ
2dh+4cw4ObMwOy8xsgJL6pzwz1+gqAXJWAnBZIHfHbR5UKMVrrc0k0
xJyfJS1tXJmAlz2QcdmtwbEeA+wZ7JL1nq3g7V8xvHdZrXetPAPtaT
cczW8Iv/BCpxNlCFsLbC1I7x72EoNQOdEBtX2ITt/e8D7zKZtO1C7V
3q0H0J7eCgOsB+LR9IuOovtu8GVKpZilvNOIwx7G/FwL user@example.com
```
This SSH key can be added to your Git account to allow you to authenticate with Git over SSH.


==========================================================

### Step 1: Create Repository
First, open up GitHub hosting service and click on the “+” icon to create a new repository:
https://linuxhint.com/clone-repo-with-ssh-key-in-git/

### Step 2: Open Git Bash
Now, open the “Git Bash” on your system using the “Startup” menu:

### Step 3: Generate SSH key
Next, execute the below-provided command to generate the SSH public-private key pair:

```bash
ssh-keygen
ls -al ~/.ssh
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
clip < ~/.ssh/id_rsa.pub
ssh -T git@github.com
```
### Step 4: Clone Repository
Execute the “git clone” command with copied URL to clone the repository:

```bash
git clone git@github.com:GitUser0422/Cloning_branch.git
```

## Conclusion
To clone the repository with the SSH key in Git, first create a new repo on “GitHub”, then open “Git Bash” and generate the SSH key using the “$ SSH key-gen” command. Launch the SSH agent, run the “$ ssh-add ~/.ssh/id_rsa”, add the SSH public key into the agent, and copy it. Next, go to the “GitHub” settings, and add the SSH key. Lastly, open the repository, copy “SSH URL” and execute the “$ git clone” command with copied URL. In this blog, we have illustrated the method of cloning Git repo with SSH key in Git.
