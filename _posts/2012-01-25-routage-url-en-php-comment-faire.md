---
date: 2013-12-25 20:39:00
layout: default
slug: routage-url-en-php-comment-faire
title: Routage d'URL en PHP - Comment faire ?
categories:
- Web Development
tag:
-PHP5
-URI
-URL
---


##  Routage d'URL en PHP - Comment faire ?

Aujourd'hui, je vais <del>tenter de</del> vous expliquer comment mettre en
place un système simple de routage des URLs. Nous verrons dans un premier
temps pourquoi il peut être utile de mettre en place un tel système et nous
passerons ensuite à sa réalisation. Le but de l'article n'est pas de vous
apprendre à bâtir un système complet et pouvant rivaliser avec les scripts
qu'on peut trouver dans certains frameworks. Au contraire cet article est là
pour poser quelques bases et permettre à ceux qui le souhaitent d'aller plus
loin. Considérez cet article comme un introduction imagée au routage d'URL =)

## Pourquoi ? Pour qui ? Comment ?

Commençons par voir comment le système va se matérialiser. Un exemple concret
est le meilleur moyen de visualiser la solution finale. Nous allons dans un
premier temps partir du besoin «client»
pour voyager vers la réalisation technique d'une solution.

### Le besoin

Vous disposez de diverses pages sur votre site que vous atteignez par le biais
d'URLs tels que fichier.php?arg=xx&arg1=xx

Dans ce flot de page, imaginez un instant que vous possédez une page
présentant le profil d'un utilisateur. Un utilisateur X  veut donner l'adresse
de son profil à un utilisateur Y. Il lui donne alors l'adresse : [http://site.
domaine/voir_user.php?pseudo=xXxX](http://site.domaine/voir_user.php?pseudo=xX
xX)

Difficile à retenir non ? Une URL plus «user-
friendly» sera plus adaptée, par exemple :
[http://site.domaine/profil/xXxX](http://site.domaine/profil/xXxX)

Mais ceci n'est qu'un aspect de notre besoin. Naturellement, dans une optique
de référencement, vous souhaiteriez que vos URLs soit révélatrices du contenu.
Autre chose, le moteur de référencement de Google a la réputation de ne pas
aimer les adresses qui comportent plus de deux arguments. ( Type ?x=1&z=3&e=4
)

Nous avons exprimé le besoin d'un point de vue Webmaster, nous allons
maintenant descendre vers l'analyse technique de notre système.

## Mise en place du système

Le première chose à faire est de mettre en place la redirection de tout appel
votre fichier index.php. Pour ce faire nous utiliserons un fichier .htaccess
(Partant du principe que vous utilisez Apache )

    
    
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    
    RewriteRule ^(.*)$ index.php?rt=$1 [L,QSA]
    

Voici donc le contenu de notre fichier .htaccess. Rien d'extravagant juste une
redirection vers notre fichier index.php de nos requêtes.

Voyons maintenant le contenu de notre fichier index.php

    
    
    <?php
    /* Index.php file
    * Created in October, the 26th 2011
    */
    
    /* Constantes */
    define ('DEFAULT_MODULE', 'acceuil');
    define ('DEFAULT_FUNCTION', 'index');
    
    define ('DIR_CLASS', __DIR__.'/controller/');
    define ('DIR_MODEL', __DIR__.'/lib/');
    define ('DIR_VIEW',  __DIR__.'/view/');
    
    /* On récupère l'URL */
    $appel_url = $_SERVER['REQUEST_URI'];
    
    /* Test if URI == null
    *  If null, display default module
    */
    
    /* On détermine les arguments
    *
    *	L'URL doit être comme suit : controller/function/arg1/arg2/arg3
    *
     */
    $arguments = explode('/', $appel_url);
    
    /*
    * Structure ternaire pour définir quel module & fonction appeler
    */
    $class_to_load = $arguments[1] == '' ? DEFAULT_MODULE : $arguments[1];
    
    $function_to_exec = $arguments[2] == '' ? DEFAULT_FUNCTION : $arguments[2];
    
    $file_to_call = DIR_CLASS.'/'.$class_to_load.'.php';
    
    try {
    
    	/* Vérification de routine */
    	if (!file_exists($file_to_call) )
    	{
    		throw new Exception('Erreur : La classe '.$class_to_load.' n\'existe pas. ');
    	}
    	require_once(DIR_CLASS.'/'.$class_to_load.'.php');
    
    	if (!class_exists($class_to_load) )
    	{
    		throw new Exception('Erreur : La classe '.$class_to_load.' \' n\'est pas définie.' );
    	}
    
    	/* Appel du contrôleur */
    	$object_call = new $class_to_load();
    
    	if (!method_exists($object_call, $function_to_exec) )
    	{
    		throw new Exception('Erreur: La fonction '.$function_to_exec.' n\'est pas d&eacute;finie. ');
    	}
    	if (!is_callable(array($object_call, $function_to_exec) )  )
    	{
    		throw new Exception('Erreur: La methode '.$function_to_exec.' doit &ecirc;tre définie comme public. ');
    	}
    
    	$object_call->$function_to_exec();
    
    	/*
    	* And We Are Done !
    	*/
    
    }catch(Exception $e)
    {
    	exit($e->getMessage() );
    }
    
    ?>
    

Un peu plus à expliqué ici :

On commence par définir quelques constantes pour le bon fonctionnement du
script. Viens ensuite une partie importante qui est celle de la récupération
de la requête entrante grâce à la variable $_SERVER['REQUEST_URI'] (_Si vous
ne connaissez pas cette variable, je vous renvoie vers une recherche google
sur le thème des Variables Superglobales en PHP_ )

On sépare ensuite nos différents éléments avec la fonction **explode** pour
récupérer deux éléments important :

  * Le nom du contrôleur à appeler
  * Le nom de la fonction à exécuter

On pratique ensuite une série de tests pour être sûres que notre contrôleur
existe, et que la fonction visée est utilisable. On instancie ensuite notre
contrôleur, puis nous lançons notre fonction.

## Et pour la suite ?

L'article touche à sa fin, avec un goût de non fini et ce de manière
volontaire. J'ai posé ici les bases pour un contrôle des URLs avec
l'utilisation du pattern MVC, mais il manquerais bien des choses pour ce que
soit réellement intéressant. Je vous laisse faire des recherches et modifier
le script selon vos envies. Une première amélioration serait d'intégrer
l'autoloading rendu possible avec la présence de
[SPL](http://fr.php.net/manual/fr/intro.spl.php)  _(Standard PHP Library)
_ou/et encore un système automatisé de gestion des vues. Bien du travail en
perspective pour arriver à un système complet et flexible. A défaut de temps,
il y a toujours [Symfony](http://http://www.symfony.com)