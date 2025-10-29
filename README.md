
# 🐲 Pokeneas - Flask + Docker + AWS S3 + Docker Swarm


Integrantes:
- Luis Angel Nerio
- Camilo Salazar 
- María Alejandra Ocampo 

Taller 2 de Arquitectura de Software 
- **Pokeneas** (Pokémon paisas).
Este taller muestra información de Pokeneas, sus imágenes desde AWS S3 y permite ejecutarlo tanto **localmente** como en un **clúster Docker Swarm** desplegado en AWS.

---

##  Acceso público

El proyecto está desplegado en las siguientes direcciones IP (puedes usar cualquiera):

```
http://44.222.253.65:8000
http://54.164.60.154:8000
http://18.207.127.103:8000
http://54.235.231.206:8000
```

Cada dirección corresponde a un nodo del clúster.
Al **recargar la página**, se puede observar que la respuesta proviene de diferentes contenedores, confirmando que el servicio se **actualiza y balancea automáticamente** dentro del Swarm.

---

##  Ejecución local

También puedes correr el proyecto localmente para pruebas o desarrollo.

### 1️ Crear entorno virtual

**Windows:**

```powershell
python -m venv .venv
.venv\Scripts\activate
```

**Linux/Mac:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2️ Instalar dependencias

```bash
pip install -r requirements.txt
```

### 3️ Ejecutar la app

```bash
python app.py
```

Luego abre [http://localhost:8000](http://localhost:8000)

---

##  Ejecutar con Docker

### 1️ Construir imagen

```bash
docker build -t pokeneas .
```

### 2️ Correr contenedor

```bash
docker run -d -p 8000:8000 pokeneas
```

---

##  Despliegue con Docker Swarm (AWS)

###  Paso 1: Crear 4 instancias EC2

* Una será **líder** (`swarm-leader`)
* Tres serán **managers** (`swarm-mgr-1`, `swarm-mgr-2`, `swarm-mgr-3`)
* Abre puertos TCP: `22, 8000, 2377, 7946` y UDP: `7946, 4789`

###  Paso 2: Instalar Docker en todas

```bash
sudo dnf -y update
sudo dnf -y install docker
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
exit
```

(Vuelve a entrar con *Connect* desde AWS)

###  Paso 3: Iniciar Swarm en el líder

```bash
docker swarm init --advertise-addr $(hostname -I | awk '{print $1}')
docker swarm join-token manager
```

###  Paso 4: Unir los 3 managers

```bash
sudo docker swarm join --token <TOKEN_DEL_LIDER> <IP_PRIVADA_DEL_LIDER>:2377
```

###  Paso 5: Verificar desde el líder

```bash
docker node ls
```

---

##  Desplegar el servicio

En el **líder**:

```bash
docker service create \
  --name pokeneas \
  --replicas 10 \
  --publish published=8000,target=8000 \
  --env S3_BUCKET=pokeneas-media-maocampog1 \
  --env AWS_REGION=us-east-1 \
  maocampog1/pokeneas:latest
```

Verifica:

```bash
docker service ls
docker service ps pokeneas
```

---

