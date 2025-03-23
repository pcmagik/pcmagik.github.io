





mkdir -p ~/.ssh

cd .ssh
ssh-keygen -b 4096 -t rsa -f pcmagik-zurich-arm-docker-pcmagik-com

cat ~/.ssh/pcmagik-zurich-arm-docker-pcmagik-com.pub >> ~/.ssh/authorized_keys
lub
echo "TWÓJ_KLUCZ_PUBLICZNY" >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

sudo apt install putty-tools

puttygen ~/pcmagik-zurich-arm-docker-pcmagik-com -o ~/.ssh/pcmagik-zurich-arm-docker-gronioss-pamagik-com.ppk



sudo cp pcmagik-zurich-arm-docker-pcmagik-com /home/ubuntu/
sudo cp pcmagik-zurich-arm-docker-pcmagik-com.pub /home/ubuntu/


sudo chown ubuntu:ubuntu pcmagik-zurich-arm-docker-pcmagik-com
sudo chown ubuntu:ubuntu pcmagik-zurich-arm-docker-pcmagik-com.pub


