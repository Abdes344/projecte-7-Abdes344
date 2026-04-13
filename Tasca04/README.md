# 🖨️ T04: Servidor d’impressió – FoodLogistic S.A. 🚛📦

---

## 🚀 Introducció

En una empresa logística com **FoodLogistic S.A.** 🏭🍎, la impressió d’albarans i documents de transport és crítica.

👉 Si una impressora falla, els camions no poden sortir i es trenca tota la cadena logística ❌🚛

Per això, s’implementa un **Servidor d’Impressió centralitzat amb balanceig de càrrega (Printer Pooling)** 🖨️⚖️

---

# 🏗️ 1. Preparació de l’entorn

## 🖨️ Impressores creades

S’han instal·lat dues impressores virtuals PDF24:

- 🖨️ IMP_MAGATZEM_A  
- 🖨️ IMP_MAGATZEM_B  

👉 Objectiu: simular dues impressores físiques reals al magatzem

---

# ⚙️ 2. Instal·lació del rol i configuració

## 🧩 Rol instal·lat

- 📦 Print and Document Services

---

## 🖥️ Configuració Printer Pooling

### 🔧 Passos realitzats

1. 📂 Obrir **Print Management**
2. 🖨️ Seleccionar `IMP_MAGATZEM_A`
3. ⚙️ Anar a pestanya **Ports**
4. ✅ Activar **Enable printer pooling**
5. ➕ Afegir el port de `IMP_MAGATZEM_B`

---

## ⚖️ Resultat

👉 Les dues impressores funcionen com una sola cua de treball  
👉 El sistema distribueix automàticament la càrrega 🧠⚖️

---

# 👥 3. Desplegament amb GPO

## 📦 Objectiu
Evitar instal·lació manual per part dels usuaris del magatzem ❌🧑‍💻

---

## 🧷 GPO creada

- 📛 Nom: `GPO_Impressores_Magatzem`

---

## 🔗 Assignació

- 📍 Vinculada a la OU del magatzem
- 🖨️ Impressora desplegada automàticament

---

## 💻 Comprovació al client

- 👤 Inici de sessió Windows 11
- 🖨️ Impressora apareix automàticament
- 🔄 Execució de:
```bash
gpupdate /force
