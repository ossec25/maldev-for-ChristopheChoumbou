Loader C# : exécution de shellcode (version CreateThread & version Threadless)

Ce dossier contient deux implémentations d’un loader C# permettant d’exécuter du shellcode dans un environnement contrôlé.
Les deux programmes utilisent un shellcode généré via msfvenom, mais diffèrent dans leur manière de l’exécuter.

1. Objectif général
L’objectif de l’exercice est de comprendre :

comment un programme C# peut allouer de la mémoire exécutable,

comment copier un tableau de bytes (shellcode) dans cette mémoire,

comment déclencher l’exécution de ce shellcode,

quelles différences existent entre une exécution via CreateThread et une exécution inline via un pointeur de fonction.

Cet exercice est réalisé dans un cadre strictement académique.

2. Génération du shellcode
Le shellcode est généré depuis Kali Linux :

bash
msfvenom -p windows/x64/messagebox TEXT=SalutChoumbou TITLE=Salut_Choumbou -f csharp
Paramètres :

Plateforme : Windows x64

Payload : MessageBox

Texte : SalutChoumbou

Titre : Salut_Choumbou

Format : C# (byte[])

msfvenom génère un tableau de 314 octets.

 3. Programme A — Loader avec CreateThread
(Version classique)

Description

Ce programme charge le shellcode en mémoire puis l’exécute dans un nouveau thread via l’API Windows CreateThread.

Fonctionnement
Allocation mémoire via VirtualAlloc

Copie du shellcode via Marshal.Copy

Création d’un thread pointant sur l’adresse du shellcode

Exécution du shellcode dans un thread séparé

 Avantages
Simple à comprendre

Représente la méthode classique utilisée dans la plupart des loaders pédagogiques

Inconvénients
CreateThread est une API très surveillée par les antivirus et EDR

L’exécution dans un thread séparé est plus visible dans les logs du système

4. Programme B — Loader sans CreateThread
(Version threadless / inline execution)
Description
Ce programme charge le shellcode en mémoire mais ne crée pas de thread.
Il exécute le shellcode dans le thread courant, en le transformant en pointeur de fonction via un delegate C#.

Fonctionnement
Allocation mémoire via VirtualAlloc

Copie du shellcode via Marshal.Copy

Conversion de l’adresse mémoire en fonction C# :

csharp
ShellcodeRun run = (ShellcodeRun)Marshal.GetDelegateForFunctionPointer(addr, typeof(ShellcodeRun));
Appel direct :

csharp
run();

 Avantages
Pas d’appel à CreateThread → moins détectable

Exécution plus discrète

Technique utilisée dans certains loaders avancés

Inconvénients
Plus difficile à comprendre pour les débutants

Le shellcode s’exécute dans le thread principal → peut bloquer l’application si le shellcode est long

5. Comparaison des deux approches
Critère	Programme A (CreateThread)	Programme B (Threadless)
API utilisée	CreateThread	Delegate + fonction inline
Détection AV/EDR	Plus détectable	Moins détectable
Complexité	Simple	Moyenne
Exécution	Thread séparé	Thread principal
Usage pédagogique	Loader de base	Loader avancé


6. Structure du dossier
Code

Ex1/
 ├── Program.cs        (version CreateThread)
 ├── ProgramB.cs       (version threadless)
 ├── README.md         (ce fichier)
 └── Program.exe       (optionnel)

 7. Conclusion

Ces deux programmes permettent de comprendre :

les bases de l’exécution de shellcode en C#,

les différences entre une exécution via thread et une exécution inline,

les implications en termes de détection et de comportement,

les fondements des loaders utilisés dans les exercices suivants.

Commandes utilisées:

 Compiler et exécuter via dotnet (si tu veux recompiler)
Ton dossier contient aussi le fichier ConsoleAppCH.csproj, donc tu peux utiliser le SDK .NET.

 Compiler :

powershell

dotnet build

Exécuter :

powershell

dotnet run

 Méthode 3 — Exécuter un fichier .cs directement (si tu veux tester ProgramCH.cs)

powershell

dotnet run ConsoleApp1B.cs