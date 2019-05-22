# CompteRendu
## Le sujet du stage est d'améliorer le pretty printer Pharo

# Présentation de Pharo :
+ un language de programmation orienté objet
+ inspiré par Smaltalk
+ pure et élégant
+ dynamiquement typée
+ écrit en lui même.

# Syntaxe :
## éléments syntaxique :
<table>
      <tr>
        <td>commentaire</td>
        <td>"un commentaire"</td>
      </tr>
      <tr>
        <td>caractère</td>
        <td>$a</td>
      </tr>
      <tr>
        <td>chaîne de caractère</td>
        <td>'uneChaine'</td>
      </tr>
      <tr>
        <td>symbole(chaîne unique)</td>
        <td>#unSymbole</td>
      </tr>
      <tr>
        <td>tableau de littéraux</td>
        <td>#(1 2 3 4 5 6 )</td>
      </tr>
      <tr>
        <td>entier</td>
        <td>1</td>
      </tr>
      <tr>
        <td>booléen</td>
        <td>true, false</td>
      </tr>
      <tr>
        <td>indéfinie</td>
        <td>nil</td>
      </tr>
</table>

## constructeurs :

<table>
      <tr>
        <td>declaration de variable temporaires</td>
        <td>| tmp |</td>
      </tr>
      <tr>
        <td>affectation de variables</td>
        <td>tmp:= uneValeur</td>
      </tr>
      <tr>
        <td>séparateur</td>
        <td>message1. Message2</td>
      </tr>
      <tr>
        <td>return</td>
        <td>^ uneValeur</td>
      </tr>
      <tr>
        <td>block (fonction anonyme)</td>
        <td>[:x | x+2 ] value: 5</td>
      </tr>
</table>

## les messages dans pharo :
<table>
      <tr>
        <td>message unaire (receveur selecteur)</td>
        <td>3 factorial</td>
      </tr>
      <tr>
        <td>message binaire (receveur selecteur argument)</td>
        <td>1+2</td>
      </tr>
      <tr>
        <td>message à mots clés(receveur clé1:arg1 clé2:arg2)</td>
        <td>2 between : 10 and: 2</td>
      </tr>
</table>

## definition de classe de pharo:

### Exemple de la definition de la classe Point avec deux attributs: x et y

    Object subclass: #Point
    instanceVariableNames: 'x y'
    classVariableNames: ''
    package: 'Graphics'

### definition de methode

    MethodeExample
    "aComment"

    | tmp |
    tmp := 1.
    ^ tmp


# L'AST (Abstract Syntax Trees)
les methodes sont encodés sous forme d'un arbre syntaxique. (composés de différent noeuds)

par exemple:

    | tmp |
    tmp := 1.
    ^ tmp
    
est representé ainsi:

    RBSequenceNode(| tmp| tmp := 1)
        RBVariableNode(tmp)
        RBAssignementNode(tmp := 1)
            RBLiteralValueNode(1)
            RBVariableNode(tmp)

# Le pretty printer
le pretty printer sert a reformater du code suivant une configuration définie

par exemples l'ajout d'espace pour l'assignation d'une variable:

    'tmp:=1' sera reformaté en 'tmp := 1'

Soit une méthode :

        MethodeExample
        "aComment"
        |tmp|
        tmp:=1. ^tmp

le pretty printer reformate cette méthode ainsi:

        MethodeExample
            "aComment"

            | tmp |
            tmp := 1.
            ^ tmp


Il existe des configurations afin de définir les règles de formatage.

# Semaine du 29/04:
J'ai regardé comment le pretty printer est implémenté.
Le RBProgrameNodeVisitor est un visiteur abstrait permettant de visiter les noeuds de l’ast.

Le pretty printer hérite du RBProgrameNodeVisitor.
Il surcharge les méthodes de visite afin de reformater les noeuds visités selon une configuration.
Pour cela il possede un attribut nommé codeStream dans lequel il écrit le résultat.
La configuration est un attribut du pretty printer (contextClass),
elle est une instance de la classe BIPrettyPrinterContext et contient une trentaine de variables.
par exemple: 

    newLinesAfterMethodPattern: un booleen indiquant s'il faut passer une ligne après la signature d'une methode
        

# Semaine du 06/05:
tests unitaires sur le formatage des différents noeuds syntaxiques en fonction d'une configuration donné.

Exemple d'une methode de test verifiant que les éléments d'un assignmentNode soient espacé:    
le test:

    testAssignment   
    | expr source |
    expr := RBParser parseExpression: 'a:=1'.
    configurationSelector := #emptyDefaultConfiguration.
    source := self formatter format: expr.
    self assert: source equals: 'a := 1'
    
la configuration:

    emptyDefaultConfiguration
    "Here we can control explicitely the configuration we want."
    ^ self contextClass new

# Semaine du 13/05:
    suite des tests sur le formatage des noeuds de l'ast.
    Cependant les tests présentent un défaut.
    
    les configurations sont toutes crées avec "self contextClass new" qui les initialises avec des valeurs par defaut.
    les tests dépendent donc d'une configuration par défaut (composé d'une trentaine d'attributs).
    
    J'ai donc modifié les configuration en utilisant "self contextClass basicNew"
    basic new crée une configuration avec tous ses attributs à nil.
    en partant de cette configuration vide, j'instancie les attributs nécessaire et laisse les autres à nil.
    ce qui me permet de faire des tests en isolant au plus la configuration.
