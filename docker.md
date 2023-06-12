![Logo](https://i.ibb.co/jhJ5T0p/1468603615docker-logo-removebg-preview.png)

### Installation

Install all the dependency packages using the following command...

```bash
sudo apt-get update
sudo snap install docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Before testing Docker, check the version installed using the following command...

```bash
docker --version
```

Pull an image from the Docker hub using the following command...

```bash
sudo docker run projectName
```

Check if the docker image has been pulled and is present in your system using the following command...

```bash
sudo docker images
```

To display all the containers pulled, use the following command...

```bash
sudo docker ps -a
```

To check for containers in a running state, use the following command...

```bash
sudo docker ps
```

### Docker Uninstall

Docker Uninstall command ...

```bash
sudo snap remove docker
```
