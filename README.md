# ARIMA-GARCH-modelling

# Modélisation ARIMA & GARCH - Rendements et volatilité du S&P 500

Projet de prévision des rendements et de la volatilité du S&P 500 (2015–2024), combinant un modèle ARIMA pour la moyenne conditionnelle et un modèle GJR-GARCH pour la variance conditionnelle, évalués en walk-forward (fenêtre roulante) sur la période 2022–2024.

## Données

- Source : cours de clôture du S&P 500 (`^GSPC`), via `yfinance`, du 2015-01-01 au 2024-12-31.
- Rendements log, exprimés en %.
- Découpage : entraînement sur 2015-2021, test sur 2022-2024 (prévisions à un pas, en fenêtre extensible).

## Outils

Python, `statsmodels`, `arch`, `pmdarima`, `yfinance`, `scikit-learn`, `matplotlib`

## Méthodologie

1. **Analyse exploratoire** : test de stationnarité (ADF), analyse ACF/PACF.
2. **Modèle ARIMA** : sélection par grille AIC/BIC (ordres p, q de 0 à 3), confirmée par `auto_arima`. Modèle retenu : ARIMA(3,0,2). Diagnostic des résidus (Ljung-Box, Jarque-Bera).
3. **Test ARCH-LM** sur les résidus ARIMA, pour justifier la modélisation de la variance conditionnelle.
4. **Modèles GARCH** : comparaison d'un GARCH(1,1) et d'un GJR-GARCH(1,1) (loi t de Student) sur les résidus d'entraînement, sélection par AIC/BIC/log-vraisemblance. Modèle retenu : GJR-GARCH(1,1), pour son terme d'asymétrie (effet de levier).
5. **Prévisions walk-forward** : réestimation quotidienne des deux modèles sur la période de test, avec prévision à un pas à chaque étape (aucune fuite de données futures).
6. **Évaluation** : MAE et RMSE, comparées à des prévisions de référence (benchmarks naïfs) plutôt qu'interprétées en valeur absolue.

## Résultats

### Rendements (ARIMA)

| | MAE | RMSE |
|---|---|---|
| ARIMA(3,0,2) | 0.8361 % | 1.1207 % |
| Prévision naïve (constante) | 0.8123 % | 1.1026 % |

L'ARIMA ne bat pas une prévision constante des rendements. C'est un résultat attendu et cohérent avec l'hypothèse de marché efficient : les rendements financiers sont proches d'un bruit blanc en moyenne, donc difficiles à prévoir avec un modèle linéaire simple sur la seule base de leur propre passé.

### Volatilité (GJR-GARCH)

Deux cibles de volatilité réalisée ont été utilisées pour l'évaluation :

**1. Écart-type glissant sur 21 jours**

| | MAE | RMSE |
|---|---|---|
| GJR-GARCH | 0.2030 % | 0.2799 % |
| Persistance (valeur de la veille) | 0.0349 % | 0.0589 % |

Sur cette cible, la persistance bat largement le GJR-GARCH. Ce résultat est trompeur : le chevauchement des fenêtres de 21 jours rend la cible très autocorrélée par construction (deux jours consécutifs partagent 20 des 21 rendements utilisés dans leur calcul), ce qui avantage artificiellement toute prévision naïve, indépendamment de tout pouvoir prédictif réel.

**2. Rendement au carré du jour (proxy non lissé de la variance)**

| | MAE | RMSE |
|---|---|---|
| GJR-GARCH | 1.3087 % | 2.3583 % |
| Persistance (valeur de la veille) | 1.6579 % | 3.1710 % |

Sur cette cible sans chevauchement de fenêtre, le GJR-GARCH bat la persistance d'environ 20 % sur les deux métriques. Ce résultat confirme que le modèle capture une vraie dynamique de la volatilité, une fois l'artefact du lissage écarté.

## Conclusion

Le pipeline distingue deux résultats de nature différente :
- Sur la **moyenne** des rendements, l'ARIMA n'apporte pas de valeur ajoutée mesurable par rapport à une prévision constante, un résultat cohérent avec la littérature sur la prévisibilité limitée des rendements financiers.
- Sur la **variance**, le GJR-GARCH bat un benchmark de persistance dès lors que la cible d'évaluation n'est pas elle-même lissée par une fenêtre glissante. Le coefficient d'asymétrie du GJR est significatif et positif, confirmant que les chocs négatifs augmentent davantage la volatilité future que les chocs positifs de même ampleur (effet de levier).

## Limites

- La cible de volatilité réalisée (écart-type glissant ou rendement au carré) reste un proxy imparfait de la volatilité latente.
- Le modèle GARCH est réestimé quotidiennement sur toute la période de test, ce qui est coûteux en temps de calcul et discutable en usage réel (une réestimation moins fréquente pourrait être envisagée).

## Pistes d'amélioration

- Backtest de Value-at-Risk (test de Kupiec, comptage des exceptions) à partir de la volatilité prévue par le GJR-GARCH, pour évaluer le modèle dans une optique de gestion des risques plutôt que de pure prévision ponctuelle.
- Diagnostic complet des résidus standardisés du modèle GARCH retenu.
- Comparaison à d'autres spécifications (EGARCH, modèles à mémoire longue).

## Auteur

Lilian Nkwemfo
