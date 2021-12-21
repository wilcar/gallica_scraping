Gallica\_scraping
================

# Identifiants ARK (Archival Resource Key)

## De quoi s’agit-il ?

L’ARK (Archival Resource Key) est un format d’identifiant créé en 2001
par la California Digital Library (CDL) qui a vocation à identifier des
ressources de tous types – physiques (échantillons destinés à une
expérience scientifique, etc.), numériques (livres numérisés, notices de
catalogue, etc.). Son but est de fournir des identifiants adaptés aux
besoins des producteurs et diffuseurs de données sur le web, mais
également capables de durer sur le long terme (identification pérenne).
Depuis 2007, la BnF utilise l’identifiant ARK pour nommer les ressources
qu’elle crée.

## Références

<https://github.com/hackathonBnF/hackathon2016/wiki/SERVICE-VISU-ISSUES>
<https://github.com/hackathonBnF/hackathon2016/wiki/API-Gallica---Document>

## composition

La partie obligatoire de l’identifiant est constituée des éléments
suivants :

1.  Le type, ou schème, d’identifiant « ark: » déclare qu’il s’agit d’un
    identifiant ARK.

2.  Le numéro d’autorité nommante correspondant à une organisation
    habilitée à attribuer des ARK.

3.  Le nom ARK est une chaîne de caractères spécifique à la ressource
    attribué par l’autorité nommante.

4.  Optionnellement, sont définis des qualificatifs, c’est-à-dire des
    suffixes permettant d’identifier des composantes ou variantes d’une
    ressource (par exemple /f pour folio et /highres pour haute
    définition).

# Exemple du livre d’heures en français selon l’usage de Paris (images)

"<https://gallica.bnf.fr/ark:/12148/btv1b525070859/f1/highres%22> :
folio 1 en haute définition de l’ark btv1b525070859 attribué par
l’organisation 12148.

## Téléchargement d’une image unique

``` r
url <- "https://gallica.bnf.fr/ark:/12148/btv1b525070859/f1/highres"    

download.file(url,"fichier.jpg", mode = "wb") # mode wb pour Windows.
```

## Téléchargement de la totalité des images

Le livre d’heures en français selon l’usage de Paris comprend 340 folios

``` r
for (i in 1:340) {
  download.file(paste0("https://gallica.bnf.fr/ark:/12148/btv1b525070859/f",i ,"/highres"),
                paste0("gallica_livre_heures_btv1b525070859-",i, ".jpg"), mode = "wb")
}
```

# Exemple de l’Ouest éclair, édition de Nantes (texte brut)

## Numéro unique du premier janvier 1916 de l’Ouest éclair, édition de Nantes

``` r
library(htm2txt)
url <- 'https://gallica.bnf.fr/ark:/12148/bpt6k567105k.texteBrut'
loecl16_01_01 <- gettxt(url)

# Visualisation

cat(loecl16_01_01)
```

## Liste des numéros pour l’année 1915

Url (Gallica)

<https://gallica.bnf.fr/ark:/12148/cb41193663x/date1915>

Url (xml)

<https://gallica.bnf.fr/services/Issues?ark=ark:/12148/cb41193663x/date&date=1915>

### Algorithme

1.  Lecture des identifiants arks de chaque numéro et des dates de
    publication au format XML contenus dans une page web.

2.  Analyse syntaxique (parser, en anglais) du contenu XML de la page
    web et conversion en tableau de données.

3.  Sélection de la colonne contenant les identifiants arks.

4.  Initalisation d’un objet de type texte dont la fonction sera de
    stocker le texte de la totalité des numéros publiés. Il s’agit de
    créer un objet vide.

5.  Boucle.

### Programme

``` r
# L'Ouest Eclair édition de Nantes, année 1915
#############################################
# Chargement des bibliothèques

library(httr)      # connexion R / internet
library(xml2)      # parsing xml
library(htm2txt)   # fonction gettxt récupère le texte contenu dans une page html
library(tidyverse) # 

# loen15 : l'ouest éclaire édition de Nantes 1915

# Récupération du texte contenu dans la page xml listant les exemplaires de L'OE

loen15_numeros_xml <- GET("https://gallica.bnf.fr/services/Issues?ark=ark:/12148/cb41193663x/date&date=1915")

# Parsing XML et conversion en tableau de données
loen15_liste_numeros_tbl  <- loen15_numeros_xml  %>%
  content() %>% 
  xml_find_all(".//issue") %>% 
  map_df(~ c(as.list(xml_attrs(.x)), date_parution = xml_text(.x)))

# Sélection de la colonne contenant les arks

arks_liste <- loen15_liste_numeros_tbl [,'ark']

# Initialisation d'une variable "string"

loen15_text <- c()

# Boucle 

for (i in unlist(arks_liste)) {
  url <- paste0("https://gallica.bnf.fr/ark:/12148/", i, ".texteBrut")
   loen15_text<- c(loen15_text ,gettxt(url))
}
```
