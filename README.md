# congestiones-H3
Código que permite identificar los casos que se atribuyen a H3. El viaje fue eliminado por una razón atribuible al concesionario, sin embargo, el siguiente viaje fue eliminado por congestión vehicular.
import pandas as pd

# ===============================
# CARGA DE DATOS
# ===============================
RUTA_ARCHIVO = r"C:\Users\DELL\Downloads\viajes.xlsx"
df = pd.read_excel(RUTA_ARCHIVO)

# ===============================
# NORMALIZACIÓN DE TEXTO
# ===============================

def normalizar(texto):
    return (
        str(texto)
        .strip()
        .lower()
        .replace("á","a")
        .replace("é","e")
        .replace("í","i")
        .replace("ó","o")
        .replace("ú","u")
    )

df["Descripcion_norm"] = df["Descripcion"].apply(normalizar)
df["Eliminado_norm"] = df["Eliminado"].apply(normalizar)

# ===============================
# CREAR LLAVE DE BLOQUE
# ===============================

df["Bloque"] = (
    df["Fecha"].astype(str) + "_" +
    df["Linea"].astype(str) + "_" +
    df["Coche"].astype(str)
)

df = df.sort_values(["Bloque", "ViajeLinea"])

df["Marca"] = ""
bloques_marcados = []

# ===============================
# CAUSALES A VALIDAR EN VIAJE ANTERIOR
# ===============================

CAUSALES = [
    "concesionario no envia movil",
    "no se presenta conductor a realizar servicio",
    "bus varado en la via"
]

# ===============================
# ANALISIS POR BLOQUE
# ===============================

for bloque, grupo in df.groupby("Bloque"):

    grupo = grupo.sort_values("ViajeLinea").reset_index()

    for i in range(1, len(grupo)):

        viaje_actual = grupo.loc[i]
        viaje_anterior = grupo.loc[i - 1]

        # Si el actual fue eliminado por congestión
        if (
            viaje_actual["Eliminado_norm"] == "eliminado"
            and viaje_actual["Descripcion_norm"] == "congestion vehicular"
        ):

            # Validar el inmediatamente anterior
            if (
                viaje_anterior["Eliminado_norm"] == "eliminado"
                and any(causal in viaje_anterior["Descripcion_norm"] for causal in CAUSALES)
            ):

                # Marcar TODO el bloque como H3
                df.loc[df["Bloque"] == bloque, "Marca"] = "H3"
                bloques_marcados.append(bloque)

# ===============================
# RESULTADO FINAL
# ===============================

df_resultado = df[df["Bloque"].isin(bloques_marcados)]

df_resultado.to_excel(r"C:\Users\DELL\Downloads\viajes_marcado_H3.xlsx", index=False)

print("Proceso finalizado.")
print("Bloques marcados:", len(set(bloques_marcados)))
