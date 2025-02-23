# DIA3--rendu-equat-diff
rendu sur les activités des équations différentielles

import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
from sklearn.metrics import mean_squared_error

# Partie 1: Modélisation et compréhension mathématique du problème

# 1. Écrire l’équation différentielle avec les valeurs du problème.
 dP/dt = r * P * (1 - P/K)
# où:
# P = population (nombre de clients) au temps t
# r = taux de croissance
# K = capacité maximale (taille du marché total)
# Avec les valeurs fournies:
 dP/dt = 0.05 * P * (1 - P/100000)

# 2. Expliquer pourquoi l’interprétation fonctionne au début de l’activité et à la fin de l’activité.
# Début: Au début, P est petit, donc P/K est proche de 0. L'équation se rapproche de dP/dt = r * P, ce qui est une croissance exponentielle.
# Fin: Lorsque P approche K, P/K approche 1, et (1 - P/K) approche 0. Cela signifie que dP/dt approche 0, la croissance ralentit.

# 3. Utiliser la méthode de discrétisation pour obtenir l’équation d’Euler qui permet d’approximer la valeur de la solution.
# P(t + dt) = P(t) + dt * dP/dt
# En substituant l'équation logistique:
# P(t + dt) = P(t) + dt * r * P(t) * (1 - P(t)/K)

# 4. Implémenter en Python cette équation en choisissant un pas approprié.
def euler_logistic(P0, r, K, dt, num_steps):
    """
    Approximation de la solution de l'équation différentielle logistique en utilisant la méthode d'Euler.

    Args:
        P0: Population initiale.
        r: Taux de croissance.
        K: Capacité maximale.
        dt: Pas de temps.
        num_steps: Nombre de pas à effectuer.

    Returns:
        Un tuple contenant les valeurs de temps et les valeurs de population à chaque pas.
    """
    time = np.arange(0, num_steps * dt, dt)
    P = np.zeros(num_steps)
    P[0] = P0

    for i in range(1, num_steps):
        P[i] = P[i-1] + dt * r * P[i-1] * (1 - P[i-1]/K)

    return time, P

# Paramètres
P0 = 1000
r = 0.05
K = 100000
dt = 1  # Pas de temps (1 jour)
num_steps = 365  # Nombre de jours dans l'année

# Exécuter la méthode d'Euler
time_euler, P_euler = euler_logistic(P0, r, K, dt, num_steps)

# Afficher les résultats
plt.plot(time_euler, P_euler)
plt.xlabel("Temps (jours)")
plt.ylabel("Population")
plt.title("Approximation de la croissance logistique par la méthode d'Euler")
plt.grid(True)
plt.show()

# Partie 2: Etude de la précision de la solution

# 1. Expliquer la cohérence de cette solution avec le lancement exponentiel et la saturation
# La solution analytique capture la croissance exponentielle initiale lorsque P est petit par rapport à K. Lorsque P approche K, la croissance ralentit.

# 2. Implémenter cette fonction
def analytical_logistic(P0, r, K, t):
    """
    Calcul la solution analytique de l'équation différentielle logistique.

    Args:
        P0: Population initiale.
        r: Taux de croissance.
        K: Capacité maximale.
        t: Temps (scalaire ou tableau).

    Returns:
        La population au temps t.
    """
    return K / (1 + (K - P0) / P0 * np.exp(-r * t))

# Calculer la solution analytique pour les mêmes valeurs de temps que Euler
P_analytical = analytical_logistic(P0, r, K, time_euler)

# Afficher les résultats
plt.plot(time_euler, P_analytical)
plt.xlabel("Temps (jours)")
plt.ylabel("Population")
plt.title("Solution analytique de la croissance logistique")
plt.grid(True)
plt.show()

# 3. Comparer les résultats d’approximations avec le résultat de cette fonction pour plusieurs valeurs en vous basant sur la MSE.

# Calculer la MSE
mse = mean_squared_error(P_analytical, P_euler) # Compare la solution analytique et Euler
print(f"Erreur quadratique moyenne (MSE): {mse}")

# 4. Tracez les courbes de votre approximation et de la solution analytique.

# Tracer les deux solutions
plt.plot(time_euler, P_euler, label="Approximation d'Euler")
plt.plot(time_euler, P_analytical, label="Solution analytique")
plt.xlabel("Temps (jours)")
plt.ylabel("Population")
plt.title("Comparaison de l'approximation d'Euler et de la solution analytique")
plt.legend()
plt.grid(True)
plt.show()

# Partie 3: Application sur des données réels et recalibrage du modèle

# 1. Tracer la courbe du nombre d’utilisateurs.

# Charger les données du fichier CSV
df = pd.read_csv("Dataset_nombre_utilisateurs.csv")
df = df.rename(columns={'Jour': 'Temps', 'Utilisateurs': 'Nombre_utilisateurs'})

# Afficher la courbe du nombre d'utilisateurs
plt.figure(figsize=(12, 6))
plt.plot(df['Temps'], df['Nombre_utilisateurs'])
plt.xlabel("Jour")
plt.ylabel("Nombre d'utilisateurs")
plt.title("Nombre d'utilisateurs au fil du temps")
plt.grid(True)
plt.show()

# 2. Quand est ce qu’on atteint la phase de “saturation” ? Quand est ce qu’on atteint 50% de la saturation ?

# On utilise une valeur proche du max des utilisateurs dans le dataset
K_estime = df['Nombre_utilisateurs'].max()

# Approximation du point de saturation (95% de K)
saturation_threshold = 0.95 * K_estime
saturation_day = df[df['Nombre_utilisateurs'] >= saturation_threshold]['Temps'].min()

# Approximation du jour à 50% de saturation
half_saturation = 0.5 * K_estime
half_saturation_day = df[df['Nombre_utilisateurs'] >= half_saturation]['Temps'].min()

print(f"Jour approximatif de saturation: {saturation_day}")
print(f"Jour approximatif à 50% de saturation: {half_saturation_day}")

# 3. Calculer la MSE pour les 2 solutions sur plusieurs intervalles de temps.

# Intervalles de temps (en jours)
intervals = [(0, 30), (30, 90), (90, 180), (180, 365)]

for start, end in intervals:
    # Sélectionner les données réelles dans l'intervalle
    real_data_interval = df[(df['Temps'] >= start) & (df['Temps'] <= end)]['Nombre_utilisateurs']
    #Euler
    time_interval = time_euler[(time_euler >= start) & (time_euler <= end)]
    P_euler_interval = np.interp(time_interval, time_euler, P_euler)  # Interpoler Euler values
    #Analytical
    P_analytical_interval = analytical_logistic(P0, r, K, time_interval)
    #MSE Calculation
    mse_euler = mean_squared_error(real_data_interval, P_euler_interval[:len(real_data_interval)])
    mse_analytical = mean_squared_error(real_data_interval, P_analytical_interval[:len(real_data_interval)])
    print(f"MSE entre {start} et {end} jours (Euler): {mse_euler}")
    print(f"MSE entre {start} et {end} jours (Analytique): {mse_analytical}")

# 4. Tracer simultanément les 3 courbes.

# Tracer les trois courbes
plt.figure(figsize=(12, 6))
plt.plot(df['Temps'], df['Nombre_utilisateurs'], label="Données réelles")
plt.plot(time_euler, P_euler, label="Approximation d'Euler")
plt.plot(time_euler, P_analytical, label="Solution analytique")
plt.xlabel("Temps (Jours)")
plt.ylabel("Nombre d'utilisateurs")
plt.title("Comparaison des données réelles, de l'approximation d'Euler et de la solution analytique")
plt.legend()
plt.grid(True)
plt.show()

# 5. Expliquer pour quelle(s) raison(s) les écarts sont significatifs ?
# Les écarts peuvent être dus à:
# - Saisonnalité: L'adoption peut varier selon les périodes de l'année.
# - Campagnes marketing: Des pics peuvent être liés à des efforts marketing.
# - Facteurs externes: Événements, concurrence, économie.
# - Limitations du modèle: Le modèle logistique est une simplification.

# 6. Proposer de nouvelles hypothèses pour donner une meilleur approximation
# Hypothèses à considérer:
# - Taux de croissance variable (r): Influencé par le marketing ou la saisonnalité.
# - Capacité maximale variable (K): La taille du marché peut évoluer.
# - Incorporer des facteurs externes: Ajouter des termes pour le marketing ou autres.

# 7. Recalculer la MSE et retracer les courbes

# Exemple de modèle modifié (Taux de croissance variable)
def modified_logistic(P0, r_func, K, t):
    """
    Équation logistique avec un taux de croissance variable.
    r_func doit être une fonction qui prend le temps (t) en entrée et renvoie le taux de croissance à ce moment.
    """
    return K / (1 + (K - P0) / P0 * np.exp(-np.cumsum([r_func(ti) for ti in t])))

# Exemple de fonction de taux de croissance (peut être ajustée)
def r_func(t):
    """
    Exemple: Un taux de croissance qui diminue avec le temps.
    """
    return 0.05 * np.exp(-t / 100)  # Décroissance exponentielle du taux de croissance

# Recalculer en utilisant le modèle modifié
P_modified = modified_logistic(P0, r_func, K, time_euler)

# Calculer la MSE
mse_modified = mean_squared_error(df['Nombre_utilisateurs'], P_modified[:len(df['Nombre_utilisateurs'])])
print(f"MSE avec le modèle modifié: {mse_modified}")

# Tracer les résultats
plt.figure(figsize=(12, 6))
plt.plot(df['Temps'], df['Nombre_utilisateurs'], label="Données réelles")
plt.plot(time_euler, P_euler, label="Approximation d'Euler (Original)")
plt.plot(time_euler, P_analytical, label="Solution analytique (Original)")
plt.plot(time_euler, P_modified, label="Modèle modifié")
plt.xlabel("Temps (Jours)")
plt.ylabel("Nombre d'utilisateurs")
plt.title("Comparaison des modèles")
plt.legend()
plt.grid(True)
plt.show()
