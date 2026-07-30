#!/usr/bin/env bash
set -euo pipefail

echo "=== [1/6] System update ==="
sudo apt-get update -y
sudo apt-get upgrade -y

echo "=== [2/6] Swap file (2GB) ==="
if [ -f /swapfile ]; then
  echo "Swap file already exists, skipping."
else
  sudo fallocate -l 2G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  echo "Swap file created and enabled."
fi
free -h

echo "=== [3/6] Docker ==="
if command -v docker &> /dev/null; then
  echo "Docker already installed, skipping."
else
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh get-docker.sh
  sudo usermod -aG docker ubuntu
  rm get-docker.sh
  echo "Docker installed. NOTE: log out/in (or start a new SSH session) for group membership to take effect."
fi
docker --version

echo "=== [4/6] K3s ==="
if command -v k3s &> /dev/null; then
  echo "K3s already installed, skipping."
else
  curl -sfL https://get.k3s.io | sh -
  echo "K3s installed."
fi
sudo systemctl status k3s --no-pager | head -5

echo "=== [5/6] kubectl access for the ubuntu user ==="
mkdir -p /home/ubuntu/.kube
sudo cp /etc/rancher/k3s/k3s.yaml /home/ubuntu/.kube/config
sudo chown ubuntu:ubuntu /home/ubuntu/.kube/config
chmod 600 /home/ubuntu/.kube/config
if ! grep -q "KUBECONFIG" /home/ubuntu/.bashrc; then
  echo 'export KUBECONFIG=/home/ubuntu/.kube/config' >> /home/ubuntu/.bashrc
fi
sudo ln -sf /usr/local/bin/kubectl /usr/bin/kubectl 2>/dev/null || true


echo "=== [6/6] Helm ==="
if command -v helm &> /dev/null; then
  echo "Helm already installed, skipping."
else
  curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 -o get_helm.sh
  chmod 700 get_helm.sh
  ./get_helm.sh
  rm get_helm.sh
fi
helm version

echo "=== Bootstrap complete ==="
echo "Run 'source ~/.bashrc' or start a new shell, then verify with: kubectl get nodes"