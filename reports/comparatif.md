# Comparatif des approches - projet fraude (TP04)

Sources des couts cloud : Google (docs.cloud.google.com/vertex-ai/pricing), AWS
(aws.amazon.com/sagemaker/ai/canvas/pricing), Microsoft (azure.microsoft.com/en-us/pricing/details/machine-learning)
- consultations du 22/10/2026.

| Approche | F1 test | Temps humain | Temps machine | Cout estime (source, date) | Explicabilite | Export / reversibilite |
|-----------------------|-------------|--------------|---------------|-------------------------|---------------|------------------------|
| Logistique (ref.) | 0.7407 | ~2 min (1 script de 10 lignes) | 0.36 s | 0 (poste local) | Maximale (coefficients) | Total |
| Manuel (TP2) | 0.8669 | ~1 h au TP2 (grille manuelle) | 5.2 s | 0 (poste local) | Moyenne/Bonne (feature_importances_) | Total |
| FLAML 60 s | 0.8614 | ~20 min (script + lecture du journal) | 60 s (82 essais) | ~0.02 EUR d'electricite | Moyenne/Basse (importances du gagnant seulement) | Total |
| Vertex AI AutoML | non mesure | ~30-60 min (console, navigateur) | n/d (1-3 h conseilles pour <100k lignes) | 21.25 USD/node-h d'entrainement, soit ~21-64 USD pour 1-3 h (Google, 22/10/2026); endpoint facture des qu'il est deploye | Bonne (valeurs de Shapley globales et locales) | Faible / Verrouillage (Vendor Lock-in) : export Docker du modele tabulaire possible mais sans l'explicabilite Vertex, CPU x86 AVX uniquement |
| SageMaker Autopilot | non mesure | ~30-60 min (studio/Canvas) | n/d | session Workspace 1.9 USD/h + heures d'instances d'entrainement (~2.3-9.2 USD pour <100 Mo; build standard ~7.30 USD) (AWS, 22/10/2026); tier gratuit 2 mois | Bonne (rapport SHAP via Clarify, notebooks generes, rapport de performance) | Faible / Verrouillage (Vendor Lock-in) : artefacts S3 et notebooks recuperables, redeployer ailleurs demande du travail |
| Azure Automated ML | non mesure | ~30-60 min (studio) | n/d | pas de surcharge AutoML : facturation VM sous-jacente (ex. DS14 v2 : 1.196 USD/h) (Microsoft, 22/10/2026); facturation a la seconde | Bonne (Responsible AI dashboards : SHAP, analyse d'erreurs) | Faible / Verrouillage (Vendor Lock-in) : modele enregistre au format MLflow telechargeable, meilleur portage que GCP/AWS |

## Notes E1 - Journalisation

- **Google Vertex AI** : pipelines d'entrainement, jeux de donnees versionnes, metriques et essais accessibles via l'interface des pipelines et le Model Registry ; le detail par essai est moins lisible que le journal FLAML.
- **SageMaker Autopilot** : Autopilot job avec leaderboard des candidats, SageMaker Experiments, trois notebooks generes (exploration des donnees, definition des candidats, insights du meilleur modele), lineage disponible.
- **Azure Automated ML** : runs MLflow natifs (params/metriques/artefacts), Model Registry, dashboards Responsible AI ; logs d'essais complets.

## Commentaire (5 lignes)

- Logistique : bonne reference a garder en surveillance de base, pour sa transparence totale (coefficients), son cout nul et son F1 honnete (0.7407) ; a recommander quand l'interpretabilite prime (secteur regule).
- Manuel (TP2) : preferable quand le probleme est stable et qu'un expert peut consacrer une heure de reglage ; ici le meilleur F1 (0.8669) et la maitrise complete du code.
- FLAML : a recommander comme brique locale repetitive du pipeline : le meilleur compromis qualite/effort (~20 min humaines, 60 s machine) quand l'equipe veut garder le code et le journal sous controle.
- Plateformes managées : a recommander pour une equipe sans competence ML ni ops, qui veut une infra geree (scalabilite, endpoints, monitoring) et accepte le cout des heures d'entrainement et le verrouillage fournisseur.
- Pour ce TP : FLAML offre ~95-99 % de la qualité du reglage manuel pour ~0.02 EUR ; les plateformes cloud ne se justifient que si l'entreprise ne veut rien exploiter en local (ops, mise a l'echelle, conformite) et selon le volume budgete.
