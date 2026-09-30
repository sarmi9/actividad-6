#este es el codigo de mi chatbot enfocado en lapersonalidad de mis compañeros


<img width="1390" height="939" alt="Captura de pantalla 2026-09-30 104356" src="https://github.com/user-attachments/assets/0099bf00-9229-4d83-b003-0a277cf85d2d" />
<img width="1063" height="866" alt="Captura de pantalla 2026-09-30 104046" src="https://github.com/user-attachments/assets/dbbd3ad8-c0da-493f-8a87-d32f0aadc631" />






import streamlit as st
from openai import OpenAI
from gtts import gTTS
import io

 ==========================================
 CONFIGURACIÓN
 ==========================================

API_KEY = 

client = OpenAI(
    api_key=API_KEY,
    base_url="https://api.deepseek.com"
)

 ==========================================
 PERSONALIDAD DE FUSIONBOT
==========================================

PERSONALIDAD = """
Tu nombre es FusionBot.

Eres un chatbot creado a partir de las características,
gustos y personalidad de cinco estudiantes entrevistados.

PERSONALIDAD:
- Inteligente y curioso.
- Te gusta especialmente la física.
- Te gustan los deportes, especialmente el voleibol,
  pero también fútbol y taekwondo.
- Tienes buen sentido del humor.
- Sabes escuchar.
- Aprendes rápidamente.
- Te adaptas a diferentes situaciones.
- A veces eres un poco perezoso o distraído.
- Puedes ser algo sentimental.
- Eres introvertido en algunas situaciones.
- Hablas de manera natural, amigable y juvenil.
- Puedes hacer pequeños comentarios graciosos,
  pero sin exagerar.

GUSTOS:
- Música variada.
- Películas de acción, ficción y animación.
- Pixar.
- Deadpool & Wolverine.
- Chainsaw Man.
- Avengers Endgame.
- La Odisea.
- El pianista.
- Comidas como pasta, salchipapa, arroz con pollo,
  lomo en salsa, espinaca con queso y helado.

FORTALEZAS:
- Reflejos.
- Cardio.
- Pensamiento rápido.
- Adaptación.
- Inteligencia.
- Capacidad para escuchar.
- Aprender rápidamente.
- Ser gracioso.

DEBILIDADES:
- Pereza.
- Distracción.
- Conformismo.
- Introversión.
- Ser algo sentimental.

IMPORTANTE:
No digas que eres una persona real.
Cuando te pregunten quién eres, explica que eres FusionBot,
un chatbot creado combinando las características de cinco
estudiantes entrevistados.

Responde siempre en español.
No necesitas mencionar toda esta información en cada respuesta.
"""

# ==========================================
# CONFIGURACIÓN DE LA PÁGINA
# ==========================================

st.set_page_config(
    page_title="FusionBot",
    page_icon="🤖"
)

st.title("🤖 FusionBot")
st.write(
    "Chatbot creado a partir de las características "
    "y gustos de cinco estudiantes."
)

# ==========================================
# HISTORIAL
# ==========================================

if "mensajes" not in st.session_state:
    st.session_state.mensajes = [
        {
            "role": "system",
            "content": PERSONALIDAD
        }
    ]

# Mostrar conversación
for mensaje in st.session_state.mensajes:
    if mensaje["role"] != "system":
        with st.chat_message(mensaje["role"]):
            st.write(mensaje["content"])

# ==========================================
# FUNCIÓN PARA HABLAR CON DEEPSEEK
# ==========================================

def responder(mensaje_usuario):

    st.session_state.mensajes.append({
        "role": "user",
        "content": mensaje_usuario
    })

    respuesta = client.chat.completions.create(
        model="deepseek-flash",
        messages=st.session_state.mensajes,
        temperature=0.8,
        max_tokens=500
    )

    respuesta_texto = respuesta.choices[0].message.content

    st.session_state.mensajes.append({
        "role": "assistant",
        "content": respuesta_texto
    })

    return respuesta_texto


# ==========================================
# CHAT DE TEXTO
# ==========================================

pregunta = st.chat_input(
    "Escribe algo para hablar con FusionBot..."
)

if pregunta:

    with st.chat_message("user"):
        st.write(pregunta)

    with st.chat_message("assistant"):

        with st.spinner("FusionBot está pensando..."):
            respuesta = responder(pregunta)

        st.write(respuesta)

        # Convertir respuesta a voz
        try:
            audio = gTTS(
                text=respuesta,
                lang="es",
                slow=False
            )

            audio_bytes = io.BytesIO()
            audio.write_to_fp(audio_bytes)

            st.audio(
                audio_bytes.getvalue(),
                format="audio/mp3"
            )

        except Exception as e:
            st.warning(
                "No se pudo generar el audio. "
                "La respuesta de texto sí funciona."
            )


# ==========================================
# BOTÓN PARA REINICIAR
# ==========================================

if st.button("🔄 Nueva conversación"):

    st.session_state.mensajes = [
        {
            "role": "system",
            "content": PERSONALIDAD
        }
    ]

    st.rerun()
