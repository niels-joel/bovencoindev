# Solana token metadata and logo's
This is a Repository with Metadata and images for solana spl tokens and nfts.

Below there is a guide on how to make your own tokens and nfts.
<br><br><br><br>
# How to make solana spl-tokens and nfts, cnfts
How to make a solana spl-token, nft or cnft.

This is for linux, on windows use WSL with linux.

<h3>Table of Contents</h3>
<ul>
<li><a href="https://github.com/niels-joel/Solana-tokens-metadata#install-docker-and-other-dependencies">Install Docker</a></li>
<li><a href="https://github.com/niels-joel/Solana-tokens-metadata#Make-a-Solana-Spl-token">Make a Solana Spl-Token</a></li>
<li><a href="https://github.com/niels-joel/Solana-tokens-metadata#Make-a-nft">Make a NFT</a></li>
<li><a href="https://github.com/niels-joel/Solana-tokens-metadata#Make-a-cnft">Make a CNFT</a></li>
</ul>
<br><br>

## Install Docker and other dependencies
Run these commands to install docker and other tools you need
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Create a docker image with all the needed tools preinstalled
First, create a `Dockerfile` with `nano Dockerfile`
paste this into the `Dockerfile`.
```
# Use a lightweight base image
FROM debian:bullseye-slim

# Set non-interactive frontend for apt
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    unzip \
    build-essential \
    libssl-dev \
    pkg-config \
    nano \
    ca-certificates \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/root/.cargo/bin:$PATH"
RUN rustc --version

# Install Solana CLI
RUN curl -sSfL https://release.anza.xyz/stable/install | sh
ENV PATH="/root/.local/share/solana/install/active_release/bin:$PATH"
RUN echo 'export PATH="/root/.local/share/solana/install/active_release/bin:$PATH"' >> /root/.bashrc
RUN solana --version

# Set Solana config to devnet
RUN solana config set --url https://api.devnet.solana.com

# Install Bun
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"
RUN bun --version

# Install Node.js (LTS) via NodeSource
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && \
    apt-get install -y nodejs

# Install global NPM packages (incl. Metaplex & UMI)
RUN npm install -g \
    typescript \
    @solana/web3.js@1 \
    esrun \
    bs58 \
    @metaplex-foundation/umi \
    @metaplex-foundation/mpl-bubblegum \
    @metaplex-foundation/mpl-token-metadata

# Set working directory
WORKDIR /solana-nft-data

# Default to bash shell
CMD ["/bin/bash"]

```
To save the `Dockerfile` press `ctrl+x` to exit and then press `y` and `Enter` to save

Second, build a docker image with this `Dockerfile`. Run this:
```
docker build -t createtokensandnfts .
```
To build the image. 

*you can change the "createtokensandnfts" to something you like*

## Run the Container
Now that we have build the image, we can create a new docker container with this image.
```
docker run -it --rm -v $(pwd):/solana-nft-data -v $(pwd)/solana-nft-data:/root/.config/solana createtokensandnfts
```
This will run the docker container in the current directory. make sure you are in the right directory before executing this.
To make a new folder use `mkdir foldername`
- *-it* so we hop into the containers shell
- *-v* for mapping our current directory inside the docker container, so that all the work will be saved
- *--rm* so the docker container will be deleted after you exit it. We dont need it anymore because everything we do will be saved in the current directory of the host machine.

### Create enterdockercontainer.sh (optional)
I like to create an executable file that creates the container, so you dont have to run the same code everytime.
create `enterdockercontainer.sh` with 
```
nano enterdockercontainer.sh
```
and paste this into the file.
```
sudo docker run -it --rm -v $(pwd):/solana-nft-data -v $(pwd)/solana-nft-data:/root/.config/solana createtokensandnfts
```
then press `ctrl+x` to exit and then press `y` and `Enter` to save.

To make the `enterdockercontainer.sh`file executable. We need to add an executable parameter to the file.
```
chmod +x enterdockercontainer.sh
```
To execute the file and hop into the new container. Run 
```
./enterdockercontainer.sh
```
