# Burke–Xu Algorithmus für LP (Python) – Bachelorarbeit

Dieses Repository enthält die Python-Implementierung des in **Kapitel 3** meiner Bachelorarbeit beschriebenen
**nicht-inneren Prädiktor–Korrektor Pfadverfolgungs-Algorithmus** (nach Burke–Xu) zur Lösung von
**linearen Programmen (LP)**.

---

## Kurzbeschreibung

Wir betrachten LPs in Standardform

> min cᵀx  s.t.  Ax = b,  x ≥ 0

mit Dualvariablen λ und Slacks s. Über zentrale Pfadbedingungen (µ > 0) und eine geglättete
Komplementaritätsfunktion Φ(x,s,µ) wird ein nichtlineares Gleichungssystem F(x,λ,s,µ)=0 formuliert,
das iterativ via **Predictor/Corrector**-Schritten und **Backtracking** gelöst wird.

---

## Repository-Struktur

- `burke_xu_lp.py`  
  Hauptsolver `burke_xu_lp(...)` (LP-Daten ähnlich SciPy `linprog`)
- `methods.py`  
  Kernbausteine: Φ/∇Φ, Predictor/Corrector, Line-Search/Backtracking, Umformungen LP ↔ Standardform,  
  sparse Cholesky (CHOLMOD via scikit-sparse) + Regularisierung
- `mainlp.py`  
  Batch-Runner für Netlib-Probleme (`.npz`), schreibt Ergebnisdateien (z.B. Text/LaTeX-Ausgaben)

---

## Installation

Getestet mit Python 3.12.x.

Benötigte Pakete:
- numpy
- scipy
- scikit-sparse (benötigt SuiteSparse/CHOLMOD)

Beispiel:
```bash
pip install numpy scipy scikit-sparse
```

Hinweis: `scikit-sparse` setzt systemseitig SuiteSparse/CHOLMOD voraus (Installation je nach OS).

---

## Quickstart: ein Problem lösen

Beispiel (wenn ein Problem als `.npz` im üblichen Benchmark-Format vorliegt):

```python
import numpy as np
import scipy.sparse as sp
from burke_xu_lp import burke_xu_lp

data = np.load("data/AFIRO.npz", allow_pickle=True)

c = data["c"]
A_eq = sp.csc_matrix(data["A_eq"])
b_eq = data["b_eq"]
A_ub = sp.csc_matrix(data["A_ub"])
b_ub = data["b_ub"]
bounds = np.squeeze(data["bounds"])  # (n,2) mit (lb, ub)

x, slack, (fun, mu, phi_inf), nullsteps, iters, t = burke_xu_lp(
    c,
    A_eq=A_eq, b_eq=b_eq,
    A_ineq=A_ub, b_ineq=b_ub,
    bounds=bounds,
    acc=1e-4,
    maxiter=1000,
    presolve=(True, True, None),
    scaling=0,
    sigma=0.5,
    alpha_1=0.75,
    alpha_2=0.8,
    verbose=False,
)

print("fun =", fun, "mu =", mu, "phi_inf =", phi_inf, "iters =", iters, "time =", t)
```

---

## Netlib Batch-Run

Wenn Netlib-Probleme als `.npz` in `data/` liegen (und die zugehörigen Metadaten/Listen wie in `mainlp.py`
erwartet werden), dann:

```bash
python mainlp.py
```

---

## Thesis / PDF

Die Bachelorarbeit (PDF) liegt im Repo unter:

- `docs/Bachelorarbeit.pdf`

---

## Lizenz

Noch nicht festgelegt (TBD).
