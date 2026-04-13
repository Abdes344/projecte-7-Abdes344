# 🗄️ T03: Servidor de fitxers – FoodLogistic S.A. 💻

---

## 🚀 Introducció

Amb el creixement de FoodLogistic S.A. 🏭🍎, la gestió de dades s’ha convertit en un problema: cada departament guardava fitxers localment, sense control ni estructura.

👉 L’objectiu d’aquest projecte és implementar un **servidor de fitxers centralitzat, segur i eficient**, amb:
- 🔐 Permisos NTFS i SMB
- 📊 Quotes d’emmagatzematge
- 🚫 Filtrat de fitxers (FSRM)
- 🧑‍💻 Gestió amb GUI i PowerShell

---

# 🏗️ 1. Preparació i Active Directory (AD)

## 🧑‍🤝‍🧑 Grups creats

| Grup | Descripció |
|------|------------|
| Administracio | Gestió de factures i albarans 📄 |
| Transport | Xofers i caps de flota 🚛 |
| Direccio | Gerència i direcció 👔 |

---

## 🏢 Estructura OU (proposta)

- OU=FoodLogistic
  - OU=Usuaris
  - OU=Departaments
  - OU=Grups
  - OU=Servidors

👉 Aquesta estructura permet una gestió ordenada i escalable.

---

# 📁 2. Recursos compartits

## 📂 A. Carpeta Public (Explorador de fitxers)

- 📍 Ruta: `C:\Shares\Public`
- 🌍 Accés: Tothom
- 🔐 Permisos:
  - SMB: Lectura
  - NTFS: Modificació

👉 Resultat: tots els usuaris poden accedir i modificar fitxers

---

## 📂 B. Carpeta Operacions (Server Manager)

- 📍 Ruta: `C:\Shares\Operacions`
- 👥 Grup amb accés: Transport 🚛
- ⚙️ Configuració:
  - Share creat amb **File and Storage Services**
  - ✅ Access-Based Enumeration activat

👉 Només els usuaris de Transport veuen i accedeixen a la carpeta

---

## 📂 C. Carpeta Confidencial (PowerShell bàsic) 🔐

- 📍 Ruta: `C:\Shares\Direccio$`
- 🧑‍💼 Grup: Direccio
- 💻 Comanda:

```powershell
New-SmbShare -Name "Direccio$" -Path "C:\Shares\Direccio$" -FullAccess "Direccio"
