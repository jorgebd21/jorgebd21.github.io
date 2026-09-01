# Mini PCs con NPU en 2026: Tu Homelab de IA Local y Domótica Sin Suscripciones

Un clúster de APUs modernas consume entre 15W y 35W en reposo, mientras que la factura acumulada de suscripciones como OpenAI Plus, Claude Pro y almacenamiento en la nube para domótica supera fácilmente los 600 euros anuales por usuario. En 2026, la madurez de los modelos de lenguaje pequeños (SLMs) como Qwen 2.5, Llama 3.2 y Phi-3, sumada a la llegada de procesadores x86 con unidades de procesamiento neuronal (NPU) dedicadas, ha cambiado drásticamente la ecuación financiera de la infraestructura doméstica.

Montar un servidor Homelab ya no requiere reciclar una torre de escritorio ruidosa con una GPU dedicada consumiendo 250W. Los Mini PCs de última generación permiten ejecutar agentes de procesamiento de lenguaje natural, transcripción de voz en tiempo real y automatización del hogar 100% privados, locales y con latencias de respuesta en red local por debajo de los 200 milisegundos.

---

## Arquitectura de Hardware en 2026: NPUs y la Muerte de la GPU Dedicada en Homelabs

Durante años, ejecutar inferencia de IA en casa requería GPUs Nvidia de la serie RTX debido a la hegemonía de CUDA. Sin embargo, en la arquitectura de silicio de 2026, las NPUs integradas (como AMD XDNA 2 e Intel AI Boost) junto con las iGPUs de alto ancho de memoria (Radeon 780M/890M e Intel Arc) asumen el trabajo de inferencia continua para modelos de hasta 14 mil millones de parámetros (14B) cuantizados.

La clave no es solo la potencia bruta en TOPS (Tera Operations Per Second), sino la eficiencia por vatio y el soporte en frameworks como ONNX Runtime, OpenVINO y vLLM/Ollama. Mientras que la iGPU destaca en el procesamiento matricial masivo en paralelo (generación de tokens en LLMs), la NPU es idónea para tareas de fondo continuas y de bajo consumo, como la detección de presencia por visión computacional o la escucha activa de palabras de activación (*wake words*).

| Procesador | Arquitectura NPU / iGPU | Consumo (TDP) | Caso de Uso Ideal |
|---|---|---|---|
| AMD Ryzen 7 8840HS | XDNA (16 TOPS) + Radeon 780M | 15W - 30W | Inferencia balanceada LLMs 7B-14B |
| Intel Core Ultra 5 125H | AI Boost (11 TOPS) + Arc Graphics | 20W - 28W | Pipeline OpenVINO y visión local |
| AMD Ryzen 9 9955HS | XDNA 2 (50 TOPS) + Radeon 890M | 28W - 54W | Agentes multitarea y RAG denso |
| Intel N100 / N305 | Sin NPU (Solo Intel UHD) | 6W - 15W | Domótica básica (sin LLMs locales) |

Los chips de clase ultra-económica como el Intel N100 siguen siendo válidos para servicios de red ligeros (Pi-hole, Reverse Proxies, Home Assistant básico), pero quedan obsoletos en el momento en que se introduce inferencia de voz o lenguaje local. Para un Homelab orientado a IA en 2026, el punto dulce se sitúa en las plataformas AMD Ryzen 8000/9000 o Intel Core Ultra de primera y segunda generación.

---

## Dimensionamiento Técnico: El Peligro de Errar en Memoria y Térmicas

El error más común al seleccionar un Mini PC para IA local es priorizar la frecuencia del procesador por encima de la arquitectura del subsistema de memoria y la disipación térmica del chasis.


+------------------------------------------------------------------+
|                      MINI PC CORE ARCHITECTURE                   |
|                                                                  |
|  +--------------------+      +--------------------------------+  |
|  | CPU Core Complex   |      | Dual-Channel DDR5 (5600+ MT/s) |  |
|  +---------+----------+      +---------------+----------------+  |
|            |                                 |                   |
|            v                                 v                   |
|  +--------------------+             +-----------------+          |
|  | Shared Memory Bus  |<----------->| System VRAM Pool|          |
|  +---------+----------+             +--------+--------+          |
|            |                                 |                   |
|            v                                 v                   |
|  +--------------------+             +-----------------+          |
|  | NPU / iGPU Accelerator             Unified RAM Allocation     |
|  +--------------------+             +-----------------+          |
+------------------------------------------------------------------+


### 1. El cuello de botella insalvable del ancho de banda de memoria
En la inferencia de LLMs, la velocidad de generación de tokens no depende de los TFLOPS del procesador, sino del ancho de banda de la memoria RAM (GB/s). Cuando un modelo de lenguaje está cargado en la VRAM compartida de la iGPU/NPU, cada token generado exige leer la totalidad de los pesos del modelo desde la memoria principal.

*   **Configuración en Monocanal (Single-Channel):** Reduce el ancho de banda a la mitad (~38.4 GB/s en DDR5). La velocidad de generación en un modelo de 7B (Q4_K_M) cae a unos infumables 3 a 5 tokens por segundo.
*   **Configuración en Doble Canal (Dual-Channel DDR5-5600):** Ofrece hasta ~89.6 GB/s, elevando la velocidad de inferencia a 18-25 tokens por segundo en el mismo modelo.
*   **Memorias LPDDR5x Soldadas (7500+ MT/s):** Alcanzan anchos de banda superiores a los 120 GB/s. Si compras un Mini PC con memoria soldada, asegúrate de pedir 32 GB o 64 GB desde el primer día, ya que no podrás ampliarla.

### 2. Térmicas y Throttling en chasis de 0.5 Litros
La ejecución continua de un agente de IA procesando documentos o transcribiendo audio mantendrá la APU al 100% de carga durante minutos. En chasis extremadamente compactos:

*   El procesador alcanza rápidamente el límite térmico (Tjunction ~95°C) y aplica *thermal throttling*, reduciendo las frecuencias de reloj hasta un 40%.
*   Busca Mini PCs que utilicen disipación por cámara de vapor o ventiladores de gran diámetro con perfiles BIOS configurables en modo "Performance" a 35W o 45W sostenidos.

### 3. Degradación del almacenamiento NVMe por IOPS en Bases de Datos Vectoriales
El uso de bases de datos vectoriales (Qdrant, ChromaDB) para sistemas RAG (Retrieval-Augmented Generation) genera escrituras aleatorias constantes de embeddings. Los SSDs de gama baja sin DRAM Cache sufren caídas drásticas de rendimiento bajo estas cargas de trabajo y reducen exponencialmente su vida útil (TBW). Elige unidades NVMe PCIe 4.0 con DRAM integrada y añade disipadores térmicos de cobre pasivos.

---

## Despliegue del Stack Local: Proxmox, Ollama, Open WebUI y Home Assistant

La arquitectura recomendada consiste en utilizar **Proxmox VE 8** como hipervisor base. Esto permite aislar los servicios de domótica crítica en una máquina virtual dedicada, mientras que los servicios de IA se ejecutan en un contenedor LXC o una VM con *passthrough* directo de los nodos de renderizado de la iGPU/NPU (`/dev/dri/renderD128` y accesos a controladores de NPU).

A continuación se detalla una pila de producción utilizando **Docker Compose v2** sobre Debian 12 / Ubuntu Server 24.04 LTS dentro del entorno virtualizado:

yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama_engine
    restart: unless-stopped
    devices:
      - /dev/dri:/dev/dri  # Passthrough de iGPU/NPU para aceleración hardware
    volumes:
      - ./ollama_data:/root/.ollama
    ports:
      - "11434:11434"
    environment:
      - OLLAMA_KEEP_ALIVE=24h
      - OLLAMA_NUM_PARALLEL=4

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open_webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - WEBUI_SECRET_KEY=cambia_este_hash_super_seguro_2026
    volumes:
      - ./webui_data:/app/backend/data
    depends_on:
      - ollama

  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - ./ha_config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro

  whisper-asr:
    image: fedelman/faster-whisper-server:latest-cpu
    container_name: whisper_service
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      - MODEL_SIZE=medium.en
      - DEVICE=cpu
      - COMPUTETYPE=int8


### Configuración del Passthrough de Aceleración en Linux
Para verificar que el sistema operativo reconoce los nodos de aceleración gráfica y NPU dentro del contenedor o servidor host, ejecuta:

bash
## Verificar la presencia de dispositivos de aceleración gráfica e inferencia
ls -la /dev/dri

## Salida esperada:
## drwxr-xr-x  2 root root        80 Jan 15 10:00 .
## crw-rw----  1 root render 226,   0 Jan 15 10:00 card0
## crw-rw----  1 root render 226, 128 Jan 15 10:00 renderD128

## Comprobar el uso en tiempo real de la GPU/NPU AMD
sudo amdgpu_top

## Para procesadores Intel Core Ultra
intel_gpu_top


---

## Casos de Uso del Día a Día: Agentes Locales y Domótica Sin Latencia

Al eliminar las llamadas a APIs externas, el Homelab procesa flujos de información sensibles dentro de la red de área local (LAN), manteniendo privacidad absoluta y velocidad constante incluso si la conexión a Internet cae.


+-------------------------------------------------------------------+
|                     LOCAL AUTOMATION PIPELINE                     |
|                                                                   |
|  +--------------------+       +--------------------------------+  |
|  | Local Audio Input  | ----> | Faster-Whisper (STT Engine)    |  |
|  +--------------------+       +---------------+----------------+  |
|                                               |                   |
|                                               v                   |
|  +--------------------+       +--------------------------------+  |
|  | Home Assistant     | <---- | Local LLM (Intent Parsing)     |  |
|  | Real-time Exec     |       | (Qwen 2.5 7B / Llama 3.2 3B)   |  |
|  +--------------------+       +--------------------------------+  |
+-------------------------------------------------------------------+


### 1. Transcripción de Reuniones y Resumen de Documentos Confidenciales
Utilizando la combinación de `faster-whisper` procesado mediante las extensiones de la NPU y un modelo como `Qwen2.5-Coder-7B` o `Llama-3.2-3B-Instruct`:

*   **Flujo:** Graba notas de voz o reuniones de trabajo desde tu teléfono mediante una automatización con la app local. El archivo de audio entra en una carpeta monitorizada por un contenedor `n8n` ejecutable en el Mini PC.
*   **Procesamiento:** Whisper convierte el audio a texto en cuestión de segundos (aprovechando la aceleración vectorial). Seguidamente, Ollama procesa la plantilla de resumen, extracta los puntos de acción y los envía directamente a tu gestor de tareas o base de datos local (Vault de Obsidian / PostgreSQL).
*   **Ventaja:** Ningún dato empresarial o conversación privada sale de tu red local.

### 2. Domótica con Procesamiento de Intenciones por Lenguaje Natural
El procesamiento clásico de voz en domótica mediante soluciones basadas en la nube sufre latencias inherentes (1.5 a 3 segundos) y problemas de privacidad. Integrando Home Assistant con la extensión **Local LLM Conversation**:

*   Puedes emitir comandos ambiguos o complejos: *"Hace fresco en la sala y voy a empezar a ver una película"*.
*   El modelo LLM local (ejecutado en la NPU/iGPU en < 150ms) analiza la intención, determina que debe subir la temperatura del termostato del salón a 21°C, bajar las persianas al 80% y atenuar la iluminación de la TV.
*   La acción se ejecuta instantáneamente mediante protocolos locales como Zigbee (vía Zigbee2MQTT) o Matter sin tocar servidores de terceros.

---

## Recomendaciones Prácticas y Siguientes Pasos

Para garantizar el éxito en el montaje de tu Homelab de IA local durante este año, sigue esta lista de verificación técnica antes de realizar cualquier compra o despliegue:

1.  **Selección estricta de hardware:** Opta por Mini PCs equipados con procesadores **AMD Ryzen 7 8840HS / 9850HS** o **Intel Core Ultra 5/7**. Evita procesadores de generaciones anteriores a 2024 si tu objetivo principal incluye ejecutar LLMs locales de manera fluida.
2.  **Dimensionamiento de RAM:** Instala un kit de **32 GB o 64 GB de memoria RAM DDR5 a 5600 MT/s en configuración de doble canal**. Asegúrate en el BIOS de asignar al menos 8 GB o 16 GB de memoria del sistema como VRAM compartida (*UMA Frame Buffer Size*) para la iGPU/NPU.
3.  **Almacenamiento térmicamente protegido:** Utiliza SSDs NVMe M.2 PCIe 4.0 con memorias TLC (evita QLC para bases de datos vectoriales) y asegúrate de que el chasis incluya un disipador de aluminio o un pequeño ventilador dedicado a las unidades M.2.
4.  **Estrategia de modelos:** No intentes ejecutar modelos de 70B parámetros. La excelencia en Homelabs locales en 2026 se obtiene combinando modelos especializados pequeños:
    *   **Whisper Medium/Small** para transcripción de voz.
    *   **Llama 3.2 (3B)** o **Phi-3.5** para tareas de control domótico ultra-rápidas.
    *   **Qwen 2.5 (7B o 14B)** cuantizado en `Q4_K_M` para tareas complejas de RAG, programación y generación de texto.
5.  **Mantenimiento:** Configura copias de seguridad automatizadas de tus volúmenes de Docker y estados de Home Assistant utilizando herramientas como **BorgBackup** o **Kopia** hacia un almacenamiento masivo local (NAS).

---

> *Disclosure: This article was researched and structured with the assistance of advanced AI language models, followed by automated technical validation.*
