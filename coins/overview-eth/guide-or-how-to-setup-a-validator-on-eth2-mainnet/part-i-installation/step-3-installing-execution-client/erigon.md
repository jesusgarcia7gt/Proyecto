# Erigon

{% hint style="info" %}
**Erigon** - Sucesor de OpenEthereum, Erigon es una implementación de Ethereum (también conocido como "cliente Ethereum"), en la frontera de la eficiencia, escrita en Go.
{% endhint %}

## Descripción General

#### Enlaces oficiales

| El Tema         | Enlace|
| -------------   | -------------------------------------------------------------------------------------------------------- |
| Lanzamientos    | [https://github.com/ledgerwatch/erigon/releases](https://github.com/ledgerwatch/erigon/releases)         |
| Documentación   | [https://erigon.readthedocs.io/en/latest/index.html](https://erigon.readthedocs.io/en/latest/index.html) |
| Sitio Web       | [https://erigon.substack.com](https://erigon.substack.com/)                                              |

### 1. Configuración Inicial

Cree un usuario de servicio para el servicio de ejecución, cree un directorio de datos y asigne la propiedad.

```bash
sudo adduser --system --no-create-home --group execution
sudo mkdir -p /var/lib/erigon
sudo chown -R execution:execution /var/lib/erigon
```

Instalar dependencias.

```bash
sudo apt install curl libsnappy-dev libc6-dev jq libc6 unzip -y
```

### 2. Instalar Binarios

* La descarga de archivos binarios suele ser más rápida y cómoda.
* Construir a partir de código fuente puede ofrecer una mejor compatibilidad y está más alineado con el espíritu de FOSS (software gratuito de código abierto).
<details>

<summary>Option 1 - Download binaries</summary>

Ejecute lo siguiente para descargar automáticamente la última versión de Linux, descomprimir y limpiar.

```bash
RELEASE_URL="https://api.github.com/repos/ledgerwatch/erigon/releases/latest"
BINARIES_URL="$(curl -s $RELEASE_URL | jq -r ".assets[] | select(.name) | .browser_download_url" | grep linux_amd64)"

eco URL de descarga: $BINARIES_URL

cd $HOME
wget -O erigon.tar.gz $BINARIES_URL
tar -xzvf erigon.tar.gz -C $HOME
rm erigon.tar.gz
```

Instale los binarios.

```bash
sudo mv $HOME/erigon /usr/local/bin/erigon
```

</details>

<details>

<summary>Option 2 - Build from source code</summary>

Instale las dependencias de Go. Última versión [disponible aquí](https://go.dev/dl/).

```bash
wget -O go.tar.gz https://go.dev/dl/go1.20.5.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go.tar.gz
echo export PATH=$PATH:/usr/local/go/bin >> $HOME/.bashrc
source $HOME/.bashrc
```

Verifique que Go esté instalado correctamente verificando la versión y los archivos de limpieza.

```bash
go version
rm go.tar.gz
```

Instalar dependencias de compilación.

```bash
sudo apt-get update
sudo apt install build-essential git
```

Construye el binario.

```bash
mkdir -p ~/git
cd ~/git
git clone https://github.com/ledgerwatch/erigon.git
cd erigon
git fetch --tags
# Get latest tag name
latestTag=$(git describe --tags `git rev-list --tags --max-count=1`)
# Checkout latest tag
git checkout $latestTag
make erigon
```

Intala el binario

<pre class="language-bash"><code class="lang-bash"><strong>sudo cp $HOME/git/erigon/build/bin/erigon /usr/local/bin
</strong></code></pre>

</details>

### **3. Instalar y configurar systemd**

Cree un **archivo de unidad systemd** para definir su configuración `execution.service`.

```bash
sudo nano /etc/systemd/system/execution.service
```

Pegue la siguiente configuración en el archivo.

```cáscara
[Unidad]
Descripción=Servicio de cliente de capa de ejecución de Erigon para Mainnet
Quiere = red-en línea.objetivo
Después=red-en línea.objetivo
Documentación = https://www.coincashew.com

[Servicio]
Tipo=simple
Usuario=ejecución
Grupo=ejecución
Reinicio=on-falla
ReinicioSegudo=3
KillSignal=SIGINT
TimeoutStopSec=900
ExecStart=/usr/local/bin/erigon \
   --datadir /var/lib/erigon \
   --chain mainnet \
   --port 30303 \
   --torrent.port 42069 \
   --maxpeers 50 \
   --private.api.addr 127.0.0.1:9099 \
   --authrpc.port 8551 \
   --metrics \
   --pprof \
   --prune htc \
   --prune.r.before=11052984 \
   --authrpc.jwtsecret=/secrets/jwtsecret

[Instalar]
WantedBy=multiusuario.objetivo
```

Para salir y guardar, presione `Ctrl` + `X`, luego `Y`, luego `Enter`.

Ejecute lo siguiente para habilitar el inicio automático en el momento del arranque.

```golpecito
sudo systemctl daemon-reload
sudo systemctl habilitar la ejecución
```

Finalmente, inicie su cliente de capa de ejecución y verifique su estado.

```golpecito
sudo systemctl inicia la ejecución
ejecución del estado de sudo systemctl
```

Presione `Ctrl` + `C` para salir del estado.

### 4. Comandos útiles del cliente de ejecución

{% tabs %}
{% tab title="View Logs" %}
```bash
sudo journalctl -fu execution | ccze
```

Un cliente de ejecución **Erigon** que funcione correctamente indicará "Manejando nueva carga útil". Por ejemplo,

```
erigon[3]: [INFO] [09-29|03:36:24.689] [NewPayload] Manejo de nueva carga útil       height=19999 hash=0xea060...2846a907ceb4
erigon[3]: [INFO] [09-29|03:36:25.278] [updateForkchoice] Actualización de elección de bifurcación: vaciado del estado en memoria (creado por newPayload anterior)
erigon[3]: [INFO] [09-29|03:36:25.280] RPC Daemon notificado de nuevos encabezados       from=19998 to=19999 hash=0xeeed..710b597 header sending=13.32µs log sending=290ns
erigon[3]: [INFO] [09-29|03:36:25.280] ¿cabeza actualizada                            hash=0xea06098ad5e...5e5f43 number=20000
```
{% endtab %}

{% tab title="Stop" %}
```bash
sudo systemctl stop execution
```
{% endtab %}

{% tab title="Start" %}
```bash
sudo systemctl start execution
```
{% endtab %}

{% tab title="View Status" %}
```bash
sudo systemctl status execution
```
{% endtab %}

{% tab title="Reset Database" %}
Las razones comunes para restablecer la base de datos pueden incluir:

* Recuperación de una base de datos dañada debido a un corte de energía o falla de hardware
* Resincronización para reducir el uso de espacio en disco
* Actualización a un nuevo formato de almacenamiento

```bash
sudo systemctl stop execution
sudo rm -rf /var/lib/erigon/*
sudo systemctl restart execution
```

El tiempo para volver a sincronizar el cliente de ejecución puede tardar desde algunas horas hasta un día.
{% endtab %}
{% endtabs %}

Ahora que su cliente de ejecución está configurado e iniciado, continúe con el siguiente paso para configurar su cliente de consenso.

{% hint style="warning" %}
Si está revisando los registros y ve alguna advertencia o error, tenga paciencia, ya que normalmente se resolverán una vez que tanto su cliente de ejecución como el de consenso estén completamente sincronizados con la red Ethereum.
{% endhint %}
