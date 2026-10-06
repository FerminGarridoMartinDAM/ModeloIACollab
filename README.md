#  ==============================================================================
# 🧬 PROYECTO SCIENTIA: SISTEMA MULTI-AGENTE (Versión Colab/VSCode)
# ==============================================================================


#              Este archivo contiene el código para las 4 CELDAS de Colab.

#   ==============================================================================
#    🛠️ PASO 0: Configuración del Entorno (Léelo antes de copiar) 
#  ==============================================================================
#  Ve a la web de Google Colab: https://colab.research.google.com/
# Haz clic en "Nuevo cuaderno".
# En el menú superior, ve a Entorno de ejecución > Cambiar tipo de entorno de ejecución.
# En "Acelerador de hardware", selecciona T4 GPU (¡Importantísimo!).
# Haz clic en Guardar.
# Ahora, crea 4 celdas de código en Colab (puedes añadir celdas pulsando + Texto o + Código arriba).
# Copia y pega el contenido de abajo celda por celda.


# INSTRUCCIONES RÁPIDAS:
# Copia el bloque correspondiente a cada celda y ejecútalo en orden.
# ==============================================================================


# ⬇️⬇️⬇️ COPIA DESDE AQUÍ EN LA PRIMERA CELDA DE COLAB ⬇️⬇️⬇️
# ==============================================================================
# [CELDA 1] MOTOR NEURONAL E INSTALACIÓN
# ==============================================================================
# Esta celda instala Ollama, arranca el servidor en segundo plano y descarga
# el modelo de IA. Tarda unos 40-60 segundos.

import os
import time
import subprocess
import threading
import requests

print("🚀 INICIANDO FASE 1: INSTALACIÓN...")

# 1. Instalar dependencias (Usamos os.system para compatibilidad con VS Code)
print("🔧 Instalando Motor Ollama...")
os.system("sudo apt-get install -y zstd")
os.system("curl -fsSL https://ollama.com/install.sh | sh")
os.system("pip install ollama")

import ollama # Importamos tras instalar

# 2. Arrancar el servidor (Truco del hilo paralelo)
print("\n🧠 Encendiendo el Cerebro Digital...")
def run_ollama():
    subprocess.run(["ollama", "serve"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

server_thread = threading.Thread(target=run_ollama)
server_thread.daemon = True
server_thread.start()

# 3. Verificar conexión
print("⏳ Esperando al servidor (aprox 20 seg)...")
servidor_listo = False
intentos = 0

while not servidor_listo and intentos < 30:
    try:
        if requests.get("http://localhost:11434").status_code == 200:
            servidor_listo = True
            print("✅ ¡CONEXIÓN ESTABLECIDA!")
    except:
        time.sleep(2)
        intentos += 1

# 4. Descargar Modelo
if servidor_listo:
    print("\n⬇️ Descargando modelo Llama 3.2...")
    os.system("ollama pull llama3.2")
    print("🟢 TODO LISTO. Pasa a la Celda 2.")
else:
    print("❌ ERROR: El servidor no arrancó. ¿Activaste la T4 GPU?")
# ⬆️⬆️⬆️ FIN DE LA CELDA 1 ⬆️⬆️⬆️



# ⬇️⬇️⬇️ COPIA DESDE AQUÍ EN LA SEGUNDA CELDA DE COLAB ⬇️⬇️⬇️
# ==============================================================================
# [CELDA 2] CONEXIÓN A MEMORIA (DRIVE)
# ==============================================================================
# Al ejecutar esto, te saldrá una ventana emergente pidiendo permiso
# para acceder a Google Drive. Acepta para guardar los datos.

import os
try:
    from google.colab import drive
    print("📂 Solicitando acceso a Google Drive...")
    drive.mount('/content/drive')
    RUTA_CARPETA = '/content/drive/MyDrive/PROYECTO_SCIENTIA'
except ImportError:
    print("⚠️ Estás en modo local (sin Drive). Se usará carpeta local.")
    RUTA_CARPETA = './PROYECTO_SCIENTIA'

if not os.path.exists(RUTA_CARPETA):
    os.makedirs(RUTA_CARPETA)

print(f"✅ Memoria configurada en: {RUTA_CARPETA}")
# ⬆️⬆️⬆️ FIN DE LA CELDA 2 ⬆️⬆️⬆️



# ⬇️⬇️⬇️ COPIA DESDE AQUÍ EN LA TERCERA CELDA DE COLAB ⬇️⬇️⬇️
# ==============================================================================
# [CELDA 3] SIMULACIÓN (EL DEBATE)
# ==============================================================================
# Esta es la celda principal. Ejecútala para ver a las IAs hablar.
# Para PARAR, pulsa el botón de Stop (⏹️) de la celda.

import json
import random

# CONFIGURACIÓN
MODELO = "llama3.2"
ARCHIVO_MEMORIA = f'{RUTA_CARPETA}/matriz_cientifica_4x4.json'
ARCHIVO_BORRADOR = f'{RUTA_CARPETA}/borrador_sesion_actual.txt'

RAMAS = {
    "COSMOS": "Astrofísica, Relatividad y Espacio-Tiempo",
    "CUANTICA": "Física de Partículas y Probabilidad",
    "BIO": "Evolución, Genética y Conciencia",
    "LOGICA": "Matemáticas Puras y Geometría"
}

ARQUETIPOS = {
    "DOGMATICO": {"actitud": "Defiende las leyes clásicas."},
    "DISRUPTOR": {"actitud": "Propone teorías arriesgadas."},
    "EMPIRISTA": {"actitud": "Solo cree en datos medibles."},
    "CONECTOR": {"actitud": "Busca unificar conceptos."}
}

# GESTIÓN DE ARCHIVOS
def cargar_sistema():
    with open(ARCHIVO_BORRADOR, 'w', encoding='utf-8') as f:
        f.write("--- INICIO DE SESIÓN ---\n")
    if os.path.exists(ARCHIVO_MEMORIA):
        with open(ARCHIVO_MEMORIA, 'r', encoding='utf-8') as f: return json.load(f)
    else:
        poblacion = []
        for rama, desc in RAMAS.items():
            for arq, info in ARQUETIPOS.items():
                poblacion.append({"id": f"{rama}-{arq}", "grupo": rama, "tema": desc, "personalidad": info['actitud'], "nivel": 1, "xp": 0})
        return {"poblacion": poblacion, "actas_congreso": []}

def guardar_sistema(datos, texto):
    with open(ARCHIVO_MEMORIA, 'w', encoding='utf-8') as f: json.dump(datos, f, indent=4)
    with open(ARCHIVO_BORRADOR, 'a', encoding='utf-8') as f: f.write(texto + "\n\n")

# BUCLE PRINCIPAL
def ejecutar_simulacion():
    sistema = cargar_sistema()
    print(f"\n🏛️ CONGRESO ACTIVO. Agentes: {len(sistema['poblacion'])}")
    
    while True:
        try:
            agente = random.choice(sistema['poblacion'])
            ultimos = sistema['actas_congreso'][-10:]
            chat_txt = "\n".join([f"{m['autor']}: {m['texto']}" for m in ultimos])
            
            print(f"🎤 {agente['id']} pensando...")
            prompt = f"Eres {agente['id']} ({agente['grupo']}). Experto en {agente['tema']}. Actitud: {agente['personalidad']}. Nivel {agente['nivel']}."
            
            resp = ollama.chat(model=MODELO, messages=[
                {'role': 'system', 'content': prompt},
                {'role': 'user', 'content': f"DIÁLOGO PREVIO:\n{chat_txt}\n\nTu turno."}
            ])

            txt = resp['message']['content']
            print(f"\n⚛️ {agente['id']} [Nvl {agente['nivel']}]:\n{txt}\n" + "-"*50)
            
            sistema['actas_congreso'].append({"autor": agente['id'], "texto": txt})
            guardar_sistema(sistema, f"[{agente['id']}]: {txt}")
            
            # Subir de nivel
            for a in sistema['poblacion']:
                if a['id'] == agente['id']:
                    a['xp'] += 1
                    if a['xp'] >= 6 * a['nivel']:
                        a['nivel'] += 1
                        print(f"🌟 ¡{a['id']} SUBE A NIVEL {a['nivel']}! 🌟")
            
            time.sleep(2)

        except KeyboardInterrupt:
            print("\n🛑 PAUSA. Ejecuta la Celda 4 para el resumen.")
            break
        except Exception as e:
            # Si hay error de conexión, intentamos reconectar automáticamente
            if "connect" in str(e).lower():
                print("⚠️ Reconectando servidor...")
                time.sleep(5)
                # En un caso real aquí iría el protocolo Lázaro completo, 
                # simplificado aquí para que quepa bien en la celda.
            else:
                print(f"❌ Error: {e}")
                time.sleep(5)

# Ejecutamos la simulación
ejecutar_simulacion()
# ⬆️⬆️⬆️ FIN DE LA CELDA 3 ⬆️⬆️⬆️



# ⬇️⬇️⬇️ COPIA DESDE AQUÍ EN LA CUARTA CELDA DE COLAB ⬇️⬇️⬇️
# ==============================================================================
# [CELDA 4] EL SECRETARIO (RESUMEN)
# ==============================================================================
# Ejecuta esta celda DESPUÉS de parar la Celda 3 para ver qué ha pasado.

def generar_resumen():
    if not os.path.exists(ARCHIVO_BORRADOR):
        print("❌ No hay datos recientes.")
        return

    print("🧠 Leyendo actas y generando informe...")
    with open(ARCHIVO_BORRADOR, 'r', encoding='utf-8') as f:
        texto = f.read()

    prompt = f"Resume los puntos clave y descubrimientos de este debate:\n\n{texto[-6000:]}"
    
    try:
        res = ollama.chat(model="llama3.2", messages=[{'role': 'user', 'content': prompt}])
        print("\n📑 RESUMEN DE LA SESIÓN:\n" + "="*30)
        print(res['message']['content'])
    except Exception as e:
        print(f"❌ Error (asegúrate que el servidor de la Celda 1 sigue activo): {e}")

generar_resumen()
# ⬆️⬆️⬆️ FIN DE LA CELDA 4 ⬆️⬆️⬆️
