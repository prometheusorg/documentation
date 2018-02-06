---
title: "Génération automatique de texte"
date: 2018-02-05T06:58:36+01:00
keywords: [ "Episodus", "Nautilus", "epiClassifier", "CISP", "ICPC" ]
---

### Génération automatique de texte

#### Table des matières

1. [Introduction]({{< relref "#introduction" >}})
2. [Les objets]({{< relref "#les-objets" >}})
    1. [Le mot : `NSPhraseMot`]({{< relref "#NSPhraseMot" >}})
    2. [La phrase de base : `NSPhraseur`]({{< relref "#NSPhraseur" >}})
    3. [La proposition et la phrase : `NsProposition`]({{< relref "#NSProposition" >}})
    4. [Le générateur : `NSGenerateur`]({{< relref "#NSGenerateur" >}})
3. [Génération de phrases]({{< relref "generation-de-phrases" >}})
  1. La veine grande saphène gauche est compressible
  2. La veine grande saphène gauche est partiellement compressible
  3. La veine grande saphène gauche est compressible. Elle a une perméabilité altérée
  4. La veine grande saphène gauche est compressible et elle a une perméabilité altérée
  5. Une endoprothèse aortique d'aspect normal est visualisée
  6. Plaque d'athérome responsable d'une sténose artérielle serrée au niveau de l'artère carotide interne
droite

#### Introduction

Le module de synthèse de langage naturel met en œuvre plusieurs types d'objet afin de constituer un graphe de concept grammaticaux puis de les transformer en phrase. Nous allons d'abord énumérer et décrire les objets à l’œuvre, puis donner des exemples de phrases en indiquant comment se mettent en place les objets qui permettent de les générer.

#### Les objets

##### Le mot `NSPhraseMot` {#NSPhraseMot}
L'objet de base, `NSPhraseMot`, représente un mot du Lexique, éventuellement complété de ses compléments (épithètes, compléments du nom, etc).

##### La phrase de base `NSPhraseur` {#NSPhraseur}
La phrase de base est de type « sujet, verbe, compléments » ; ce sont les informations contenues dans un objet `NSPhraseur`.

##### La proposition et la phrase `NsProposition` {#NSProposition}
Les propositions sont de deux types : principale ou secondaire. Il existe de nombreux types de secondaires, les complétives, les circonstancielles, les relatives (que nous traitons en tant que complément du nom) et un type particulier local, la phrase située à droite d'un ' :'.
Dans le cas simple usuel, où il n'existe qu'une unique principale, la `NsProposition` contient un unique `NSPhraseur`. Dans les cas où une même phrase contient plusieurs propositions, que ce soit plusieurs principales (comme « Il mange une pomme et boit de l'eau ») ou une principale et sa/ses secondaire(s), la `NsProposition` contient elle même une array de `NsProposition` (`NSPropositionArray`).

##### Le générateur `NSGenerateur` {#NSGenerateur}
Le générateur prend un objet `NsProposition` et le transforme en une phrase en langage naturel. `NSGenerateur` est un objet virtuel dont dérivent des générateurs spécialisés dans un langage donné, par exemple `NSGenerateurFr` pour le français et `NSGenerateurEn` pour l'anglais.

#### Génération de phrases

Ce document indique la phrase type à générer et l'algorithme d'exemple correspondant

On part du principe que les objets ont été initialisés de la façon suivante:

```c++
_pPhraseur = new NSPhraseur(pContexte);
_pPropos = new NsProposition(pContexte, &_pPhraseur, NsProposition::notSetType, NsProposition::notSetConjonct);
_pGenerateur = new NSGenerateurFr(pContexte, _pPropos, _sLang);
```

Le générateur de langage est donc lié à une proposition simple, elle même liée à un objet de description de phrase. La `NsProposition` est construite à partir d'un pointeur sur `_pPhraseur` qui est, lui même un pointeur. Le comportement par défaut de ce constructeur est de signaler que `_pPhraseur` doit être détruit en même temps que la `NsProposition`.

##### La veine grande saphène gauche est compressible.

Phrase simple, sujet et attribut du sujet. Le verbe «être» est automatiquement sélectionné.

```c++
_pPhraseur->initialise();
_pPhraseur->addSubject(string("AGDGS1"));
_pPhraseur->AttSujet.push_back(new NSPhraseMot(string("QCMPR2")));
_pGenerateur->genereProposition(dcPhrase);
_pGenerateur->termineProposition();
string sSentence = _pGenerateur->getPropositionPhrase();
```

Attention : le sujet doit être un nom et l'attribut un adjectif, sinon la phrase ne sera pas générée. Si on ajoute plusieurs attribut, on obtient une phrase de type «est compressible, échogène et continente».

##### La veine grande saphène gauche est partiellement compressible.

L'attribut du sujet reçoit un adverbe.
```c++
_pPhraseur->initialise();
_pPhraseur->addSubject(string("AGDGS1"));
NSPhraseMot *pAttribut = new NSPhraseMot(string("QCMPR2"));
NSPhraseur* pAttrPhr = pAttribut->getOrCreateFirstComplementPhr();
pAttrPhr->adverbe.push_back(new NSPhraseMot(pContexte, string("1PART2")));
_pPhraseur->AttSujet.push_back(pAttribut);
_pGenerateur->genereProposition(dcPhrase);
_pGenerateur->termineProposition();
string sSentence = _pGenerateur->getPropositionPhrase();
```
Attention : le terme du lexique désigné comme adverbe doit être... un adverbe.

##### La veine grande saphène gauche est compressible. Elle a une perméabilité altérée

Le sujet de la seconde phrase n'est pas répété et remplacé par un pronom.

```c++
_pPhraseur->initialise();
_pPhraseur->addSubject(string("AGDGS1"));
_pPhraseur->AttSujet.push_back(new NSPhraseMot(string("QCMPR2")));
_pGenerateur->genereProposition(dcPhrase);
_pGenerateur->termineProposition();
string sSentence = _pGenerateur->getPropositionPhrase() ;
_pPhraseur->initialise();
_pPhraseur->addSubject(string("AGDGS1")) ;
_pPhraseur->iTypeSujet = NSPhraseur::sujetNoRepeat;
_pPhraseur->addVerb(string("4AVOI1"));
NSPhraseMot Cod(string("QPERM1"));
NSPhraseur* pAttrPhr = Cod.getOrCreateFirstComplementPhr();
if (pCplmt)
  pCplmt->adjEpithete.push_back(new NSPhraseMot(string("SALTD2"), pContexte));
_pPhraseur->addCOD(&Cod);
_pGenerateur->genereProposition(dcPhrase) ;
_pGenerateur->termineProposition();
sSentence += string(" ") + _pGenerateur->getPropositionPhrase();
```

##### La veine grande saphène gauche est compressible et elle a une perméabilité altérée

Phrase similaire à la phrase précédente, mais sous forme de deux propositions principales au sein de la même phrase.

```c++
NsProposition metaProposition(pContexte, true);
_pGenerateur->setProposition(&metaProposition);
_pPhraseur->initialise();
_pPhraseur->addSubject(string("AGDGS1"));
_pPhraseur->AttSujet.push_back(new NSPhraseMot(string("QCMPR2")));
metaProposition.addPropositionToArray(_pPhraseur, NsProposition::principale,
NsProposition::notSetConjonct);
_pPhraseur->initialise();
_pPhraseur->addSubject(string("AGDGS1"));
_pPhraseur->addVerb(string("4AVOI1"));
NSPhraseMot Cod(string("QPERM1"));
NSPhraseur* pAttrPhr = Cod.getOrCreateFirstComplementPhr();
if (pCplmt)
  pCplmt->adjEpithete.push_back(new NSPhraseMot(string("SALTD2"), pContexte));
_pPhraseur->addCOD(&Cod);
metaProposition.addPropositionToArray(_pPhraseur, NsProposition::principale, NsProposition::notSetConjonct);
_pGenerateur->genereProposition(dcPhrase);
_pGenerateur->termineProposition();
string sSentence = _pGenerateur->getPropositionPhrase();
_pGenerateur->setProposition(_pPropos);
```

Dans cet exemple, on commence par remplacer, au sein du générateur, la proposition simple (qui ne contient qu'un objet NSPhraseur) par une proposition multiple. La dernière ligne de l'exemple remet en place la proposition simple par défaut.
Après que le NSPhraseur qui correspond à une principale a été initialisé, il est ajouté à la proposition multiple par la méthode addPropositionToArray.

##### Une endoprothèse aortique d'aspect normal est visualisée

Dans cet exemple, «d'aspect normal» est une contraction de la subordonnée relative « dont l'aspect est normal ». On construit donc la subordonnée complète (sujet, verbe être, attribut du sujet), puis on précise que cette subordonnée doit être générée en « mode tiret »

```c++
_pPhraseur->initialise();
NSPhraseMot Endoprothese(pContexte, string("AENAO1"));
Endoprothese.setArticle(NSPhraseMot::articleIndefini);
NSPhraseur* pPhraseur = new NSPhraseur(pContexte) ;
pPhraseur->addSubject(string("0ASPV1"));
pPhraseur->addVerb(string("4ETRE1"));
pPhraseur->addAttSujet(&_prothAspect);
pPhraseur->_iRelativeType = dcTiret;
Endoprothese.addComplementPhr(pPhraseur, phraseRelative);
_pPhraseur->addCOD(&Endoprothese);
_pPhraseur->addVerb(string("4VISU1"));
_pPhraseur->iForme = NSPhraseur::formePassive;
_pGenerateur->genereProposition(dcPhrase);
_pGenerateur->termineProposition();
```

##### Plaque d'athérome responsable d'une sténose artérielle serrée au niveau de l'artère carotide interne droite.

La phrase est une « phrase à tiret ». De telles phrases doivent être construites comme une phrase à la forme passive (« Il existe une plaque... ») qui verra son sujet et son verbe (« Il existe ») supprimés par la précision du « mode tiret » au moment de la génération en langage naturel:
`_pGenerateur->genereProposition(dcTiret);`

On commence par définir la localisation, avec pour particularité de préciser la préposition «au niveau de»:

```c++
_pPhraseur->initialise();
NSPhraseMot localisation(pContexte, string("ADT231"));
localisation.setArticle(NSPhraseMot::articleDefini);
_pPhraseur->addCCLieu(&localisation);
_pPhraseur->PrepositionLieu.setLexique(string("1AUNI1"));
```

On inscrit ensuite le COD (« plaque d'athérome »):
```c++
NSPhraseMot COD(pContexte, string("SPLAQ1"));
COD.initComplement();
```
Puis on construit une subordonnée relative (« qui est responsable d'une sténose... ») :
```c++
NSPhraseur* pPhraseur = new NSPhraseur(pContexte) ;
pPhraseur->iPhraseType = phraseRelative ;
NSPhraseMot verbe(pContexte, string("4ETRE1")) ;
pPhraseur->addVerb(&verbe) ;
NSPhraseMot attribut(pContexte, string("1RESP1")) ;
NSPhraseur* pCplt = attribut.getOrCreateFirstComplementPhr() ;
if (pCplt) {
  // Il existe "une" sténose
  NSPhraseMot stenose(pContexte, string("PSTA01"));
  stenose.setArticle(NSPhraseMot::articleIndefini);
  pCplt->addSubject(&stenose);
}
pPhraseur->addAttSujet(&attribut);
COD.addComplementPhr(pPhraseur, phraseRelative);
```
La construction de la phrase « résume » le récit suivant : « il existe une plaque, qui est responsable que une sténose existe ». C'est ce qui explique que « sténose » soit en position de sujet du complément du nom « responsable ».
Il ne reste plus qu'a finaliser, en précisant bien qu'on veut une phrase « à tiret » :
```c++
_pPhraseur->addCOD(&Cod);
_pGenerateur->genereProposition(dcTiret);
_pGenerateur->termineProposition();
string sSentence = _pGenerateur->getPropositionPhrase();
```
