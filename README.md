#etapa 4
##  Instalación y ejecución local
### 1. Clonar el repositorio

```bash
git clone <url-del-repo>
cd heart-disease-mlops
```



### 2. Crear entorno Python y dependencias

```bash
conda create -n ml_venv python=3.10
conda activate ml_venv
pip install pandas scikit-learn joblib matplotlib seaborn fastapi uvicorn evidently
```


### 3. Instalar Minikube

En **PowerShell como Administrador:**

```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing

$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
    [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```

Cerrar y reabrir PowerShell, luego verificar:

```powershell
minikube version
```



### 4. Instalar kubectl

```powershell
winget install -e --id Kubernetes.kubectl
```

Cerrar y reabrir PowerShell, luego verificar:

```powershell
kubectl version --client
```



### 5. Iniciar el cluster de Minikube

```powershell
minikube start --driver=docker
```

Verificar que el cluster está corriendo:

```powershell
minikube status
kubectl get nodes
```

El nodo debe aparecer en estado `Ready`.



### 6. Construir la imagen Docker dentro de Minikube

> Este paso es clave: apunta el Docker de tu terminal al Docker **interno** de Minikube para que el cluster pueda usar la imagen sin necesitar Docker Hub.

```powershell
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker build -t heart-api:v1.0.0 -f docker/Dockerfile .
```

Verificar que la imagen fue creada:

```powershell
docker images | findstr heart-api
```



### 7. Desplegar en Kubernetes

```powershell
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Verificar que el pod está corriendo:

```powershell
kubectl get pods
```

Debe mostrar `STATUS: Running` y `READY: 1/1`.



### 8. Exponer el servicio

En una **terminal aparte**, dejar corriendo:

```powershell
minikube tunnel
```

En la terminal principal verificar que el servicio tiene IP externa:

```powershell
kubectl get svc heart-service
```

Debe mostrar `EXTERNAL-IP: 127.0.0.1`.



### 9. Probar la API

Abrir en el navegador:

```
http://127.0.0.1/docs
```

Usar el endpoint `POST /predict` con este body de ejemplo:

```json
{
  "Age": 52,
  "Sex": "M",
  "ChestPainType": "ATA",
  "RestingBP": 125,
  "Cholesterol": 212,
  "FastingBS": 0,
  "RestingECG": "Normal",
  "MaxHR": 168,
  "ExerciseAngina": "N",
  "Oldpeak": 1.0,
  "ST_Slope": "Up"
}
```

Respuesta esperada:

```json
{
  "heart_disease_probability": 0.0,
  "prediction": 0
}
```



##  Apagar el entorno

Cuando termines de trabajar:

```powershell
# Detener Minikube (conserva el cluster)
minikube stop

# O eliminar el cluster completamente
minikube delete
```
