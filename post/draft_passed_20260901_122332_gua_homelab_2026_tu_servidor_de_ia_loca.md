# Guía Homelab 2026: Tu Servidor de IA Local y Domótica sin Suscripciones

La factura mensual de servicios de IA y domótica en la nube se ha vuelto insostenible para muchos, y la privacidad de nuestros datos, una preocupación creciente. Afortunadamente, la evolución del hardware en 2026 nos ofrece una salida robusta y eficiente. Con un Mini PC moderno y un `docker compose up -d`, podemos desplegar un servidor local capaz de gestionar nuestro hogar y ejecutar modelos de lenguaje (LLMs) sin depender de terceros ni pagar suscripciones.

## La Los procesadores lanzados en 2026 y 2025, y ahora estandarizados en 2026, han democratizado la inteligencia artificial local. La clave está en la Unidad de Procesamiento Neuronal (NPU) integrada directamente en la CPU. Chips como los Intel Core Ultra (ej. 125H, 155H, 185H de "Meteor Lake" y "Lunar Lake") o los AMD Ryzen serie 8000 y 9000 (con arquitecturas XDNA y XDNA 2) no son solo más potentes, sino que están diseñados específicamente para acelerar cargas de trabajo de IA con una eficiencia energética asombrosa.

Mientras que una GPU dedicada sigue siendo el rey para el entrenamiento de modelos masivos, para la *inferencia* de LLMs de tamaño medio (7B, 13B, incluso 30B parámetros) y la ejecución de tareas de visión por computador, la NPU es un cambio de juego. Permite ejecutar modelos cuantizados (ej. GGUF) a velocidades utilizables con un consumo de energía que antes era impensable. Un Mini PC con un Core Ultra 5 o Ryzen 7 puede ejecutar un modelo Llama 3 8B en local a varias decenas de tokens por segundo, consumiendo apenas 15-25W en carga máxima. Esto abre la puerta a asistentes de voz locales, resúmenes de texto, y automatizaciones inteligentes sin enviar un solo byte a la nube.

Para presupuestos más ajustados, incluso los procesadores Intel N100 o N305, aunque carecen de una NPU dedicada de alto rendimiento, ofrecen una mejora significativa en la eficiencia y capacidad de cómputo respecto a generaciones anteriores, siendo perfectamente válidos para Home Assistant y modelos LLM muy pequeños o para tareas de inferencia menos exigentes.

## Comparativa de Eficiencia: Mini PC Moderno vs. Torre Vieja

Reutilizar un viejo PC de sobremesa como servidor 24/7 es una tentación común, pero es un error costoso a largo plazo. La eficiencia energética de los Mini PCs modernos es incomparable.

| Característica      | Mini PC Moderno (2026)                               | Torre PC Antigua (2018)                                  |
| :------------------ | :--------------------------------------------------- | :------------------------------------------------------- |
| **CPU/NPU**         | Intel Core Ultra 5/7, AMD Ryzen 8000/9000 (15-25W)   | Intel Core i5/i7 (80-120W), AMD Ryzen 2000/3000 (65-105W) |
| **Consumo Típico**  | 10-25W (ralentí a carga LLM)                         | 60-150W (ralentí a carga ligera)                         |
| **Ruido**           | Prácticamente inaudible (ventiladores pequeños/pasivo) | Ventiladores de CPU/GPU/PSU ruidosos                     |
| **Tamaño/Formato**  | Compacto (0.5-2 litros), fácil de ocultar            | Voluminoso (15-30 litros), ocupa espacio                 |
| **Coste Eléctrico** | ~20-50€/año (24/7)                                   | ~120-300€/año (24/7)                                     |
| **Capacidad IA**    | NPU dedicada, inferencia LLM eficiente              | Depende de GPU dedicada, menos eficiente para LLMs locales |

La diferencia en el consumo eléctrico es el factor más crítico. Un Mini PC que consume 20W de media durante todo el año te costará unos 40€ anuales en electricidad (calculando 0.25€/kWh). Una torre antigua que consume 100W de media te costará 200€ anuales. La inversión inicial en un Mini PC de 250-400€ se amortiza rápidamente solo con el ahorro energético, sin contar el menor ruido, el tamaño reducido y la capacidad de IA integrada.

## Errores Fatales al Comprar Hardware para Homelab

La elección del hardware es crucial y un par de decisiones equivocadas pueden arruinar la experiencia o la longevidad de tu servidor.

### 1. Cuellos de Botella en la RAM No Ampliable

Muchos Mini PCs ultracompactos, especialmente los de gama baja, vienen con la RAM soldada a la placa base. Esto es un *error fatal* para un servidor de Homelab.

*   **Necesidad de RAM:** Home Assistant, Docker, y especialmente los LLMs, son voraces en cuanto a memoria. 16GB de RAM es un mínimo absoluto para un servidor con Home Assistant y un LLM pequeño. Para modelos de 13B o 30B, 32GB o incluso 64GB son recomendables.
*   **Ampliabilidad:** Si compras un Mini PC con 8GB o 16GB soldados, te quedarás sin margen de mejora. En cuanto intentes cargar un LLM más grande o añadir más servicios Docker, el rendimiento se desplomará debido al *swapping* constante al disco.
*   **Recomendación:** Busca Mini PCs con ranuras SO-DIMM (DDR5 es el estándar actual) que permitan la ampliación. Idealmente, compra uno con 16GB y la opción de añadir otro módulo para llegar a 32GB si es necesario.

### 2. Almacenamiento NVMe de Bajo Coste en Servidores de Escritura Continua

Los SSD NVMe son rápidos y asequibles, pero no todos son iguales, especialmente para un servidor 24/7 con bases de datos (como la de Home Assistant) y logs que generan escrituras constantes.

*   **Tipos de NAND:**
    *   **QLC (Quad-Level Cell):** La más barata, pero la menos duradera y con peor rendimiento sostenido. Cada celda almacena 4 bits, lo que reduce su vida útil (TBW - Terabytes Written).
    *   **TLC (Triple-Level Cell):** Un buen equilibrio entre coste y durabilidad. Cada celda almacena 3 bits.
    *   **MLC (Multi-Level Cell):** Más cara, pero más duradera y rápida. Cada celda almacena 2 bits.
*   **Caché DRAM:** Los SSD de calidad incluyen una caché DRAM dedicada que mejora drásticamente el rendimiento en escrituras aleatorias y la durabilidad. Los NVMe "DRAM-less" son más baratos pero sufren de rendimiento inconsistente y mayor desgaste en cargas de trabajo intensivas.
*   **TBW (Terabytes Written):** Es la métrica clave de durabilidad. Un SSD de 1TB QLC de bajo coste puede tener un TBW de 200-400TB. Un TLC de calidad puede superar los 600-800TB. Para un servidor, busca un NVMe TLC con caché DRAM y un TBW decente.
*   **Recomendación:** Invierte en un NVMe de marca reconocida (Samsung 970 EVO Plus/980 Pro, Crucial P5 Plus, Western Digital Black SN770/SN850X) con caché DRAM. Si el presupuesto es muy ajustado, al menos asegúrate de que sea TLC y no QLC. Considera un NVMe más pequeño (250-500GB) para el sistema operativo y las bases de datos críticas, y si necesitas mucho almacenamiento, un HDD externo USB 3.2 o un NAS aparte para datos menos sensibles.

## Guía de Despliegue Paso a Paso: Home Assistant y Ollama en tu Mini PC

Vamos a montar un sistema robusto y modular usando Docker Compose.

### 1. Preparación del Hardware y Sistema Operativo

1.  **Mini PC:** Adquiere un Mini PC con al menos 16GB de RAM (ampliable a 32GB), un NVMe TLC con DRAM cache, y un procesador con NPU (Intel Core Ultra o AMD Ryzen 8000/9000) si tu presupuesto lo permite, o un N100/N305 para empezar.
2.  **Instalación del SO:** Instala una distribución Linux ligera y estable. Debian 12 "Bookworm" (versión Server, sin entorno gráfico) es una excelente opción.
    *   Descarga la ISO, crea un USB booteable.
    *   Instala Debian, configurando una IP estática para tu servidor.
    *   Actualiza el sistema:
        ```bash
        sudo apt update && sudo apt upgrade -y
        ```
    *   Instala herramientas básicas:
        ```bash
        sudo apt install -y curl wget git htop neofetch
        ```

### 2. Instalación de Docker y Docker Compose

Docker es la base de nuestro despliegue, permitiéndonos ejecutar Home Assistant y Ollama en contenedores aislados.

1.  **Instalar Docker Engine:**
    ```bash
    sudo apt update
    sudo apt install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    ```
2.  **Añadir tu usuario al grupo docker:** Esto te permite ejecutar comandos docker sin `sudo`.
    ```bash
    sudo usermod -aG docker $USER
    newgrp docker # O cierra y vuelve a abrir la sesión SSH
    ```
3.  **Verificar instalación:**
    ```bash
    docker run hello-world
    ```

### 3. Configuración de Home Assistant y Ollama con Docker Compose

Crearemos un archivo `docker-compose.yml` para gestionar ambos servicios.

1.  **Crear directorios:**
    ```bash
    mkdir -p ~/homelab/homeassistant/config
    mkdir -p ~/homelab/ollama/models
    ```
2.  **Crear `docker-compose.yml`:**
    ```bash
    nano ~/homelab/docker-compose.yml
    ```
    Pega el siguiente contenido (asegúrate de ajustar la zona horaria y el dispositivo NPU si es necesario):

    ```yaml
    version: '3.8'

    services:
      homeassistant:
        container_name: homeassistant
        image: homeassistant/home-assistant:stable
        volumes:
          - ./homeassistant/config:/config
          - /etc/localtime:/etc/localtime:ro
        restart: unless-stopped
        privileged: true # Necesario para algunas integraciones de hardware USB
        network_mode: host # Permite a HA descubrir dispositivos en la red local
        environment:
          - TZ=Europe/Madrid # Ajusta tu zona horaria

      ollama:
        container_name: ollama
        image: ollama/ollama:latest
        volumes:
          - ./ollama/models:/root/.ollama # Almacena los modelos aquí
        restart: unless-stopped
        ports:
          - "11434:11434" # Puerto por defecto de Ollama
        # Si tu Mini PC tiene NPU, puedes intentar mapear el dispositivo:
        # devices:
        #   - /dev/dri:/dev/dri # Para Intel iGPU/NPU
        #   - /dev/kfd:/dev/kfd # Para AMD iGPU/NPU
        #   - /dev/accel/accel0:/dev/accel/accel0 # Ejemplo para NPU específica
        # environment:
        #   - OLLAMA_HOST=0.0.0.0:11434 # Opcional, por defecto ya escucha en 0.0.0.0
        #   - OLLAMA_NUM_GPU=1 # Opcional, para especificar el número de GPUs/NPUs
    ```
    *   **Nota sobre `devices` para Ollama:** La forma de exponer la NPU al contenedor Docker puede variar. Para Intel Core Ultra, a menudo es suficiente con `/dev/dri` (para la iGPU que la NPU utiliza). Para AMD Ryzen con XDNA, puede ser `/dev/kfd` o dispositivos específicos bajo `/dev/accel`. Consulta la documentación de Ollama y de tu distribución Linux para la configuración exacta de tu NPU. Si no estás seguro, omite la sección `devices` inicialmente; Ollama funcionará en CPU, aunque más lento.

3.  **Iniciar los servicios:**
    ```bash
    cd ~/homelab
    docker compose up -d
    ```
    Esto descargará las imágenes y arrancará los contenedores.

### 4. Configuración Inicial de Home Assistant

1.  Accede a Home Assistant desde tu navegador: `http://<IP_DE_TU_MINIPC>:8123`
2.  Sigue el asistente de configuración para crear tu usuario y contraseña.
3.  Home Assistant debería detectar automáticamente algunos dispositivos en tu red.

### 5. Descarga y Uso de Modelos LLM con Ollama

1.  **Descargar un modelo:** Desde tu Mini PC (o vía SSH), usa el cliente de Ollama para descargar un modelo. Llama 3 8B es un buen punto de partida.
    ```bash
    docker exec -it ollama ollama run llama3
    ```
    Esto descargará el modelo y te permitirá interactuar con él directamente en la terminal. Escribe `bye` para salir.
2.  **Probar la API de Ollama:** Puedes probar que Ollama está sirviendo el modelo con `curl` desde tu Mini PC o cualquier máquina en la misma red:
    ```bash
    curl http://localhost:11434/api/generate -d '{
      "model": "llama3",
      "prompt": "¿Cuál es la capital de Francia?"
    }'
    ```

### 6. Integración de Ollama con Home Assistant para IA Local

Home Assistant tiene una integración nativa para Ollama, lo que te permite usar tu LLM local para automatizaciones, respuestas de voz, etc.

1.  **En Home Assistant:** Ve a `Ajustes` -> `Dispositivos y Servicios` -> `Añadir Integración`.
2.  Busca "Ollama".
3.  Introduce la URL de tu servidor Ollama: `http://<IP_DE_TU_MINIPC>:11434`.
4.  Una vez configurado, puedes usar el servicio `llm.generate` en tus automatizaciones o scripts. Por ejemplo, para un asistente de voz local, puedes configurar un micrófono USB en tu Mini PC y usar la integración "Assist" de Home Assistant, dirigiéndola a tu modelo Ollama.

    ```yaml
    # Ejemplo de automatización en Home Assistant (automations.yaml)
    - alias: 'Respuesta de IA a comando de voz'
      trigger:
        platform: event
        event_type: assist_pipeline_result
        event_data:
          pipeline_id: your_local_voice_pipeline # Asegúrate de tener una pipeline local configurada
          intent_type: conversation
      condition:
        - condition: template
          value_template: "{{ trigger.event.data.intent_output.response.response_type == 'query' }}"
      action:
        - service: llm.generate
          data:
            model: ollama_llama3 # Nombre de tu modelo Ollama configurado en HA
            prompt: "{{ trigger.event.data.intent_output.response.speech.plain.speech }}"
          response_variable: llm_response
        - service: tts.speak
          data:
            media_player_entity_id: media_player.your_speaker # Tu altavoz local
            message: "{{ llm_response.text }}"
    ```
    *Este es un ejemplo simplificado. La configuración de un asistente de voz local completo en Home Assistant implica configurar una "Assist Pipeline" con un motor de reconocimiento de voz (ej. Whisper local) y un motor de texto a voz (ej. Piper local), todo ello ejecutándose en tu Mini PC.*

## Hoja de Ruta para tu Despliegue

Has sentado las bases de un Homelab potente y privado. Aquí tienes algunos pasos siguientes y consideraciones:

1.  **Seguridad:**
    *   Cambia las contraseñas por defecto.
    *   Configura un firewall (`ufw`) en tu Mini PC para permitir solo los puertos necesarios (SSH, 8123 para HA, 11434 para Ollama).
    *   Considera una VPN (ej. WireGuard) para acceder a tu Homelab de forma segura desde fuera de casa, en lugar de abrir puertos directamente al router.
2.  **Copias de Seguridad:** Configura copias de seguridad automáticas del directorio `~/homelab/homeassistant/config` y `~/homelab/ollama/models` a un almacenamiento externo o a la nube (cifrado).
3.  **Más Servicios:** Una vez que te sientas cómodo, puedes añadir más servicios Docker:
    *   **AdGuard Home:** Bloqueador de anuncios y rastreadores a nivel de red.
    *   **Vaultwarden:** Gestor de contraseñas autoalojado.
    *   **Nextcloud:** Nube personal para archivos y colaboración.
    *   **n8n o Node-RED:** Herramientas de automatización visual para flujos de trabajo complejos.
    *   **Plex/Jellyfin:** Servidor multimedia.
4.  **Monitorización:** Instala herramientas como `Portainer` (interfaz gráfica para Docker) o `Grafana` con `Prometheus` para monitorizar el rendimiento y el estado de tus servicios.
5.  **Optimización de NPU:** Experimenta con las configuraciones de `devices` en Ollama y busca versiones de modelos LLM optimizadas para NPUs (a menudo etiquetadas como "quantized" o "GGUF" con capas específicas para aceleración de hardware).

Este enfoque te proporciona un control total sobre tu infraestructura digital, eliminando dependencias de terceros y protegiendo tu privacidad, todo ello con una eficiencia energética que hace que la inversión inicial sea rápidamente rentable.

---

> *Disclosure: This article was researched and structured with the assistance of advanced AI language models, followed by automated technical validation.*
