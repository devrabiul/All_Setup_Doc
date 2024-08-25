### Installation

Install Github Desktop with...

```bash
sudo wget https://github.com/shiftkey/desktop/releases/download/release-3.1.1-linux1/GitHubDesktop-linux-3.1.1-linux1.deb
sudo apt-get install gdebi-core 
sudo gdebi GitHubDesktop-linux-3.1.1-linux1.deb
```

### Installation

Setup SSH Key with...

```bash
ssh-keygen
ls
cd /
ls
cd home/
ls
cd username/.ssh/
ls
cat id_rsa.pub
```

Git SSH Connetion

```bash
git config --get remote.origin.url
ssh -T git@github.com

git remote -v
git remote set-url origin git@github.com:username/repositoryname.git
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

#### Remove the .git Directory:
Use the rm command with the -rf flags to forcefully remove the .git directory.

```bash
rm -rf .git
rm -rf *
```
