# Setting Up SSH Keys

Follow the steps below to set up SSH keys:

## 1. Create the `.ssh` Directory
```bash
mkdir -p ~/.ssh
cd ~/.ssh
```

## 2. Generate an SSH Key Pair
```bash
ssh-keygen -b 4096 -t rsa -f pcmagik-zurich-arm-docker-pcmagik-com
```

## 3. Add the Public Key to `authorized_keys`
You can use one of the following methods:

### Method 1: Append the Key
```bash
cat ~/.ssh/pcmagik-zurich-arm-docker-pcmagik-com.pub >> ~/.ssh/authorized_keys
```

### Method 2: Echo the Key
Replace `YOUR_PUBLIC_KEY` with your actual public key:
```bash
echo "YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys
```

## 4. Set Permissions
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

## 5. Install `putty-tools`
```bash
sudo apt install putty-tools
```

## 6. Convert the Private Key to PPK Format
```bash
puttygen ~/pcmagik-zurich-arm-docker-pcmagik-com -o ~/.ssh/pcmagik-zurich-arm-docker-gronioss-pamagik-com.ppk
```

## 7. Copy Keys to the `ubuntu` User's Home Directory
```bash
sudo cp pcmagik-zurich-arm-docker-pcmagik-com /home/ubuntu/
sudo cp pcmagik-zurich-arm-docker-pcmagik-com.pub /home/ubuntu/
```

## 8. Change Ownership of the Keys
```bash
sudo chown ubuntu:ubuntu /home/ubuntu/pcmagik-zurich-arm-docker-pcmagik-com
sudo chown ubuntu:ubuntu /home/ubuntu/pcmagik-zurich-arm-docker-pcmagik-com.pub
```

