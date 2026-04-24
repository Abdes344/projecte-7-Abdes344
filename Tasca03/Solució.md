# 🗂️ Guia d'implantació del servidor de fitxers – FoodLogistic

**Autor:** Abdeslam Khfif Koubee
**Data:** 24/04/2026  
**Entorn:** Windows Server 2025 amb domini foodlogistic.test 

## 🧱 1. Preparació de l’Active Directory (OUs i grups)

### 🔹 Creació del domini i l’OU

Primer, vam pujar el controlador de domini. A les captures veiem com es promociona el servidor a controlador de domini creant un nou forest foodlogistic.test.

![Configuració inicial del servidor](/Tasca03/IMG/1.png)

La imatge mostra el Server Manager local. Encara no tenim el rol d’AD instal·lat.

![Selecció del servidor destí](/Tasca03/IMG/2.png)
 Escollim el servidor Foodlogistic13.

![Selecció del rol AD DS](/Tasca03/IMG/3.png)
 Marcarem Active Directory Domain Services per convertir el servidor en DC.

![Creació d’un nou forest](/Tasca03/IMG/4.png)
 Triem Add a new forest i posem el nom foodlogistic.test.

![Opcions del controlador de domini](/Tasca03/IMG/5.png)
Nivell funcional Windows Server 2025, activem DNS i Global Catalog, i posem la contrasenya de DSRM.

![Nom NetBIOS](/Tasca03/IMG/6.png)
 El NetBIOS queda com FOODLOGISTIC.

![Revisió de la configuració](/Tasca03/IMG/7.png)
 Confirmem tot abans d’instal·lar.


![Creació de l’OU Departments](/Tasca03/IMG/8.png)
Fem clic dret sobre el domini → New → Organizational Unit. Li posem Departments i marquem la protecció per no esborrar-la accidentalment.

Des de l’OU Departments creem els tres grups de seguretat que ens demana la tasca:

![Grups creats a l’OU Departments](/Tasca03/IMG/9.png)
 Dins de Departments tenim els grups **Administració**, **Direcció** i **Transport**. Cadascun és un Security Group de tipus Global (per defecte). Aquests grups s’utilitzaran per donar permisos a les carpetes compartides.

---

## 📂 2. Implementació de les carpetes compartides (tres mètodes)

### 🔸 A. Carpeta Public – Mètode: Explorador d’arxius

Primer de tot, creem la carpeta C:\Public al servidor.

![Creació de la carpeta Public](/Tasca03/IMG/10.png)
Des de l’Explorador, dins de C:\, creem una nova carpeta anomenada Public.

Ara compartim la carpeta des de les propietats → Advanced Sharing.

![Compartició de Public (permisos SMB)](/Tasca03/IMG/11.png)
 Compartim la carpeta amb el nom Public. A la finestra Permissions afegim el grup Everyone i li donem **Change** i **Read** (això són els permisos SMB). Recorda: a nivell SMB estem donant canvi i lectura.

A continuació, ajustem els permisos NTFS a la pestanya Security:

![Permisos NTFS de Public](/Tasca03/IMG/12.png)
A la llista de grups, seleccionem Everyone i marquem **Read & execute**, **List folder contents** i **Read**. **No** marquem Modify ni Write.  
**Combinació efectiva de permisos:**  
- Permís SMB = Change + Read  
- Permís NTFS = Read & execute  
  
Quan accedeixi un usuari qualsevol (per exemple de Transport), s'aplicarà el **permís més restrictiu** entre SMB i NTFS. Aquí el NTFS només permet llegir, per tant l’usuari final **només podrà llegir i executar**, però no modificar, esborrar ni crear fitxers.  

### 🔸 B. Carpeta `Operacions` – Mètode: Server Manager (FSSM)

Ara utilitzarem la consola File and Storage Services del Server Manager. Abans de començar, ens assegurem que el rol està instal·lat (ja ho estarà normalment). A la següent captura veiem que encara no tenim FSRM, però per compartir no cal:

![Vista de Shares des del Server Manager](/Tasca03/IMG/13.png)
A Server Manager → File and Storage Services → Shares veiem els shares existents (NETLOGON, SYSVOL). Clicarem New Share.

Al wizard, triem el perfil SMB Share - Quick:

![Selecció del perfil SMB Quick](/Tasca03/IMG/14.png)
Escollim SMB Share - Quick per anar més ràpid, però sense perdre les opcions importants.

Configurarem la compartició. És important activar **Access‑Based Enumeration** (ABE). Això farà que els usuaris només vegin la carpeta si tenen permisos per accedir-hi.

![Opcions de share amb ABE activat](/Tasca03/IMG/15.png)
Marcar Enable access-based enumeration i deixem la resta per defecte.

A l’apartat Permissions, configurem que només el grup Transport hi pugui accedir:

![Permisos NTFS per a Operacions](/Tasca03/IMG/16.png)
Afegim el principal Transport i li donem **Full Control** (també valdria Modify). Com que ABE està activat, si un usuari de Direcció obre l’explorador de xarxa, **no veurà la carpeta Operacions** perquè no té permisos. Això augmenta la seguretat i redueix el soroll visual.

### 🔸 C/D. Carpeta `Direccio` – Mètode: PowerShell avançat (Mètode D)

El mètode D és el més complet: creem la carpeta, la compartim amb New-SmbShare i després activem l’Access‑Based Enumeration via PowerShell. A més, la GPO mapejarà la unitat Z: només per al grup Direcció.

**Pas 1 – Crear carpeta i compartir-la amb PowerShell**

Obrim PowerShell com a administrador i executem:

powershell
New-Item -Path "C:\Direccio" -ItemType Directory
New-SmbShare -Name "Direccio" -Path "C:\Direccio" -FullAccess "Direccio"
Set-SmbShare -Name "Direccio" -FolderEnumerationMode AccessBased


![PowerShell creant el recurs Direccio](/Tasca03/IMG/17.png)
New-Item crea la carpeta física.  
New-SmbShare comparteix la carpeta amb el nom Direccio i dona FullControl al grup Direcció.  
Set-SmbShare -FolderEnumerationMode AccessBased activa l’ABE. Així els usuaris que no pertanyin a Direcció ni veuran la carpeta dins de la xarxa. Aquest pas és el que diferencia el mètode D del C.

**Pas 2 – Ajustar els permisos NTFS (opcional però recomanat)**

Per assegurar-nos que el grup Direcció té control total sobre la carpeta, mirem els permisos:

![Permisos NTFS de la carpeta Direccio](/Tasca03/IMG/18.png)
A la finestra de seguretat de la carpeta C:\Direccio (a la captura surt C:\ per error, però s’entén que és la carpeta) el grup Direcció té Full control. Això és correcte.

**Pas 3 – GPO per mapejar la unitat Z: exclusiva per a Direcció**

Ara creem una GPO que assigni la unitat Z: apuntant a \\SERVER\Direccio (canvia SERVER pel nom real del teu servidor, per exemple Foodlogistic13). La GPO s’ha d’enllaçar a l’arrel del domini.

![GPO creada al domini](/Tasca03/IMG/19.png)
Creeu una nova GPO anomenada Map_Direccio. L’enllaç apareix a foodlogistic.test.

Editem la GPO i anem a User Configuration → Preferences → Windows Settings → Drive Maps.

![Preferències de la unitat Z](/Tasca03/IMG/20.png)
Afegim una nova unitat: Action = Update, Drive Letter = Z:, Path = \\SERVER\Direccio.

Apliquem un **filtre de seguretat** perquè només s’apliqui als membres del grup Direcció. A la pestanya Common fem clic a Targeting.

![Filtre de targeting per al grup Direcció](/Tasca03/IMG/21.png)
Afegim una condició: the user is a member of the security group FOODLOGISTIC\Direcció. D’aquesta manera, quan un usuari del grup Direcció iniciï sessió, rebrà la unitat Z: automàticament. La resta no la veuran.

---

## 💾 3. Control d’emmagatzematge (Quotes NTFS i FSRM)

El client es queixa que la gent emmagatzema porqueria. Posem solucions.

### 🔸 Quotes NTFS a nivell de volum

Actuem primer sobre el volum C: (la unitat de dades). Al disc on tenim les carpetes compartides, configurem una quota per defecte de **500 MB** per a cada usuari nou. Això és una quota NTFS clàssica.

![Configuració de quota NTFS al volum C:](/Tasca03/IMG/22.png)
A les propietats del C:\, pestanya Quota:  
Enable quota management`  
Deny disk space to users exceeding quota limit  
Limit disk space to 500 MB amb avís a 450 MB  
Marcar les dues opcions de registre d’esdeveniments.  
  
Això vol dir que qualsevol usuari (p. ex. en Transport o Administració) tindrà un límit dur de 500 MB en total **dins del disc C:**. Si intenta guardar més, el sistema li ho denegarà.

---

## 🧪 4. Verificació i auditoria des d’un client

Ara connectem un client Windows (per exemple un Windows 11) al domini i ens loguejarem amb usuaris de cada grup per comprovar que tot funciona.

### 🔹 Accés i visibilitat per a un usuari de Transport

Ens loguem com a Transport (la captura mostra la pantalla de canvi d’usuari):

![Inici de sessió com a Transport](/Tasca03/IMG/25.png)

Un cop dins, obrim l’Explorador i anem a Red → foodlogistic13. Hi veiem les carpetes compartides segons els permisos:

![Vista del servidor des de client Transport](/Tasca03/IMG/27.png)
L’usuari Transport pot veure les carpetes **Public**, **Operacions** (i també NETLOGON, SYSVOL que són per defecte).  
A Public pot llegir però no modificar (NTFS).  
A Operacions té permisos de modificació.  
la carpeta Direccio no apareix (ni tant sols es veu). I la unitat Z: tampoc no es mapeja perquè no pertany a Direcció.

### 🔹 Accés i unitat Z: per a un usuari de Direcció

Quan un usuari del grup Direcció inicia sessió, automàticament rep la unitat Z: que apunta a \\SERVER\Direccio. A més, pot veure la carpeta Direccio a la xarxa? Com que l’ABE està activat, **només els membres de Direcció veuen la carpeta**. En canvi, no veuran Operacions.

![Unitats al client d’un usuari de Direcció](/Tasca03/IMG/28.png)
Al Este equipo apareix una unitat Dirección (Z:) amb **500 MB disponibles de 500 MB**. Aquesta capacitat reflecteix la quota NTFS que hem posat al volum C:. Com que la carpeta Direccio està dins de C:\, l’espai que pot utilitzar aquest usuari està limitat a 500 MB en total (per tots els seus fitxers a C:).  
