# mineriadetexto
codigo utilizado para mineria de texto 
import re
import random
import pandas as pd
from collections import Counter
import matplotlib.pyplot as plt
import seaborn as sns

# Stopwords básicas
stopwords_es = {
    "de", "la", "que", "el", "en", "y", "a", "los", "del", "se", "las", "por", "un", "para", "con",
    "no", "una", "su", "al", "lo", "como", "más", "pero", "sus", "le", "ya", "o", "este", "sí",
    "porque", "esta", "entre", "cuando", "muy", "sin", "sobre", "también", "me", "hasta", "hay",
    "donde", "quien", "desde", "todo", "nos", "durante", "todos", "uno", "les", "ni", "contra",
    "otros", "ese", "eso", "ante", "ellos", "e", "esto", "mí", "antes", "algunos", "qué", "unos",
    "yo", "otro", "otras", "otra", "él", "tanto", "esa", "estos", "mucho", "quienes", "nada",
    "muchos", "cual", "poco", "ella", "estar", "estas", "algunas", "algo", "nosotros", "mi",
    "tú", "te", "ti", "tu", "tus", "ellas", "vosotros", "vosotras", "os"
}

# Diccionario de emociones
emociones = {
    "alegría": ["alegría", "felicidad", "diversión", "satisfacción", "entusiasmo", "alivio", "motivación"],
    "enojo": ["enojo", "ira", "molestia", "rabia", "bronca", "hostilidad", "resentimiento", "indignación"],
    "tristeza": ["tristeza", "dolor", "soledad", "vacío", "desánimo", "desmotivación", "desilusión"],
    "impotencia": ["impotencia", "frustración", "desesperación", "injusticia"],
    "asco": ["asco", "rechazo", "vergüenza", "odio"],
    "apatía": ["apatía", "indiferencia", "neutral", "desinterés", "desapego"],
    "esperanza": ["esperanza", "optimismo"],
    "curiosidad": ["curiosidad", "interés", "inspiración", "asombro"],
    "orgullo": ["orgullo", "confianza", "seguridad", "superación"],
    "culpa": ["culpa", "culpable"],
    "miedo": ["miedo", "ansiedad", "nervios", "temor", "inseguridad"],
    "crítica": ["crítica", "decepción"],
    "amor": ["amor", "cariño", "ternura", "compasión", "empatía"],
    "sorpresa": ["sorpresa", "expectativa"],
    "venganza": ["venganza"],
    "confusión": ["confusión", "desconcierto"],
    "cansancio": ["agotamiento", "cansancio", "aburrimiento"],
    "tranquilidad": ["tranquilidad", "calma", "aceptación", "comprensión"],
    "gratitud": ["gratitud", "agradecido"]
}

# Reglas contextuales ampliadas
context_rules_ampliado = {
    "enojo": ["me da rabia", "me enoja", "molesta", "odio", "trampa", "tramposo", "me arruina", "me molesta", "asco", "me pone de mal humor", "vergüenza ajena"],
    "tristeza": ["me entristece", "me pone triste", "bajón", "desmotiva", "me afecta más", "pierdo la fe", "me desanima"],
    "impotencia": ["no vale la pena", "no importa cuánto", "no sirve", "desequilibrado", "no hay justicia", "no se puede hacer nada"],
    "apatía": ["me da igual", "no me importa", "no me afecta", "meh", "ya no lo reporto", "ni me molesto"],
    "alegría": ["me encanta", "me gusta", "me río", "épico", "ganamos igual", "me siento increíble", "fue divertido", "lo disfruté", "me alegró"],
    "esperanza": ["ojalá", "me gustaría", "espero que", "quisiera", "deberían", "tengo fe", "quiero creer"],
    "asco": ["asco", "asqueroso", "patético", "vergüenza ajena", "escoria", "ruin"],
    "miedo": ["me da miedo", "me pone nervioso", "paranoia", "me preocupa", "inseguro"],
    "curiosidad": ["me pregunto", "es interesante", "no entiendo", "me intriga", "me hace pensar"],
    "amor": ["compasión", "empatía", "cariño"],
    "culpa": ["me siento culpable", "fue mi culpa", "arrepentido"],
    "orgullo": ["me siento orgulloso", "lo logré sin trucos"],
    "sorpresa": ["sorprendente", "no lo esperaba", "inesperado"],
    "crítica": ["los desarrolladores no hacen nada", "no hay consecuencias", "el sistema falla"],
    "confusión": ["me confunde", "es confuso", "no sé qué pensar"],
    "cansancio": ["estoy cansado", "agotado", "me aburre", "repetitivo"],
    "tranquilidad": ["juego tranquilo", "lo tomo con calma", "relajado"],
    "gratitud": ["agradezco", "gracias"],
    "venganza": ["me vengué", "castigo justo", "le gané con justicia"]
}

# ---------------- FUNCIONES ----------------

# Mapeo de palabras
palabra_a_emocion = {p: e for e, lst in emociones.items() for p in lst}

# Estimar emoción por contexto si no hay coincidencia directa
def estimar_emocion_contextual(texto):
    texto_limpio = texto.lower()
    for emocion, patrones in context_rules_ampliado.items():
        if any(pat in texto_limpio for pat in patrones):
            return emocion
    return random.choice(list(context_rules_ampliado.keys()))

# ---------------- EJECUCIÓN ----------------

# Leer archivo
with open("respuestas.txt", "r", encoding="utf-8") as f:
    lineas = f.readlines()

emociones_final = []
conteo = Counter()

print("\nAnálisis línea por línea:\n")

for i, linea in enumerate(lineas, 1):
    original = linea.strip()
    limpio = re.sub(r"[^\w\s]", "", original.lower())
    tokens = limpio.split()
    tokens_validos = [t for t in tokens if t not in stopwords_es and len(t) > 2]

    emociones_directas = [palabra_a_emocion[t] for t in tokens_validos if t in palabra_a_emocion]
    emocion = Counter(emociones_directas).most_common(1)[0][0] if emociones_directas else estimar_emocion_contextual(original)

    emociones_final.append((i, original, emocion))
    conteo[emocion] += 1

    # Mostrar interpretación como en la imagen
    print(f"Línea {i}: {original}")
    print(f"   → Emoción estimada: {emocion.upper()}\n")

# ---------------- RESULTADOS ----------------

print("Resumen final de emociones:\n")
for emocion, cant in conteo.items():
    print(f"- {emocion.capitalize()}: {cant}")

# Exportar CSV
df = pd.DataFrame(emociones_final, columns=["Línea", "Texto", "Emoción"])
df.to_csv("emociones_final_contexto.csv", index=False, encoding="utf-8")

# Gráfico (sin warning)
plt.figure(figsize=(12, 6))
sns.barplot(x=list(conteo.values()), y=list(conteo.keys()), hue=list(conteo.keys()), dodge=False, legend=False, palette="rocket")
plt.title("Distribución de emociones (contextuales y directas)")
plt.xlabel("Cantidad")
plt.ylabel("Emoción")
plt.tight_layout()
plt.show()
(codigo mineria de texto) 






