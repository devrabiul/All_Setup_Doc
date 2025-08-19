The standard way is to use **NVM** (Node Version Manager).
Here’s how to set it up:

---

## **1️⃣ Install NVM**

```sh
brew install nvm
mkdir ~/.nvm
```

---

## **2️⃣ Configure NVM in zsh**

Edit your `~/.zshrc`:

```sh
nano ~/.zshrc
```

Add this at the bottom:

```sh
export NVM_DIR="$HOME/.nvm"
[ -s "$(brew --prefix nvm)/nvm.sh" ] && \. "$(brew --prefix nvm)/nvm.sh"
```

Save and reload:

```sh
source ~/.zshrc
```

---

## **3️⃣ Install & Switch Node Versions**

* Install latest LTS version:

```sh
nvm install --lts
```

* Install a specific version:

```sh
nvm install 20
nvm install 18
```

* Switch to a version:

```sh
nvm use 20
```

* Set default version for all new terminals:

```sh
nvm alias default 20
```

* List installed versions:

```sh
nvm ls
```

---

💡 With NVM, you can:

* Run an old Node version for legacy projects.
* Use the latest version for new projects.
* Switch instantly without uninstalling anything.