# ===============================================
# CHATBOT MÉTÉO HISTORIQUE (version dynamique)
# ===============================================

import pandas as pd
import nltk

# Téléchargement NLTK (une seule fois)
nltk.download('punkt', quiet=True)

print("Chargement des données météo...")
df = pd.read_csv('data/temperature.csv')

# Conversion de la colonne datetime
df['datetime'] = pd.to_datetime(df['datetime'])

# 🔹 Liste dynamique des villes (toutes les colonnes sauf datetime)
villes = list(df.columns)
villes.remove('datetime')

# Pour faciliter la détection (minuscules)
villes_lower = [v.lower() for v in villes]

print("\nBonjour ! Je suis ton assistant météo historique 🌦️")
print(f"Je connais {len(villes)} villes.")
print("Exemple : Quelle était la température à Seattle le 2012-10-01 ?")
print("Tape 'quit' pour arrêter.\n")

while True:
    question = input("Toi : ")

    if question.lower() == 'quit':
        print("Au revoir 👋")
        break

    ville_trouvee = None
    date_trouvee = None

    # 🔹 Recherche de la ville dans la question
    for ville in villes:
        if ville.lower() in question.lower():
            ville_trouvee = ville
            break

    # 🔹 Recherche d'une date YYYY-MM-DD
    for mot in question.split():
        try:
            date_trouvee = pd.to_datetime(mot, format='%Y-%m-%d')
            break
        except:
            pass

    # 🔹 Si ville + date trouvées
    if ville_trouvee and date_trouvee is not None:
        jour_data = df[df['datetime'].dt.date == date_trouvee.date()]

        if jour_data.empty:
            print("Chatbot : Je n’ai pas de données pour cette date.\n")
            continue

        if ville_trouvee not in jour_data.columns:
            print("Chatbot : Ville inconnue.\n")
            continue

        # Suppression des valeurs manquantes
        valeurs = jour_data[ville_trouvee].dropna()

        if valeurs.empty:
            print(f"Chatbot : Pas de données pour {ville_trouvee} ce jour-là.\n")
            continue

        # 🔹 Moyenne journalière
        temp_kelvin = valeurs.mean()
        temp_celsius = temp_kelvin - 273.15

        print(
            f"Chatbot : La température moyenne à {ville_trouvee} "
            f"le {date_trouvee.date()} était de {temp_celsius:.1f} °C 🌡️\n"
        )

    else:
        print(
            "Chatbot : Je n'ai pas compris 😕\n"
            "Utilise une ville connue et une date (YYYY-MM-DD).\n"
        )
