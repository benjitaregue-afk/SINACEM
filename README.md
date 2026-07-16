<p align="center">
  <img src="https://img.shields.io/badge/Proyecto%20estudiantil-Colegio%20Domingo%20Savio-1E88E5?style=for-the-badge" alt="Proyecto estudiantil del Colegio Domingo Savio">
  <img src="https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.10 o superior">
  <img src="https://img.shields.io/badge/Hardware-Arduino-00878F?style=for-the-badge&logo=arduino&logoColor=white" alt="Arduino">
</p>

# SINACEM

SINACEM es un proyecto educativo de interacción humano-computadora desarrollado por estudiantes del **Colegio Domingo Savio**. El sistema combina comandos de voz, visión en primera persona, inteligencia artificial y estimulación muscular eléctrica (EMS) para generar y ejecutar acciones controladas mediante relés.

El objetivo del proyecto es explorar, con fines académicos, cómo la inteligencia artificial puede interpretar el entorno y convertir instrucciones humanas en una secuencia de acciones físicas sobre un prototipo experimental.

> [!WARNING]
> SINACEM es un prototipo educativo y experimental. No es un dispositivo médico ni debe utilizarse sin supervisión responsable.

## Contenido del repositorio

- `app.py`: ciclo principal del sistema: cámara, voz, modelo de IA y envío de comandos.
- `utils/receiver.py`: servidor Flask que recibe y ejecuta secuencias temporizadas de relés y EMS.
- `manual_control_app.py`: interfaz gráfica para calibración y pruebas manuales.
- `firmware/`: firmware del controlador Arduino compatible.
- `run_hardware.sh`: inicio automatizado del sistema en modo de hardware con relés.
- `design/`: archivos de diseño e impresión 3D del prototipo.

## ¿Cómo funciona?

1. El sistema espera una instrucción de voz.
2. La cámara captura una imagen del entorno.
3. El modelo de inteligencia artificial analiza la instrucción y la imagen.
4. La respuesta se convierte en una secuencia de acciones con tiempos definidos.
5. El servidor local envía las órdenes al Arduino y al sistema de relés.
6. Los relés seleccionan la salida correspondiente para ejecutar la acción configurada.

## Seguridad y responsabilidad

Este repositorio controla hardware de estimulación muscular eléctrica. Su uso incorrecto puede causar lesiones.

- Utilizar únicamente con supervisión de una persona responsable y capacitada.
- Obtener consentimiento informado antes de cualquier prueba con una persona.
- Empezar siempre con intensidades conservadoras durante la calibración.
- Mantener accesible un mecanismo de parada de emergencia y una desconexión física.
- No colocar electrodos en la cabeza, el cuello, el pecho ni otras zonas sensibles.
- No utilizar el sistema en personas con marcapasos, implantes electrónicos o contraindicaciones médicas para EMS.
- Detener inmediatamente la prueba ante dolor, mareo, entumecimiento o una reacción inesperada.

## Requisitos

### Software

- Python 3.10 o superior.
- macOS o Linux.
- Arduino IDE.
- Una clave de Anthropic configurada como `ANTHROPIC_API_KEY`.

### Hardware

- Una placa compatible con Arduino.
- Un módulo de relés.
- Un dispositivo de estimulación EMS compatible.
- Una cámara.
- Electrodos y conexiones apropiadas para el prototipo.

## Instalación rápida

### 1. Clonar el repositorio

```bash
git clone https://github.com/benjitaregue-afk/SINACEM.git
cd SINACEM
```

### 2. Crear el entorno virtual e instalar dependencias

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install pyserial requests
```

### 3. Configurar la clave de la IA

```bash
cp .env_empty .env
```

Después, editar `.env` y agregar:

```env
ANTHROPIC_API_KEY=tu_clave_aqui
```

> Nunca publiques tu clave real en GitHub.

### 4. Cargar el firmware

1. Abrir en Arduino IDE el archivo `.ino` ubicado dentro de `firmware/`.
2. Seleccionar la placa y el puerto correctos.
3. Cargar el programa con una velocidad serial de `115200` baudios.

### 5. Ejecutar el sistema

```bash
./run_hardware.sh
```

Para indicar manualmente el puerto del relé:

```bash
RELAY_PORT=/dev/cu.usbserial-210 ./run_hardware.sh
```

El script realiza las siguientes tareas:

- Inicia `utils/receiver.py` en modo relé.
- Espera la respuesta del endpoint `/health`.
- Verifica que el hardware esté conectado.
- Inicia `app.py` con la dirección del receptor configurada.
- Cierra los procesos auxiliares al finalizar.

## Modos de ejecución

### Modo A: solo relés

Este es el modo recomendado para verificar el cableado y el funcionamiento de las salidas:

```bash
HARDWARE_MODE=relay RELAY_PORT=/dev/cu.usbserial-210 python utils/receiver.py
```

En otra terminal se puede comprobar el estado:

```bash
curl -sS http://127.0.0.1:5001/health
```

La respuesta debería incluir:

```json
{
  "hardware_mode": "relay",
  "relay_hardware_connected": true
}
```

### Modo B: relés y estimulación

Este modo habilita explícitamente la ruta completa del estimulador:

```bash
HARDWARE_MODE=full \
STIM_PORT=/dev/cu.usbserial-XXX \
RELAY_PORT=/dev/cu.usbserial-YYY \
python utils/receiver.py
```

La estimulación permanece deshabilitada si no se selecciona explícitamente el modo `full`. Si un dispositivo aparece como `SIMULATED`, significa que no fue posible abrir su puerto serial.

## Aplicación principal

Si no se utiliza `run_hardware.sh`, la aplicación puede iniciarse manualmente:

```bash
python app.py
```

Flujo de ejecución:

1. Esperar el comando de voz.
2. Capturar la imagen más reciente de la cámara.
3. Enviar la instrucción y la imagen al modelo de IA.
4. Convertir el plan generado en comandos temporizados para relés y EMS.
5. Enviar la secuencia a `utils/receiver.py` mediante una solicitud `POST`.

## Calibración manual

Para abrir la interfaz de control manual:

```bash
python manual_control_app.py
```

La interfaz permite:

- Conectar manualmente los puertos seriales.
- Seleccionar la salida correspondiente a cada dedo o muñeca.
- Ajustar amplitud, frecuencia, ancho de pulso y duración.
- Verificar cada respuesta antes de utilizar el flujo automatizado.

## Salidas disponibles

El receptor y el firmware aceptan los siguientes objetivos:

- `wrist_left`: muñeca izquierda.
- `wrist_right`: muñeca derecha.
- `thumb`: pulgar.
- `index`: índice.
- `middle`: dedo medio.
- `ring`: anular.
- `pinky`: meñique.
- `x`: apagar o aislar todas las salidas.

Mapeo de pines configurado en el firmware:

| Pin | Salida |
| --- | --- |
| D2 | `wrist_left` |
| D4 | `wrist_right` |
| D3 | `thumb` |
| D5 | `index` |
| D6 | `middle` |
| D7 | `ring` |
| D8 | `pinky` |

Comandos de diagnóstico disponibles por puerto serial:

- `test`: prueba todas las salidas en secuencia.
- `mode_low`: configura relés activos en nivel bajo.
- `mode_high`: configura relés activos en nivel alto.
- `x`: apaga todas las salidas.

## API local

### `GET /health`

Devuelve el estado del servidor y de las conexiones de hardware.

### `POST /execute`

Recibe un objeto JSON con comandos organizados por tiempo. Ejemplo:

```json
{
  "0.0": [
    {
      "type": "RELAY",
      "finger": "index"
    },
    {
      "type": "EMS",
      "channel": 1,
      "amplitude": 60,
      "duration": 1.0,
      "frequency": 100,
      "pulse_width": 1000
    }
  ],
  "1.5": [
    {
      "type": "RELAY",
      "finger": "x"
    }
  ]
}
```

## Solución de problemas

### El relé aparece como simulado

- Cerrar el monitor serial de Arduino y cualquier programa que esté utilizando el puerto.
- Verificar el valor de `RELAY_PORT`.
- Confirmar que la placa tenga alimentación y que el cable de datos funcione.
- Volver a iniciar el receptor en modo relé.

### El servidor inicia, pero no se escucha el relé

- Ejecutar el comando serial `test`.
- Probar `mode_low` y `mode_high`.
- Verificar la alimentación, la tierra común y las conexiones de entrada.
- Comparar el cableado con el mapeo de pines del firmware.

### La cámara no se abre

- Revisar los permisos de cámara del sistema operativo.
- Cerrar otras aplicaciones que puedan estar utilizando la cámara.

### Error de clave o conexión con la IA

- Verificar que `.env` exista y contenga `ANTHROPIC_API_KEY`.
- Confirmar que la clave sea válida y que exista conexión a internet.

## Arquitectura del sistema

```text
Comando de voz + imagen de cámara
              |
              v
           app.py
              |
              | genera un plan de acciones con IA
              v
     utils/receiver.py (Flask)
              |
              | envía comandos seriales
              v
      Arduino + módulo de relés
              |
              v
       Prototipo experimental
```

## Equipo

Proyecto desarrollado por estudiantes del **Colegio Domingo Savio** con fines educativos y de investigación escolar.

## Créditos y referencias

SINACEM adapta una metodología y una base técnica preexistentes al contexto educativo del Colegio Domingo Savio. El proyecto de referencia y sus autores originales conservan el crédito por su trabajo; SINACEM no afirma haber participado en sus premios, demostraciones ni actividades institucionales.

Referencias académicas relacionadas:

- [Full-Hand Electro-Tactile Feedback without Obstructing the Palmar Side of the Hand](https://github.com/humancomputerintegration/BOH-Electro-Tactile)
- [Generative Muscle Stimulation: Providing Users with Physical Assistance by Constraining Multimodal AI with Embodied Knowledge](https://arxiv.org/pdf/2505.10648)
- [Increasing Electrical Muscle Stimulation's Dexterity by Means of Back-of-Hand Actuation](https://lab.plopes.org/published/2021-CHI-BackHandEMS.pdf)

---

<p align="center">
  Hecho con fines educativos por estudiantes del <strong>Colegio Domingo Savio</strong>.
</p>
