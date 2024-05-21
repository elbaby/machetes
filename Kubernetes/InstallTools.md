# Preparación general

```
# Instalar prerrequisitos:
sudo apt-get install -y apt-transport-https ca-certificates curl

# Crear directorio para keyrings (si no existe)
sudo install -v -m 0755 -d /etc/apt/keyrings
```

# Instalar Docker Engine (Docker CE)

```
# Desinstalar paquetes no oficiales
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker \
           containerd runc; do
    sudo apt-get remove $pkg
done

# Clave pública con la que están firmados los paquetes del repositorio
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor --output /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Repositorio firmado
echo "deb [arch="$(dpkg --print-architecture)"\
 signed-by=/etc/apt/keyrings/docker.gpg]\
 https://download.docker.com/linux/ubuntu\
 "$(lsb_release -cs)" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Actualizar repositorios
sudo apt-get update
# Instalar los paquetes de Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin
```

# Instalar Herramientas de Kubernetes

```
# Versión de las herramientas que queremos instalar
KUBEVERSION=1.30

# Clave pública con la que están firmados los paquetes del repositorio
curl -fsSL https://pkgs.k8s.io/core:/stable:/v${KUBEVERSION}/deb/Release.key | \
 sudo gpg --dearmor \
 --output /etc/apt/keyrings/kubernetesV${KUBEVERSION}-apt-keyring.gpg

# Repositorio firmado
echo 'deb [arch="$(dpkg --print-architecture)"\
 signed-by=/etc/apt/keyrings/kubernetesV${KUBEVERSION}-apt-keyring.gpg]\
 https://pkgs.k8s.io/core:/stable:/v${KUBEVERSION}/deb/ /' | \
    sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null

# Instalar kubectl
sudo apt-get update
sudo apt-get install kubectl

```



___
<!-- LICENSE -->
___
<a rel="licencia" href="https://creativecommons.org/licenses/by-sa/4.0/deed.es">
<img alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
<br /><br />
Este documento está licenciado en los términos de una <a rel="licencia"
href="https://creativecommons.org/licenses/by-sa/4.0/deed.es">
Licencia Atribución-CompartirIgual 4.0 Internacional de Creative Commons</a>.
<br /><br />
This document is licensed under a <a rel="license"
href="https://creativecommons.org/licenses/by-sa/4.0/deed.en">
Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<!-- END -->
