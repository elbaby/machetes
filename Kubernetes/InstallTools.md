# Preparación general

```
# Instalar prerrequisitos:
sudo apt install -y apt-transport-https ca-certificates curl

# Crear directorio para keyrings (si no existe)
sudo install -v -m 0755 -d /etc/apt/keyrings
```

# Instalar Docker Engine (Docker CE)

```
# Desinstalar paquetes no oficiales
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker \
           containerd runc; do
    sudo apt remove $pkg
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
sudo apt update
# Instalar los paquetes de Docker
sudo apt install docker-ce docker-ce-cli containerd.io \
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
echo "deb [arch="$(dpkg --print-architecture)"\
 signed-by=/etc/apt/keyrings/kubernetesV${KUBEVERSION}-apt-keyring.gpg]\
 https://pkgs.k8s.io/core:/stable:/v${KUBEVERSION}/deb/ /" | \
    sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null

# Instalar kubectl
sudo apt update
sudo apt install kubectl

```

## Instalar múltiples versiones de kubectl

[`kubectl` no puede tener más de un 1 _minor_ número de versión de diferencia
con el cluster](https://kubernetes.io/releases/version-skew-policy/#kubectl).

Por ejemplo, `kubectl` 1.29.x puede administrar un cluster 1.28.x, 1.29.x o
1.30.x, pero no un cluster 1.26.x o 1.27.x

Si tenés que administrar diferentes clusters y al menos un par tienen una
diferencia de versión _minor_ mayor a 2, no podés usar el mismo `kubectl`.

[Acá](https://faun.pub/using-different-kubectl-versions-with-multiple-kubernetes-clusters-a3ad8707b87b)
encontré cómo instalar diferentes versiones usando **[asdf](../Linux/asdf.md)**.

Instalar primero [asdf](../Linux/asdf.md).

Para instalar, por ejemplo, las versiones 1.30 y 1.27 (que nos permitirían
administrar clusters desde versión 1.26.x hasta 1.31.x), hay que configurar las
versiones completas (incluido el _patchlevel_) que se quieren usar (ver listado
[acá](https://kubernetes.io/releases/).
```
kubectlVersion1=1.30.1
kubectlVersion2=1.27.14
asdf install kubectl ${kubectlVersion1}
asdf install kubectl ${kubectlVersion2}
```

Para ver las versiones que tenemos instaladas:
```
$ asdf list kubectl
  1.27.14
  1.30.1
```

Para elegir una versión para usar:
```
$ asdf global kubectl 1.27.14

$ kubectl version --client --short
Client Version: v1.27.14
Kustomize Version: v5.0.1

$ asdf global kubectl 1.30.1

$ kubectl version --client
Client Version: v1.30.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3

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
